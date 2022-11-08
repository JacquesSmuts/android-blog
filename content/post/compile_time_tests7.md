---
title: "Generated Classes Are More Maintainable"
author: "Jacques Smuts"
image: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "Generic", "labelled scope"]
date: 2023-06-15T14:59:23+02:00
draft: true
---

Using generated classes can make your code safer. This post highlights some great examples.

This article is Part 7 in the [series on making your code compile-time safer]({{< ref "/post/compile_time_tests" >}}).

<!--more-->

## Generated Classes?

Any library or plugin that generates type-safe classes for you to interact with, from a schema or other class definitions. Any example of that would be [GraphQL](https://graphql.org/). The [Android Library](https://github.com/apollographql/apollo-android) for the Apollo GraphQL client generates strongly typed classes based on your schema and configuration, and forces you to make queries and mutations to the GraphQL endpoint via that system. This means that no one can ever accidentally change a class that will crash when the server returns a valid response. Assuming that your Schema is updated and the library is well tested (it is), you will never run into an issue where the api call is failing due to bad definitions or types.

After using GraphQL, I realised that code generation is one of the best ways to make sure that code fails quickly and easily when breaking changes are made.

So here's a list of some Android libraries or plugins that generate type safe classes/interfaces and/or tests.

## Libraries that generate

- [Apollo GraphQL](https://github.com/apollographql/apollo-android)
- [Retrofit](https://square.github.io/retrofit/)
- [SqlDelight](https://github.com/cashapp/sqldelight)
- [Room Database](https://developer.android.com/training/data-storage/room) (I prefer SqlDelight, but both generate typesafe classes)
- [KTX Parcelize](https://medium.com/@BladeCoder/a-study-of-the-parcelize-feature-from-kotlin-android-extensions-59a5adcd5909)
- [SafeArgs](https://developer.android.com/guide/navigation/navigation-pass-data)
- [ViewBinding](https://developer.android.com/topic/libraries/view-binding)
- [Dagger](https://dagger.dev/)

I personally dislike writing annotations above my methods and variables. I think annotations look a bit ugly, but I'm willing to make that sacrifice if it means my code is safer at compile time. I'm more used to Kodein/Koin, but I'm planning to move over to Dagger for the primary reason that it generates classes that result in compile-time errors instead of runtime errors.

## Surprise, this article is about DI now

I know nothing about what makes a dependency injector good or not. I'm basically on board with the basics laid out by Fowler in [his post on this topic](https://martinfowler.com/articles/injection.html#ServiceLocatorVsDependencyInjection), but honestly that whole discussion is outside the scope of this article series. Remember, this is [article 8 in the series]({{< ref "/post/compile_time_tests" >}}) on making your code safer at **COMPILE TIME**. I personally prefer Koin for personal projects because it's got an easy setup and still allows for easy (semi-manual) dependency injection. However, for the purposes of achieving the singular goal of compile-time safety, Dagger wins. I basically played myself by writing this series and having a personal preference for compile-time safety, because now I don't have a choice but to switch over. The DI you choose will have to be based on whether your project prioritises ease-of-use, safety, or some other metric.

## Conclusion

**Libraries that generate code can make your projects safer.** If a problem has been solved by using generated code, consider using that.
