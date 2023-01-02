---
title: "Infinite Power(DNS)"
date: 2022-12-23T18:49:52+01:00
draft: true
weight: 0
tags: ["PowerDNS", "DNS", "dnsdist"]
summary: "Or how I'm using PowerDNS Authoritative Server, Recursor and dnsdist to manage and update internal domains."
cover:
    image: "img/dnsoverkill.webp"
    alt: "Dad and his son fighting with toy swords, dad's sword was the logo for PowerDNS added on top of it while the son has been labelled as 'hosting 1 internal domain'"
    caption: "God gives his greatest challenges to his silliest clowns [image credit](https://www.pexels.com/photo/a-father-and-sob-playing-with-wooden-swords-toy-7104295/)"
    relative: true
---

For the services I'm hosting at home I wanted an internal domain to make accessing those services easier. Because no one likes remembering IP addresses.  

Of course there are multiple suitable DNS servers on the market that you could implement that would get you there. However I had a few added 'wants' that led me to PowerDNS.

## Requirements

- Support for multiple backends (I'm using sqlite for the record)
- An HTTP API (SOAP would have been fine, REST using JSON is ideal)
- Support for exporting metrics (Bit overkill for a home environment but it's good practice)
- Support for Dynamic DNS Updates ([RFC2136](https://www.rfc-editor.org/rfc/rfc2136))
- Ability to perform recursive lookups to external domains.

## PowerDNS does it all, kinda

**PowerDNS Authoritative Server** on its own ticks 4/5 boxes. 
As an Authoritative DNS server it does not support recursive lookups, it used to but they split that into a separate product a long time ago.
Personally I think that's a good thing as the pieces of software, while both carrying the PowerDNS name, serve completely different purposes.

Alas, we want recursion. When running the PowerDNS authoritative server it only makes sense that we use the **PowerDNS Recursor** as well. 
Recursor allows you to send requests to specific server depending on the domain. Of course you're free to swap this out with Bind or whatever you want.
However, Recursor does not allow you to forward DNS UPDATE messages to the authoritative server. For that we'll need another piece of software.

Last but not least, also developed by the people behind PowerDNS, **dnsdist** enters the system. Quite a clever bit of software that acts as a DNS loadbalancer.
You can route or modify DNS packets based on pretty much any of their attributes and its configuration is written in Lua. 
Using a programming language as configuration tool does increase the learning curve but it is hella flexible.

## Set-up

### Choice of host
For my DNS needs it seemed like a bad idea to host the services as a virtual machine on [the Proxmox server]({{< ref "/posts/homelabbing_part_1" >}}) as it might end up creating a circular dependency.  

Luckily, I still have my Raspberry Pi 3B which up until now has maybe seen up to a few hours of usage. 
It's an 64-bit ARM device which is good because they're dropping 32-bit ARM support as soon as the last supported version (4.4.x) hits EOL (Which it did on the 20th of october 2022 lol).  

### Needs some more time in the oven
But here's the bad news, they don't have builds for 64-bit ARM yet. Some people are running it on Docker and using scary build flags to run newer versions.
But if you're running on bare Pi you're stuck on version 4.4 which is EOL. {{< extlink url="https://github.com/PowerDNS/pdns/issues/8655" text="They are working on 64-bit builds tho." >}}  

This is not ideal but in a situation where only my local devices can touch the PowerDNS services I decided the risk of running an EOL version is acceptable.

### Things to consider

PowerDNS Authoritative Server, PowerDNS Recursor and dnsdist are, in real-world cases not usually going to be running on the same host and they all expect to be the sole DNS service.
This also means that they're all going to want to live on port 53 because they're designed to handle DNS queries. 
If you know a thing or two about sockets you know you can't have two processes listening on the same socket. They wouldn't be able to determine who the data is for, anyway.

### The current situation

In the end my set-up looks like this:

{{< lightbox img="img/PowerDNS_diagram_1.png" group="diagrams" center="si" >}}

dnsdist has port 53 on the host and is listening for DNS queries from elsewhere on my network, currently all devices using DHCP use the pi (and dnsdist) as main DNS.

PowerDNS Authoritative server only listens locally on port 25301, ditto for Recursor with port 25302 instead. This way all traffic has to pass through dnsdist.

My configuration files are as follows:

**PowerDNS Authoritative server**
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
allow-dnsupdate-from=10.69.0.112/32,127.0.0.0/8
default-ttl=60
```

**PowerDNS Recursor**
```ini
local-address=127.0.0.1
local-port=25302
allow-from=127.0.0.0/8
forward-zones=dy.lan=127.0.0.1:25301
forward-zones-recurse=.=1.1.1.1;1.0.0.1
```

**dnsdist**
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

-- start the internal webserver and configure it (password moved to config item in later versions of dnsdist)
webserver("10.69.0.33:8083", "supersecretpassword")
setWebserverConfig({apiKey="supersecretAPIkey", acl="10.69.0.0/24, 10.42.0.0/24"})
```

Within the dnsdist cli you can issue the showServers() function and it'll happily spit out the backends it knows about:
```text
> showServers()
#   Name                 Address                       State     Qps    Qlim Ord Wt    Queries   Drops Drate   Lat Outstanding Pools
0   recursor             127.0.0.1:25302                  up     2.0       0   1  1         34       0   0.0   5.3           0
1   authoritive          127.0.0.1:25301                  up     0.0       0   1  1          0       0   0.0   0.0           0 auth
All                                                              1.0                        34       0

```

## Basic Operations

### Adding a domain
Earlier we've set up PowerDNS Recursor to forward requests for the 'dy.lan' domain to our authoritative server. But what if we don't have a domain yet?

### Setting up DNS Update


