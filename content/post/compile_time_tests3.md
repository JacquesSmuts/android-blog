---
title: "Custom Lint Warnings Keep Your Code Safer"
author: "Jacques Smuts"
image: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "Custom Lint"]
date: 2023-03-09T18:50:56+02:00
draft: true
---

Making your own lint-tests is one of the most powerful ways to control your codebase. This post shows you why.

<!--more-->

This article is a follow-up to [Part 2 in this series,]({{< ref "/post/compile_time_tests2" >}}) wherein I explain why named arguments are great for catching breaking changes.

## What's a Lint Warning?

When your compiler gives you a warning or error before compilation happens. Like this:

{{< figure src="/images/compile_test5.png" alt="Liiiiiiint" title="Timber's Lint Warning" width="80%" >}}

I'm using this example because Timber was the first library that showed me that you can write your own custom Lint Warnings. So I copied Timber's implementation for the [PreferencesHelper](https://github.com/flatcircle/PreferencesHelper) library (and other libraries) I built.

## Can I create my own Lint Warnings?

Yes.

## Should I create my own Lint Warnings?

Making the lint warnings work for PreferencesHelper was a nightmare. Testing it is extremely difficult because your tests will pass but then it won't appear in the IDE. Or it will compile but then do nothing. Or you'll have the wrong combination of Gradle and Lint versions and nothing will happen at all. Or it will work perfectly until you try to export the .jar file into JitPack() then it will do nothing.

Yes, all of these are what I experienced when trying to build the [NamedArgsLint Library](https://github.com/JacquesSmuts/NamedArgsLint/blob/master/README.md), as well as the PreferencesHelper library. Custom Lint Warnings are extremely experimental right now. They're badly documented and confusing, unstable and difficult to verify.

I know that this may come across as negative, but I feel strongly that you need to be emotionally prepared for a programming challenge that will not be resolved as quickly, easily or predictably as other projects you've worked on.

## Should I create my own Lint Warnings?!

Yes.

## Why?

There are often patterns in code that can be problematic or difficult to promote, and each codebase is unique. You will notice some of your own scenarios, such as:
- Whenever we access Vector Drawables in a certain way, certain devices give issues
- We use ThreeTenAbp or JodaTime and should never allow anyone to use the java.util.Date class
- Due to night-mode or Theming, certain colours should never be accessed directly
- We have a special Custom TextView and whenever someone implements a normal TextView someone has to remember to point it out in code review
- We decided to make sure BuildConfig.DEBUG always goes through a wrapper and we don't want BuildConfig.DEBUG to be accessed directly
- We are using [Safe Args](https://developer.android.com/guide/navigation/navigation-pass-data#Safe-args) and should not allow intent.putExtras for passing arguments
- We are using [View Binding](https://developer.android.com/topic/libraries/view-binding) and should therefore discourage use of findViewById()

All of these situations can be solved with Custom Lint Checks.

**A codebase should be opinionated. When you have an opinion, why not enforce it automatically?**

## How?

Things are constantly in flux, but I'll try my best to keep some references to working examples here.

There's a good [introductory video](https://www.youtube.com/watch?v=wFe0WZm_xm8) by John Rodriguez, wherein he explains some core concepts you need to be aware of. Particularly pay attention to `Psi` and `Uast`.

Alex J Lockwood's [Android Lint Demo](https://github.com/alexjlockwood/android-lint-checks-demo) has some great resources and links in the readme, as well as good samples. If you try to clone his repo and run it it might not work out of the box though, due to weird environment issue on Mac that I can't even remember how I solved it.

The first time I managed to make a lint-test work is to reverse-engineer all the lint-related items on the [Timber Logger](https://github.com/JakeWharton/timber/tree/master/timber-lint) library.

A very important tool in your debugging is to learn about the `asRecursiveLogString()` as explained [in this article](https://medium.com/supercharges-mobile-product-guide/formatting-code-analysis-rule-with-android-lint-part-1-2-4b906f717382)

I would also have recommended cloning my [NamedArgsLint Library](https://github.com/JacquesSmuts/NamedArgsLint/blob/master/README.md) if it worked reliably, which right now it doesn't.

## Conclusion

Custom lint checks are difficult, but powerful. *You never have to explain patterns or rules to someone if you know their IDE will do it for them.*

In [the next post]({{< ref "/post/compile_time_tests4" >}}) we'll look at the `when` expression, and how to ensure that it throws a compiler error at the right time.
