---
title: "Architecting to Fail Faster in Kotlin: The Pyramid"
author: "Jacques Smuts"
cover: "/images/compile_test1.png"
tags: ["Kotlin", "Testing", "TDD", "Testing Pyramid", "Android", "Margaret Hamilton", "Architecture"]
date: 2020-03-04T20:48:01+02:00
draft: true
---

Kotlin allows us to structure our code around compile-time tests. This post explains how.


<!--more-->

## Four layered testing pyramid

You probably know the testing pyramid. Unit tests run faster and are easier to develop and run, so you should have more of them, compared to your integration tests and end-to-end tests. However if you assume that this pyramid applies to any sort of automated validation of your code, then the testing pyramid should actually look like this:

{{< figure src="/images/compile_test1.png" alt="Illuminati tested" title="Testing Pyramid+" width="70%"  class="zoomable" >}}

If you're wondering what Compile Time tests mean, I mean everything that runs during and even before compilation, to provide "Compile Time Safety": Static Analysis, Syntax Errors, Linting, Generated Classes, (Gradle) Build Tool Tasks and anything else I might be forgetting. Compile time tests refers to any errors you get that prevent your unit-tests from running in the first place.

I'm sure I'm not the first one to propose this. I looked around and found [a few](https://twitter.com/aarondjents/status/805913874704674816) other [similar ideas](https://twitter.com/mrjedmao/status/1085750574996312064), but only really in the JavaScript community. My guess is that since they cannot rely on the compiler by default, they consider the addition of static tests to be part of the testing framework. They don't take the concept of static verification for granted the way Java/Kotlin developers do.

## Fail Fast Verification

Leaving the pyramid aside, I knew that the concept had to be much older than 2016. I did some research and found [this amazing paper](https://www.sciencedirect.com/science/article/pii/0164121279900049?via%3Dihub) by Margaret Hamilton. If you don't know Margaret Hamilton, all you have to know is that she's able to ship bug-free code, like some kind of mad scientist. You need to read the full article, but for our purposes I'll sum up the relevant parts. Her proposal is that a software system should be verified in this order:
1. Conceptually
2. Statically
3. Dynamically

The static verification at the time was extremely limited, so they had to build a lot of it themselves. Margaret Hamilton took "fail fast" to its logical conclusion and did everything in her power to pick up problems before they became problems. She also designed and implemented end-to-end testing systems, but preferred to have the majority of the validation be done before the end-to-end tests ran. Furthermore, her analyses concluded that bugs are less likely to occur if you have modular systems, as well as extensive documentation, specifications and conceptual planning in place before coding starts.

So with that background in place, let's get back to the four-layered testing pyramid.

## Four layered testing pyramid, in Kotlin

{{< figure src="/images/compile_test1.png" alt="Illuminati confirmed" title="Testing Pyramid+" width="20%"  class="zoomable" >}}

You should be aware that there are [some variations to and criticism of](https://martinfowler.com/articles/practical-test-pyramid.html) the original testing pyramid, but that it makes a good rule of thumb for architecting your processes and systems. When you search for blogs or literature about how the testing pyramid fits into software development, you will find little-to-zero mention of static analysis or compiler checks. Yet if you're coding in Java/Kotlin your IDE is constantly running automated checks on your code that most people consider to be completely separate from their automated tests. **It is important to consider compile-time tests as just one part of a bigger system of automated software validation.**

Considering that every function is tested for spelling and reference, for number of arguments, for return type, and various other checks by the compiler, I think it is fair to say that the number of verifications performed by the compiler exceeds those performed by your unit-tests.

## The Implication On Your Daily Process

Test-Driven-Development is meant to teach you to architect your code in such a way that makes it easy to test. First you write a failing test, then you make that test pass. The first step, according to TDD, is to have a failing test that looks something like this:

{{< figure src="/images/compile_test3.png" alt="normal test" title="First Failing Test?" width="70%"  class="zoomable" >}}


But I see this as the first failing test:

{{< figure src="/images/compile_test2.png" alt="static test" title="First Failing Test" width="70%"  class="zoomable" >}}

This seems like a dumb distinction to make, but when you change the number or type of arguments for the `convertStringToInt` function, your tests don't fail; the compilation fails first.


## The Implications On Your Architecture

Not everyone uses TDD, but everyone should architect their Android projects in such a way that it is easy to unit-test. By going down this route you will hopefully start to implement some of the popular and sensible choices:
 - Modularization
 - Inversion of Control via [Dependency Injection / Service Locator](https://www.martinfowler.com/articles/injection.html)
 - Decoupling Classes / Dependencies
 - [Functional Reactive Programming](https://old.reddit.com/r/androiddev/comments/9ifv54/are_kotlin_coroutines_really_going_to_replace/)

 And various other small techniques that you develop subconsciously as you prioritise around testability.

 However, what if you prioritise your architecture to go beyond failing fast in unit tests? What if you prioritise things to fail fast like in the 4-layered test-pyramid above? Breaking changes should ideally fail at compile-time. For me, as an Android Developer, this is the ideal. My compile-time is often around a full minute and I'd prefer if breaking changes broke things before I even run my tests.

A simple example would be if my `convertStringToInt` function required an additional argument. I'd get this error:

{{< figure src="/images/compile_test4.png" alt="what a terrible function I should just use the standard library for this" title="Perfect test: it failed before it even ran" width="70%"  class="zoomable" >}}


## "Okay, cool. So give me some practical examples"

This was the theoretical portion. In the follow-ups to this post, I'll give some practical examples, which may include:

- [Using named arguments]({{< ref "/post/compile_time_tests2" >}})
- [Using custom lint-tests]({{< ref "/post/compile_time_tests3" >}})
- [Using `when` with enums/sealed classes]({{< ref "/post/compile_time_tests4" >}})
- [\<`reified Type`\> Generics instead of <\*>]({{< ref "/post/compile_time_tests5" >}}) (https://www.zacsweers.dev/api-design-case-studies-intersection-types/)
- [Labels for `this` and scope]({{< ref "/post/compile_time_tests6" >}})
- [Using coroutines for asynchronous operations]({{< ref "/post/compile_time_tests" >}})
- [Using libraries that generate interfaces]({{< ref "/post/compile_time_tests" >}}) (View Binding, NavigationSafeArgs, Dagger, Apollo)
- [Using Kotlin Gradle DSL]({{< ref "/post/compile_time_tests" >}})
