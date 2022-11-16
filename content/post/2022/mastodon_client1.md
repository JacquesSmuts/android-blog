---
title: The Story of AndroidDev.Social's Mastodon Client
description: This is an account of the founding of a new Mastodon Client in November 2022.
slug: mastodon_client1
date: 2022-11-10 00:00:00+0000
image:
categories:
- Android
- Mastodon
- KMP
tags:
  - architecture
  - bad internet
  - old phones
draft: true
---

# Background

Some guy bought Twitter, and made a lot of Decisions and Changes. This has made a lot of people very angry and been widely regarded as a bad move. As a result, many people migrated to Mastodon instances.

One of them was Friendly Mike, who set up [androiddev.social](https://androiddev.social/@friendlymike) as a Mastodon instance, and is doing great work managing that, periodically sending out invites, and updating everyone on the work he's doing.

{{< stoot "androiddev.social" "109314476066026798" >}}

His work, and the fact that so many android developers joined and supported the creation of this instance, is a wonderful story all on its own. Consider supporting the server @[donate.androiddev.social](donate.androiddev.social).

On AndroidDev.social, he, and many others, asked this question:

{{< stoot "androiddev.social" "109294448020649297" >}}

And then Omid set up a new slack and GitHub repo for the creation of a Kotlin Multiplatform Mastodon client.

{{< stoot "androiddev.social" "109309107328541642" >}}

There are multiple good iOS and Android apps out there for Mastodon. I personally recommend [Tusky.app](https://tusky.app/). I have no idea whether this app will be better than any of the existing Mastodon apps, but for me (and I suspect others) this is more a fun project to use as an excuse to expand my knowledge of KMP, Compose, and various related technologies.

# The Repo Starts

The repo [is hosted on GitHub here.](https://github.com/AndroidDev-social/MastodonCompose)

I've never been a part of big open source project from the start. I've contributed tiny bits here and there to some open source projects, usually documentation, but I'm honestly extremely intimidated. But, despite feeling like I don't belong amongst these experts, I'm just throwing myself in there to see if I can be helpful in any way. 

{{< figure src="/images/2022/mastodon1.png" alt="list of issues" title="Issued opened all within like a day. Several more were closed and moved to Discussion. " width="50%"  class="zoomable" >}}

# Things I learnt so far

By taking part in the discussions, and going through the commits, I already learnt a few things.

### Linting

I learnt that Detekt has autoformatting based on ktlint, as [@racka98](https://github.com/racka98) demonstrates [here.](ReluctApp/Reluct@c9de419/build.gradle.kts#L80)

### Toml file for Version catalogue

I learnt you can use a .toml file for gradle dependency versioning, as [Omid demonstrates here.](https://github.com/AndroidDev-social/MastodonCompose/commit/755deab1a229e30d4c7a3ed0f6f17355927eba60#diff-697f70cdd88ba88fe77eebda60c7e143f6ad1286bca75017421e93ad84fb87df)

### Gradle TypeSafe Project Accessor

Gradle has a feature preview that allows you to access projects in your codebase in a typesafe way, as [@Rafsanjani](https://github.com/Rafsanjani) demonstrates [in this PR.](https://github.com/AndroidDev-social/MastodonCompose/pull/42)

### Mermaid

I learnt about [Mermaid-js](https://mermaid-js.github.io/mermaid/#/), which uses markdown in GitHub to show a diagram, and it's rad. It's how the graph is rendered on the readme of the MastodonCompose repo.

{{< figure src="/images/2022/mastodon2.png" alt="mermaid graph" title="A graph in mermaid that isn't rendering correctly" width="50%"  class="zoomable" >}}

I also learnt that cool tools like these can sometimes give issues and waste the time of even experienced people like the ones running this repo, resulting in them just putting it aside temporarily. That was very helpful for me to see, because I've gone through this process with other tools so many times myself, and I feel like a failure every time. Especially if the tool is giving me issues and I put it aside temporarily, and then just never come back to it.

It's nice to know that it's normal to struggle in this way.

### Accessing files on KMP for tests

The normal way I usually test api calls is with .json files that contain the typical http responses. In the test, we then access that .json file using (on Android) a classload. However this does not work on kotlin native, and there is in fact nothing out of the box you can do to just simply access a file for a test like that. So, I discovered [Icerock Moko Resources Generator](https://github.com/icerockdev/moko-resources). 

However, I struggled to implement Moko Resources Generator. So, taking inspiration from the above section, I abandoned it, and decided to just define the json response in a kotlin object until such time that we can fix the accessing of resources for tests.

### Accessing files on KMP for tests, 2

At this point I'm intimidated and scared. After being politely corrected on my factual mistake, I just wanted to leave in embarrassment. However, I forced myself to stick around. A few hours later, people were discussing Moko Resources Generator in the slack. So I went back and tried to make it work again, and opened a PR after some struggles.

### Testing cross-platform is tricky

I'm not very tapped into KMP development, but this one took a while to internalise.

We have api calls in `commonMain`. However, `commonMain` is agnostic about things like file-access and localisation, making it difficult to do those things from `commonTest`. Adding `moko resources` doesn't just give you access. A simple solution I found was to run the tests on `desktopTest`, and assume it would also work on the other platforms (android, ios)

This is not really correct, and it raises a lot of interesting questions about testing for cross platform development. Should you test everything in `common` from every supported platform? Sometimes it's necessary to make sure all your platforms can run your code, but other times it could result in you maintaining a few different sets of semi-identical tests for a single class. 

I'm sure there's lots of disagreements about where to draw the line, as well as solutions that run the same test on different platforms automatically. I'm excited to learn more about this stuff.

### Compiling cross-platform is tricky as well

It turns out my PR doesn't work on a Mac, since I wasn't using a Mac to build and open the PR.

Coding cross-platform is made doubly difficult if not everyone is using the same build tools.

Luckily, we're doing this together as a group, and helping each other figure out this stuff.

### Psychology can be an obstacle

Imposter syndrome is something I continually struggle with, and it definitely reared its head during this whole exercise. More than a hundred people joined the slack for this project, to help out and/or observe. And every time there was a tricky topic, one person out of that hundred knew exactly what they were talking about and had interesting relevant things to say.

For me, this created the effect that the "Channel" knew so much more than I ever could, and that I didn't belong amongst all these experts. However, I had to keep reminding myself that it wasn't that everyone always knew everything, it's that one or two people out of that hundred were likely to be very knowledgeable about any one topic.

Further: Even assuming that everyone else is much smarter and more experienced than me still doesn't invalidate my presence. Just because I know less than the other contributors, doesn't mean that I'm not helpful towards the project goals. Everyone was being accommodating and thankful for any bit of work, and I'm starting to realize just how powerful the "culture" and contribution guidelines can be for open source projects.

# The direction of the project

There is a lot of [discussion](https://github.com/AndroidDev-social/MastodonCompose/discussions) about various tricky topics, including Naming, Dependency Injection, Mocking/Faking, Pagination, Navigation, Licensing and minSdk. It's a nice discussion page to look for, because there is a lot of interesting things to learn about each of those topics, as well as about how to professionally discuss and resolve differences.

Some of those discussions are coming to a close now, and the architecture and overall shape of the project is solidifying. Work has begun on the first user-facing feature: the ability to log in. This seems to be one of those "go slow to go fast" projects, where things are discussed and planned, instead of the "move fast and break things" approach. I'm a big fan of proper planning and slow (democratic) discussion instead of just barging forward, so I'm enjoying this a lot.

In the next post, I'll write a nice summary of the discussions, and various decisions made throughout the project.

# Final Word

Open Source is awesome. Working together with strangers towards a common goal is uniquely powerful. There is no way I'd learn this much from a few days at work, even though I love my job. Further, if this project works out I'll have contributed a little bit towards building something free that people use to connect with each other on an ad-free platform. That's a nice feeling.

## Additional Thanks

Thank you to Bryce Wray for the [mastodon static embedding shortcode](https://www.brycewray.com/posts/2022/06/static-mastodon-toots-hugo/). It's not displaying as nicely as it does on his example yet, because I suck at css styling and making it work nicely with my theme, but it works 😁.
