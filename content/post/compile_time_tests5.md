---
title: "Architecting to Fail Faster: Reified Types"
author: "Jacques Smuts"
cover: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "Generic", "Reified Type"]
date: 2020-03-30T18:07:56+02:00
draft: true
---

Using the reified Generic type in Kotlin, your code can be much safer.

https://www.zacsweers.dev/api-design-case-studies-intersection-types/

<!--more-->

This article is Part 5 in the [series on making your code compile-time safer]({{< ref "/post/compile_time_tests" >}}).

## What is reified type?

As explained in the [kotlin documentation on it](https://kotlinlang.org/docs/reference/inline-functions.html), the reified keyword allows you to use Generic functions and also access the type of the Generic class.

## Why is reified type?

Having access to the class type of a Generic input at compile-time means that you can make your code type safe and ensure that breaking changes to your code structure will result in compile-time errors.

I consider this to be one of the many reasons why Kotlin is the best programming language, even though I have very limited experience in languages other than C# and Java. I've written about how Kotlin can solve [certain problems in a typesafe way]({{< ref "/post/generic_interface_and_methods" >}}) before, but let me just give a simpler example, given the context of this series on Architecting to fail faster.

## Before Generics

Without using generics, if you want to interpret a string and return multiple types of classes you'd need to do something like this:

{{< kotlin >}}

data class User(val name: String)

//sampleStart
fun interpretString(input: String): Any {
    return when {
        input.startsWith("name=") -> User(input.substring(5))
        else -> throw IllegalArgumentException("This input is not supported: $input")
    }
}
//sampleEnd

fun main() {
  println("Enlightenment rationality started off as like a way to banish the mystic tyranny of feudalism and allowed people to organise their lives with reason and logic instead of... appeals to superstition. But over time it became an edifice of of justifying the rule of the ruling elites and the managerial class who have absorbed those types of reasoning.")
}

{{< /kotlin >}}

And then you'd have to cast the result to the right type, which is unsafe. Generally it'd be better to just have a different function for each different type of return. However with generics we can make it typesafe.

## With Generics

With generics, you can do the same, but you can cast it to the right type inside the function. Now you can use tests to ensure that the function returns the right type.

{{< kotlin >}}

  data class User(val name: String)

  //sampleStart
  fun <T> interpretStringT(input: String): T {

      return when {
          input.startsWith("name=") -> User(input.substring(5)) as T // still casting, but we can test this function
          else -> throw IllegalArgumentException("This input is not supported: $input")
      }
  }
  //sampleEnd

fun main() {
  println("It puts this patina of logic on just a huge mass death machine.")
}

{{< /kotlin >}}

This is a step in the right direction, but we want to reach the bottom level of the 4-level testing pyramid. We want the compiler to tell us if we are returning the wrong type, before the tests even run. Can we do that?

{{< figure src="/images/compile_test1.png" alt="Illuminati confirmed" title="Testing Pyramid+" width="20%"  class="zoomable" >}}

## With Generics And Reified Types

With reified types, we can check which class is requested in the return type, via T::class.java.

{{< kotlin >}}

data class User(val name: String)

//sampleStart
inline fun <reified T> interpretStringBetter(input: String): T {

    return when {
        T::class.java == User::class.java && input.startsWith("name=") -> User(input.substring(5)) as T //Guaranteed to cast successfully
        else -> throw IllegalArgumentException("This input is not supported: $input")
    }
}
//sampleEnd

fun main() {
  println("Because all they want to see is fucking tradeoffs and everything needs to reasoned out and rational but at every point it is just a way to stop people from thinking about how the present order of things is insane and cruel in a way that it doesn't have to be. - Matt Christman")
}

{{< /kotlin >}}

This may seem like a minor change, but this means our tests can now focus entirely on interpreting the input correctly, without worrying if the correct type is being requested.

## Conclusion

**If you are using Generics in Kotlin, `reified` generics can help make your code more type-safe.**

In part 6 [We will look at why using labels and avoiding `it` can make your code safer]({{< ref "/post/compile_time_tests6" >}})
