---
title: "Vanity URLs for Go"
date: 2022-05-11T19:24:16+02:00
draft: true
weight: 0
tags: ["Go", "Programming", "Cloudflare Workers", "Hosting"]
author: ["Dylan Maassen van den Brink"]
ShowToc: true
cover:
    image: "img/header.jpg"
    alt: "A gopher eating something with another gopher sniffing the food"
    caption: "Photo by [Lukáš Vaňátko](https://unsplash.com/@otohp_by_sakul?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/gopher?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)"
    relative: true
---
## Background
In Go it is common practice to refer to imported modules by their URL at your Git hosting service of choice. This may for example look like:
```go {}
import (
	"github.com/dylantic/gotest"
)
```

This is all fine and well. It shows where the code is hosted, and you know exactly where you are `go get`-ing the module from. But in my opinion it's just plain boring.  

This is where *vanity* URLs come in, they allow you to refer to code using your own domain. In my case, I'm using go.mozoa.nl. This allows me to import my own modules by using
```go
import (
	"go.mozoa.nl/gotest"
)
```

"How does this work?" I can already hear you ask. Don't worry, it's quite simple and has to do with the way `go get` works.  

Secretly, services like GitHub and GitLab offer an HTML meta tag. GitHub does it by default on any repository page, GitLab requires passing of "?go-get=1" which is how `go get` identifies itself.  

For example, check the page source of https://github.com/dylantic/gotest. You'll see it contains the following snippet:
```html
<meta name="go-import" content="github.com/dylantic/gotest git https://github.com/dylantic/gotest.git">
```

We can use this to our advantage as Go Get doesn't really care where your code is hosted as long as you tell it how to retrieve it. Which is exactly what that tag does.
The structure of the tag is as follows:
```html
<meta name="go-import" content="import-prefix vcs repo-root">
```
- **import-prefix** - is either the full path of the module or a prefix. In this post I'll only be going over full paths.
- **vcs** - the version control system in use for the repository housing the module. (bzr, fossil, git, hg, or svn)
- **repo-root** - this is a weird one, according to the documentation it's supposed to be "the root of the version control system containing a scheme and not containing a .vcs qualifier.". However, you'll notice both GitLab and GitHub do include a 

**vcs** can also have the value 'mod' which is used to refer to a [module proxy](https://go.dev/ref/mod#goproxy-protocol). I will not be dealing with module proxies in this post either.

## Enter Cloudflare Workers
So let's get into actually setting up a system to return that XML. As you get 100k requests per day on their free plan, Cloudflare Workers are a pretty good deal. Unless you're already running a webserver, of course.

***This post assumes you already know some Cloudflare basics and that your domain has already been added to their service.***
