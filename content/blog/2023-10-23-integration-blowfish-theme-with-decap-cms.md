---
title: Integration Blowfish Theme With Decap CMS
date: 2023-10-23T18:48:39.151Z
description: A journey in making this blog
draft: false
showHero: true
thumbnail: /img/blowfish.jpeg
tags:
  - tech
  - blogging
  - hugo
  - decapcms
---
This is the story of how I integrated Blowfish theme in Hugo with DecapCMS. 

I love how Hugo builds in nano seconds and serves static HTML without extra JavaScript or network calls to a database. It offers improved SEO and a better user experience. What I don't like is that to make changes, I would often need to open up VIM on the computer with my Git repo. I want to be able log into an admin site and make edits and posts from a web browser a la WordPress style. But what I *don't* want is to manage a MySQL instance and keep all my site data in MySQL - or any database for that matter (other than git, technically a database 😅). This is where DecapCMS comes in.

{{< github repo="decaporg/decap-cms" >}}

DecapCMS, formerly NetlifyCMS, is a git based CMS. It offers a web interface to commit markdown files to my git repo - and generate my site, using Git as the single source of truth. DecapCMS [has instructions for integrating with Hugo](https://decapcms.org/docs/hugo/), but with Hugo there's also a need to integrate with whatever hugo theme your using. I'm using Blowfish theme:

{{< github repo="nunocoracao/blowfish" >}}