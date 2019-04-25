---
title: "Using a Generic Interface and Method in Kotlin"
author: "Jacques Smuts"
cover: "/images/Generic Interface and Method.jpg"
tags: ["android", "kotlin"]
date: 2019-04-25T20:35:09+02:00
draft: true
---

With Kotlin, you can easily pass an Interface and its associated method to a class. This article explains how.

<!--more-->

https://kotlinlang.org/docs/tutorials/kotlin-for-py/member-references-and-reflection.html

So let's assume you have several different Interfaces. If you're an Android developer, you're probably used to Retrofit and its associated Interfaces. You may have several classes that look like this:

{{% gist "JacquesSmuts" "8374926ea79785b96aec53714d1dd44e" %}}

And then you have a few clients which may or may not implement these interfaces.