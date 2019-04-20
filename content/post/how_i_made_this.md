---
title: "How I made this"
author: "Jacques Smuts"
cover: "/images/how_i_made_this1.png"
tags: ["hugo", "jacques smuts"]
date: 2019-04-20T10:43:30+02:00
draft: false
---

This details how I made this static github page.

<!--more-->

First, I installed [Hugo](https://gohugo.io/) on my local machine and created a site.
I chose the theme [Hugo-Nuo](https://themes.gohugo.io/hugo-nuo/) because it looked nice and because the author has a github page with regular updates and decent documentation.

It was not crashing, so I deployed it to a Github static page using the official [Hugo Guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) for hosting on Github.

Then I wrote this blog post using the standard template by running hugo new post/how_i_made_this.png and writing the rest in the resulting .md file. I ran deploy.sh (from the Hugo Guide's instructions) and did not see this post.

So I removed the "draft:true" property from this post, felt like an idiot for taking 30min to figure that out, then deployed again.

Finally, I realized I had forgotten to set the new domain in my config file after deploying. So I fixed that and now everything seems to be working, I think.

