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

Then I wrote this blog post using the standard template by running hugo new post/how_i_made_this.md and writing the rest in the resulting .md file. I ran deploy.sh (from the Hugo Guide's instructions) and did not see this post.

So I removed the "draft:true" property from this post, felt like an idiot for taking 30min to figure that out, then deployed again.

Finally, I realized I had forgotten to set the new domain in my config file after deploying. So I fixed that and now everything seems to be working, I think.

---

Nick Rout noticed that the `quote` styling didn't come out right, so I changed that. It also screwed up my normal code highlighting, but I think it's better to use GitHub gists anyway, and I'm too lazy to create my own custom code styling template and shortcode in hugo. I've had some issues with the code highlighting's limited support for Kotlin so I'm basically just pretending it doesn't exist.

Nick also noticed that the images stretch badly on mobile. This was because I was setting a constant height to my images, and the width expanded only up until it hit the edge. So I changed all my images to use width=percentage% and now the scaling is right.

---

For SEO purposes, I also added my site to GoogleAnalytics Console, and registered my site with Google Search Console to claim ownership of the domain according to google. I should now get pushed up on Google Search results a bit, and also get analytics on how many people visit my site or click through based on search terms, if I understand correctly.




I'm putting further tests in here. For example:

### Code Highlighting

{{< highlight java "linenos=table,linenostart=1">}}
inline fun<reified T: Any> get(key: String): T? {

    val instance = FirebaseRemoteConfig.getInstance()

    return when (T::class) {
        Long::class -> instance.getLong(key) as T?
        String::class -> instance.getString(key) as T?
        else -> throw IllegalArgumentException("the ${T::class} class is not supported yet")
    }
}
{{< / highlight >}}

Which you'll notice is incorrectly styled with red thanks to the single-quote styling change I made.

### Git Gist


{{% gist "JacquesSmuts" "217c77ec0e4d8ec8409ae45c3516ec07" %}}

### Image with zooming and title

{{% figure src="/images/how_i_made_this2.png" alt="This is my face" title="Obviously, this is my face. Click to zoom." width="40%" class="zoomable" %}}

### Video

{{% video
  "/videos/how_i_made_this3.mp4" %}}

### Embedded Youtube

{{% youtube "TdO8hR2QAUs" %}}