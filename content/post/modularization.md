---
title: "Lessons I learnt from modularizing everything, 1"
author: "Jacques Smuts"
cover: "/images/modularization1.png"
tags: ["android", "modularization", "room", "kotlin"]
date: 2019-04-20T21:15:17+02:00
draft: true
---

You should modularize your Android app, but you will run into some issues. Here's some tips to help.

<!--more-->


## Am I doing it right?

You're probably doing it mostly right, but how do you know? Here's a good way to check. Generate a dependency graph for your app. I used [APK Dependency Graph Generator](https://github.com/alexzaitsev/apk-dependency-graph). All you need to do is compile the project, point it to an apk, and it will generate an interactive graph like the one below.

{{% figure src="/images/modularization2.png" alt="a dependency graph" title="If you click the link above, you should see something like this." height="400" class="zoomable" %}}

[Click here to see PositiviTea's dependency graph]({{% ref "/sites/dependency_positivitea_011" %}}) It's interactive and fun, [go ahead.]({{% ref "/sites/dependency_positivitea_011" %}})

This graph demonstrates ease of refactoring with two nice examples:
 - The ServerClient in the bottom right provides a complete separation between the api and the rest of the app. The api calls can therefore be refactored easily.
 - Lots of classes are reliant on the TeaBag class in the middle. A single change there is likely to cascade outward, making refactoring difficult.

Below, an example of a project which is not easy to refactor, update or even debug:

{{% figure src="/images/modularization3.png" alt="a dependency graph" title="An unnamed legacy project." height="400" class="zoomable" %}}

There is no separation of concerns here. But where do you even begin?

## Where to begin?

Just identify one class or folder that should not be accessible to most of the app, move it into a module, and add the "internal" [Visibility modifier](https://kotlinlang.org/docs/reference/visibility-modifiers.html). Some good classes to start with are, api call internals, database management, or ui-only elements. In my case, I moved all of the database-related classes into a database module, and made my database and DAO classes internal. 

Next, try to compile and see what happens.

{{% figure src="/images/modularization4.png" alt="compiler error" title="If you see this warning, you're headed in the right direction." height="150" class="zoomable" %}}

Then you have to fix this somehow. Often times, that involves creating a little bit of boilerplate, and re-using the same classes 