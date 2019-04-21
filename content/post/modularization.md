---
title: "Is your modularization making any progress?"
author: "Jacques Smuts"
cover: "/images/modularization1.png"
tags: ["android", "modularization", "kotlin"]
date: 2019-04-20T21:15:17+02:00
---

You should modularize your Kotlin Android app, but you will run into some issues. Here's some tips to help.

<!--more-->


## Am I doing it right?

You're probably doing it mostly right, but how do you know? Here's a good way to check. Generate a dependency graph for your app. I used [APK Dependency Graph Generator](https://github.com/alexzaitsev/apk-dependency-graph). All you need to do is compile the apk-dependency project, point it to an apk, and it will generate an interactive graph like the one below.

{{% figure src="/images/modularization2.png" alt="a dependency graph" title="If you click the link above, you should see something like this." height="400" class="zoomable" %}}

[Click here to see PositiviTea's dependency graph]({{% ref "/sites/dependency_positivitea_011" %}}) It's interactive and fun, [go ahead.]({{% ref "/sites/dependency_positivitea_011" %}})

This graph demonstrates a way to look for ease of refactoring with two nice examples:

 - The ServerClient in the bottom right provides a complete separation between the api and the rest of the app. The api calls can therefore be refactored easily.
 - Lots of classes are reliant on the TeaBag class in the middle. A single change there is likely to cascade outward, making refactoring difficult.

Below, an example of a project which is not easy to refactor, update or even debug:

{{% figure src="/images/modularization3.png" alt="a dependency graph" title="An unnamed legacy project." height="400" class="zoomable" %}}

There is no separation of concerns here. This project is in dire need of modularization, but where do you even begin?

## Where to begin?

I usually follow these steps:

 1. [Create a module](https://developer.android.com/studio/projects/android-library)
 2. Identify one or more classes that should not be accessible outside of a specific context
 3. Move those classes into the module
 4. add the `internal` [visibility modifier](https://kotlinlang.org/docs/reference/visibility-modifiers.html)
 5. Try to compile and get an error
 6. Fix this error without removing the internal keyword. This may involve creating a little bit of boilerplate.

{{% figure src="/images/modularization4.png" alt="compiler error" title="If you see this warning, you're headed in the right direction." height="150" class="zoomable" %}}

A specific example of this would be the database module. I moved my `database` class, `data model` classes, `DAO` classes, and `repository` class into the database module. I made everything `internal` except for `repository`, which provides the API to the database for the rest of the app. After some restructuring and cleanup, I ended up with a nicer class structure.

{{% figure src="/images/modularization5.png" alt="a dependency graph" title="Dependency graph, but after internalizing the database module" caption="It seems similar to the one above, but with some subtle differences." height="400" class="zoomable" %}}

Okay, I have to admit, it doesn't seem like we made that much of a difference. But here's the real benefit:

## The real benefit

#### When a new developer works in a non-database module and tries to access the database directly, the compiler will stop them.

The two dependency graphs are similar, except that a moment's thoughtlessness can pull everything into spaghetti in the first instance. I have used the `internal` modifier in my database module, which means that a developer would be stopped from breaking a delicate separation of concerns with a single function.

{{% figure src="/images/modularization6.png" alt="compiler error 2" title="Trying to access the database directly from MainActivity.kt." height="100" class="zoomable" %}}

## Conclusion

We have looked at two ways in which to analyze the state of your modularization:

1. APK-Dependency-Analyzer; and
2. The prevalance of the "internal" modifier

I'll probably write some more blogs later, when I run into more issues or noteworthy tips. Modularization can be tricky, but it's worth it.
