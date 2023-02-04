---
title: "Dealing with forward zones and DNS UPDATE in PowerDNS"
date: 2022-12-23T18:49:52+01:00
draft: true
weight: 0
tags: ["PowerDNS", "DNS", "dnsdist"]
summary: "Or how I'm using PowerDNS Authoritative Server, Recursor and dnsdist to manage and update internal domains while also supporting forward lookups."
ShowToc: true
cover:
    image: "img/dnsoverkill.webp"
    alt: "Dad and his son fighting with toy swords, dad's sword was the logo for PowerDNS added on top of it while the son has been labelled as 'hosting 1 internal domain'"
    caption: "God gives her greatest challenges to her silliest clowns [image credit](https://www.pexels.com/photo/a-father-and-sob-playing-with-wooden-swords-toy-7104295/)"
    relative: true
---

For the services I'm hosting at home I wanted an internal domain to make accessing those services easier. Because no one likes remembering IP addresses.  

There are multiple suitable DNS servers on the market that you could implement that would get you there. However I had a few added 'wants' that led me to PowerDNS.

{{<callout type="info">}}
This post omits a lot of PowerDNS basics as it's specifically about setting it up for DNS UPDATE but I'll cover the basics in a future post.
{{</callout>}}

## Requirements

- Support for multiple backends (I'm using sqlite for the record)
- An HTTP API (SOAP would have been fine, REST using JSON is ideal)
- Support for exporting metrics (Bit overkill for a home environment but it's good practice)
- Support for Dynamic DNS Updates ({{< extlink url="https://www.rfc-editor.org/rfc/rfc2136" text="RFC2136" >}})    
- Ability to perform recursive lookups to external domains.

## PowerDNS does it all, kinda

**PowerDNS Authoritative Server** on its own ticks 4/5 boxes. 
As an Authoritative DNS server it does not support recursive lookups, it used to but they split that into a separate product a long time ago.
Personally I think that's a good thing as the pieces of software, while both carrying the PowerDNS name, serve completely different purposes.

Alas, we want recursion. When running the PowerDNS authoritative server it only makes sense that we use the **PowerDNS Recursor** as well. 
Recursor allows you to send requests to specific server depending on the domain. Of course you're free to swap this out with Bind or whatever you want.
However, Recursor does not allow you to forward DNS UPDATE messages to the authoritative server. For that we'll need another piece of software...

Last but not least, also developed by the people behind PowerDNS, **dnsdist** enters the system. Quite a clever bit of software that acts as a DNS loadbalancer.
You can route or modify DNS packets based on pretty much any of their attributes and its configuration is written in Lua. 
Using a programming language as configuration tool does increase the learning curve but it is hella flexible.

## Set-up

### Choice of host
For my DNS needs it seemed like a bad idea to host the services as a virtual machine on [the Proxmox server]({{< ref "/posts/homelabbing_part_1" >}}) as it might end up creating a circular dependency.  

Luckily, I still have my Raspberry Pi 3B which up until now has maybe seen up to a few hours of usage. 
It's an 64-bit ARM device which is good because they're dropping 32-bit ARM support as soon as the last supported version (4.4.x) hits EOL (Which it did on the 20th of october 2022 lol).  

### So about that 64-bit ARM support...
PowerDNS doesn't have builds for 64-bit ARM yet. If you're running on bare Pi you're stuck on the 32-bit version of version 4.4 which is EOL. {{< extlink url="https://github.com/PowerDNS/pdns/issues/8655" text="They are working on 64-bit builds however." >}} You can of course opt to run PowerDNS in Docker but I felt that added some operational overhead I didn't want at this stage. 

{{< callout type="warning" >}}
Running an outdated version is not ideal but in a situation where only my local devices can touch the PowerDNS services I decided the risk is acceptable. If your systems touch the internet you might want to reconsider.
{{< /callout >}}

### Things to consider

PowerDNS Authoritative Server, PowerDNS Recursor and dnsdist are not usually going to be running on the same host and they all expect to be the sole DNS service.
This also means that they're all going to want to live on port 53 because they're designed to handle DNS queries. 
If you know a thing or two about sockets you know you can't have two processes listening on the same socket. They wouldn't be able to determine who the data is for, anyway.  
Luckily each of them allows you to set the socket the run on and dnsdist doesn't care what ports your backend run on.

### The current situation

In the end my set-up looks like this:

{{< lightbox img="img/PowerDNS_diagram_1.png" group="diagrams" center="si" >}}

dnsdist has port 53 on the host and is listening for DNS queries from elsewhere on my network, currently all devices using DHCP use the pi (and dnsdist) as main DNS.

PowerDNS Authoritative server only listens locally on port 25301, ditto for Recursor with port 25302 instead. This way all traffic has to pass through dnsdist. 

{{<callout type="info">}}
In this set-up I am only allowing systems on my main IP range (10.69.0.1/24) to send DNS UPDATE messages through dnsdist to the authoritative server. 
The allow list on the authoritative server no longer has any use because all requests are coming from 127.0.0.1 because they come from dnsdist running on the same system. Get it?
{{</callout>}}

## Configuration files

My configuration files are as follows:

### PowerDNS Authoritative server
As you can see this configuration also enables the webserver and API in the authoritative webserver, you can set these to `no` as they're not required for our goal.
```ini
launch=gsqlite3
gsqlite3-database=/var/lib/powerdns/pdns.sqlite3

local-address=127.0.0.1
local-port=25301

webserver=yes
webserver-address=10.69.0.33
webserver-allow-from=10.69.0.1/24,10.42.0.1/24

api=yes
api-key=<generate your own key>

dnsupdate=yes
allow-dnsupdate-from=127.0.0.0/8
default-ttl=60
```
### PowerDNS Recursor
Just a simple configuration here, everything for dy.lan gets routed to our local authoritative server on port 25301 and the other zones go to Cloudflare public DNS. 
```ini
local-address=127.0.0.1
local-port=25302
allow-from=127.0.0.0/8
forward-zones=dy.lan=127.0.0.1:25301
forward-zones-recurse=.=1.1.1.1;1.0.0.1
```

### dnsdist  
Dnsdist uses Lua for its configuration, it has a bit of a learning curve and the nesting of rules can get confusing quick. If you want some seriously different rules definitely check out the official documentation on {{< extlink url="https://dnsdist.org/rules-actions.html" text="Packet policies" >}}

```lua
-- disable security status polling via DNS, we know we're EOL
setSecurityPollSuffix("")

-- set primary address
setLocal('127.0.0.1')
-- add an address
addLocal('10.69.0.33')
-- set local control socket
controlSocket('127.0.0.1:5199')
-- set encryption key for control socket
setKey("key generated within the unencrypted CLI with makeKey()")

-- add our backends
newServer({address='127.0.0.1:25302', name="recursor"})
newServer({address='127.0.0.1:25301', name="authoritive", pool={"auth"}})

-- set the server policy
setServerPolicy(firstAvailable)
-- addAction(AllRule(), LogAction("/etc/dnsdist/dnsdist.log", false, true, false))

-- set the packet cache
pc = newPacketCache(10000, {maxTTL=86400, minTTL=0, temporaryFailureTTL=60, staleTTL=60, dontAge=false})
getPool(""):setCache(pc)

-- drop everything not local
addAction(NotRule(OrRule({makeRule("10.0.0.0/8"), makeRule("127.0.0.1")})), DropAction())
-- Allow DNS Update from the specified host and send it to the pool with our authoritative server
addAction(AndRule({OpcodeRule(DNSOpcode.Update),OrRule({makeRule("10.69.0.112"), makeRule("10.69.0.69")})}), PoolAction("auth"))
```

Within the dnsdist cli you can issue the showServers() function and it'll happily spit out the backends it knows about:
```text
> showServers()
#   Name                 Address                       State     Qps    Qlim Ord Wt    Queries   Drops Drate   Lat Outstanding Pools
0   recursor             127.0.0.1:25302                  up     2.0       0   1  1         34       0   0.0   5.3           0
1   authoritive          127.0.0.1:25301                  up     0.0       0   1  1          0       0   0.0   0.0           0 auth
All                                                              1.0                        34       0

```