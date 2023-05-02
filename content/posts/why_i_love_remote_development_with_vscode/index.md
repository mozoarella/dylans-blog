---
title: "Why I love remote development with Visual Studio Code"
date: 2023-05-02T09:57:50+02:00
draft: true
weight: 0
tags: []
summary: ""
cover:
    image: "img/header.jpeg"
    alt: "Header image containing the text 'Why I love remote development with Visual Studio Code'"
    relative: true
---
## Tell me what you want
I'm not a professional software developer, but even for IT infrastructure stuffs a good code editor is indispensable.
My current setup has a combination of my favourite things in life. It's effortless to add features on the fly, it supports pretty much any language I'd want to dabble in, and best of all, it's completely free. Of course these are just features of Visual Studio Code out of the box.

However, there is 1 feature of VSCode that makes me love working with it and, dare I say it, actually gave me more joy in programming: Remote Development over SSH.  

Being able to connect to a remote server over (S)FTP in your editors has been a feature for forever but that still brings the pain of having to manage your local environment. What is different with remote development with VSCode? VSCode sends a headless version of itself over to any server you have SSH access to, and it allows you to manage and run your extensions remotely. Your local version of VSCode is then free to just open any folder on the remote server and continue developing on any device you set up to connect.

This has a few advantages:
- You only have to setup your environment once
- It's easy to offer a new developer a space to work with the same tooling already installed on the server.
- You can access your development environment from any device that'll run VSCode,
- Your local machine stays clean, your code and all of its dependencies live on the remote server.

My favourite part is that you don't even have to check everything into git all the time when you switch devices. You just log in and your code and bash history is all there. Of course that doesn't mean you shouldn't commit your code frequently, but it makes it easier to continue working.

## So what you need?
You'll need a machine, virtual or otherwise, that you can SSH into. This doesn't have to be Linux, but I only have experience doing it on Linux. Once you've got that set up you're done with the prerequisites.

Okay if you actually want to work it's also useful if you can install software on the remote server, or you could befriend the admin that can do it for you.

## So how you gonna do it?

Once you have a server setup all you need is the [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension which will install the Remote Explorer and SSH: Editing Configuration Files extensions automatically.

{{<callout type="info">}}
While the extension does support password authentication it is highly recommended you set up SSH Key authentication.
{{</callout>}}

I'd recommend either adding the new remote location through the remote explorer or manually through `~/.ssh/config`. For example, my `~/.ssh/config` looks like this:
```text
Host homedev.dy.lan
	HostName homedev.dy.lan
	User dylan
	ForwardAgent yes
```

- The `Host` field is just a name for the entry.
- `HostName` should be either a hostname or an IP address.
- `User` lets you define the user to connect as (useful when your local username is different).
- `ForwardAgent` is entirely optional but lets you forward your local SSH agent to the remote server.

{{<callout type="info">}}
VSCode knows how to load your keys when they're in a default location such as `~/.ssh/`, however, this is not the same as an SSH agent you can forward. You'll need to set up the SSH agent and load your keys into it yourself. This can be automated, but the method is dependent on your local operating system.
{{</callout>}}

After your SSH config is setup, reload the remote explorer if your host doesn't show up and then click your host. If all is well VSCode will connect and automatically install the remote server in the directory you're in when you log in on the remote host. This would be your home directory by default.