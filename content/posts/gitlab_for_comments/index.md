---
title: "GitLab for Comments"
date: 2023-02-08T03:09:29+01:00
draft: true
weight: 0
tags: ['GitLab', "API Shizzle", "Insomnia (Software)"]
summary: ""
cover:
    image: "img/header.webp"
    alt: "Recreation of the 'I made this' meme (sorry that probably doesn't help if you're blind)"
    caption: "This meme was originally made in 2013 by Tumblr user nedroidcomics, bet you didn't know that."
    relative: true
---

Recently I came across the project {{< extlink url="https://github.com/utterance/utterances" text="Utterances">}} on GitHub. I don't exactly remember on what blog it was (but I love you, forever.) but it made me wonder if the same could be done with GitLab.

To make it a little easier on myself I want this to be a one-way street, people can comment on an issue on GitLab, but there won't be any fancy widgets offering up comment capabilities on the site itself. Engagement be damned I guess.

## The GitLab API
The GitLab REST API {{< extlink url="https://docs.gitlab.com/ee/api/rest/" text="is pretty well documented" >}} but I did run into a silly little thing during my first poke. Because I wanted to do this through issue comments I went to the documentation on the `Issues` resource and to the `Comments on issues` subsection ({{< extlink url="https://docs.gitlab.com/ee/api/issues.html#comments-on-issues" text="here" >}}) you can find the text "Comments are done via the notes resource." with a link to the Notes API resource.

First, let me note that projects/:id/issues/:iid works fine without providing an API key but the /notes on that issue will give you a `401 Authorized` response. 

{{< callout type=info >}}
Do mind that :iid is not a typo, it is GitLab's way of differentiating global IDs (:id) and context specific ID's (:iid). This is the reason that the first issue on your project can have ID 1 and so on.
{{< /callout >}}

So for this plan to work we'll need an API key. In this case I decided to use a Project Access Token, this does restrict an instance of our code to only server 1 site, which is good enough for now. The reason for a Project Access Token is that we need the full API scope so we can potentially implement automatically adding a new issue. Using a Personal Access Token would increase risk of abusing the key significantly. Guests can manage project issues so that role should be sufficient.

{{< lightbox img="img/gitlab-pat.webp" group="GitLab" center="yes" caption="Adding a new access token for GitLab is pretty easy" >}}

After adding the Access token to our request and requesting the /notes for the issue we get... an array of objects containing each comment. 

Except that there's no feasible way to make connections between the comments. As soon as you do reply to an issue comment nothing changes apart from the parent comment and your new reply suddenly being of type "DiscussionNote" rather than `null`. So the /notes endpoint is not what we want. But the type of the comments does open up a path to figuring this out. In addition to the /notes endpoint you also have the /discussion endpoint which actually does exactly what we want.

Where the output for /notes is
```json
[
	{
		"id": 1269609123,
		// object data omitted
		"commands_changes": {}
	},
	{
		"id": 1269607345,
		// object data omitted
		"commands_changes": {}
	},
	{
		"id": 1269607241,
		// object data omitted
		"commands_changes": {}
	},
]
```

The output for /discussions is:

```json
[
	{
		"id": "7449e541c022b37e537de11570c95f4f8fa25933",
		"individual_note": false,
		"notes": [
			{
				"id": 1269606177,
				// object data omitted
				"commands_changes": {}
			},
			{
				"id": 1269606397,
				// object data omitted
				"commands_changes": {}
			},
			{
				"id": 1269606505,
				// object data omitted
				"commands_changes": {}
			}
		]
	},
	{
		"id": "351ade33e62d09c5faba60c82902c6ca4d5d4e09",
		"individual_note": true,
		"notes": [
			{
				"id": 1269606649,
				// object data omitted
				"commands_changes": {}
			}
		]
	},
]
```

So the output of /discussions is actually pretty good and nice to work with, the addition of individual_node removes the need to check for the length of the array. This does relieve a bit of the rage I felt as I saw how useless the /notes endpoint was.


