---
title: "Dutchguide part 1: Choosing a CMS"
date: 2024-09-25T22:23:18+02:00
draft: false
weight: 0
tags: ["Directus",  "Dutchguide", "Hugo", "Projects"]
summary: "In this one I braindump about Directus mostly. And I describe a new project I'm working on and how I'm incorporating aforementioned tool. :)"
thanks:
    icon: "fa-solid fa-heart fa-3x"
    title: "Thanks for reading"
    text: "If you have any questions or feedback feel free to contact me through the means listed [on my main site](https://dylanmaassen.nl). Sharing my posts is also really appreciated!"
---
So once again it's been a hot sec, but I started a new project I am pretty stoked about. And not just about the project itself, but I also found a few new toys to play with.

As a quick aside, I've got rid of the header images for the posts. They're pretty unnecessary, and I want to spend more time actually writing the text.


## Mission statement
My personal brief for the project is "an independent multilingual guide to life in the Netherlands" because I don't believe something like that exists quite yet.  
A lot of government sites are just about starting to get translated in anything other than Dutch and a lot of key information is simply not available.  
Also, and this pisses me off immensely, there's a few commercial companies offering "guides" but they are heavily peppered with advertisements for their own services.

I started an early version of this site, I'm dubbing it Dutchguide, earlier this year and I wanted to fully create it in Hugo originally. However, I can't/don't know everything there is to know or keep track of everything all the time.
Ideally I want to get other people to work on content for the site as well. And you can't realistically expect everything to whip out a code editor and start working with Git.

## On "traditional" Content Management Systems
From previous jobs I have pretty extensive experience with the Content Management Systems Drupal and WordPress. I personally really don't want to bother with either of them much less create themes for them fitting the content.
For actually building the site I want to stick with Hugo as site generator, and I've been reading up on how to pull content from an API and generate pages from it.  

Both Drupal and WordPress can function as "headless" CMSs, Meaning you still create the content in their admin interface, but you generally access it through an API. 
But both of them have a lot of moving parts and generally need extensions to get them set up the way you want them to. Even if it's just adding custom fields.  
And don't get me started on having to update both the core system and said extensions and generally PHP as well.

## Enter Directus
Now, don't get me wrong. There are plenty of headless CMS systems. But Directus has my heart for now. 

Setting up the data model does feel like you are pretty much just working on the underlying SQL database. But it allows you to set up a very friendly user interface on top of that with exactly the fields and restrictions you want.  
The way of setting up relations between fields and collections is also quite intuitive.

As shown here you can easily add a Many-to-One field.
{{< lightbox img="img/PopularGargantuanAoudad.png" group="directus_manytoone" center="si" h=500 caption="As shown here you can easily add a Many-to-One field" >}}
That'll show up as a proper field when creating or editing an article. In the field settings you can also let users create new items on the fly. But with website sections I want to limit that ability, so the site can stay organized.
{{< lightbox img="img/FluffyAustereBluewhale.png" group="directus_manytoone" center="si" caption="That'll show up as a proper field when creating or editing an article" >}}

And when clicked you get a list of all items in the "sections" collection. This overview can also be customized by the way. Any field in the sections collection or any linked field can be displayed in the interface.
{{< lightbox img="img/EqualDarkslateblueIberianmidwifetoad.png" group="directus_manytoone" center="si" caption="And when clicked you get a list of all items in the 'sections' collection" >}}
Because this is a Many-to-One relation you only get to pick one section. But you could easily create a Many-to-Many relation in which case Directus will automatically create a linking collection (table).

Of course a relation can work the other way around as well, for example as a One-to-Many field on Sections that you can use to get all the Articles within one Section.
{{< lightbox img="img/UrbanImpoliteGelada.png" group="directus_onetomany" center="si" >}}

Either way, Directus lets you build forms to create and edit your content with whatever fields you see fit, and it has built-in support for content and field translation

{{< lightbox img="img/MysteriousBewitchedCanine.png" group="directus_contentform" center="si" >}}

It also has a pretty granular permission system where you can control exactly which collections or even parts of a collection a user can interact with.
This lets me set up content flows where for example someone could submit a new article, but it would have to pass by an editor to approve and publish it.

Directus also has support for other flows with a plethora of triggers and actions you can implement. One of the flows I'm planning on is letting users start the build process of the site through Directus.
Considering the site will be hosted on Cloudflare Pages I can easily set up a webhook that rebuilds the website whenever the flow is triggered. 

For my use case I have decided to leave the "public" role unprivileged and made a separate role for site generators with permissions to the appropriate collections.  
This allows me to require an access token to read data from the Directus API. In the future I would like to offer access to the content API to other builders and this lays the foundation for that step.
It also keeps unauthorized users from scraping the API or overwhelming it with data requests. 

## To be continued
Some things Directus does might not be very well documented, I've definitely run into my fair share of surprises, and they could definitely do with some more examples. Although they do have a web series on "building x in 60 minutes" where they go in depth on the data models, but it would be nice to have more written examples for less common use cases. I will definitely spend more time working with Directus, hopefully gather some thoughts from regular users and try to improve the experience of working with Directus for everyone.

In part 2 I will be diving deeper into Directus data models and introducing the API. This should be out this week. I've only just started poking at the API, and I'm using a tool you might not have heard of before.