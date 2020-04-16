---
title: "Architecting to Fail Faster in Kotlin: Using The when Expression"
author: "Jacques Smuts"
cover: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "When Expression"]
date: 2020-03-09T18:50:56+02:00
draft: true
---

Using the `when` expression, you can structure your code to catch new unhandled scenarios.

<!--more-->

This article is a follow-up to [Part 3 in this series,]({{< ref "/post/compile_time_tests3" >}}) wherein I explain why Custom Lint Warnings are a powerful method to prevent known errors from resurfacing.

## What is the `when` Expression?

As explained in the [Kotlin Reference Docs](https://kotlinlang.org/docs/reference/control-flow.html), the `when` keyword is similar to Java's switch, but better. Usually, a `when` statement looks something like this:

{{< kotlin >}}
fun getNameFromId(input: Int): String = "Harrier Du Bois"

fun main() {
  val input: Any = 2
//sampleStart
  // A typical when example. Cover two cases, ignore others.
  when (input) {
      is String -> println("Hello $input")
      is Int -> println("Hello ${getNameFromId(input)}")
  }
//sampleEnd
}

{{< /kotlin >}}

This is fine. Except that any alternate `greeting`s are ignored. Furthermore, if any new `greeting`s are introduced, there's nothing telling you that this `greeting` is now unhandled. To add to that, it's difficult to structure your code in such a way that your tests automatically cover new `greeting`s when you add them.

## Forced to Cover All Cases

We are trying to architect our code to fail faster, so I wouldn't recommend using the `when` statement like this. Instead, we want to write our code in such a way that it throws an error if a case isn't handled. The first step would be to make sure that `when` expressions always return a value or reference. Technically you could say we want to avoid `when` *statements* and use only `when` *expressions*. Like this:

{{< kotlin >}}

fun getNameFromId(input: Int): String = "Kim Kardashian"

fun main() {
  val input: Any = 2
//sampleStart
  // This will not compile :)
  println( when (input) {
      is String -> "Hello $input"
      is Int -> "Hello ${getNameFromId(input)}"
  })
//sampleEnd
}
{{< /kotlin >}}

If you try to run the above code, you'll notice that it does not compile. The `println` function requires an input parameter, so the compiler requires the `when` expression to be exhaustive.

The simple solution here is to put an `else` branch into the expression, which throws an Exception if an unexpected input is encountered. It's good practice to throw exceptions (or at least log them) on unexpected use-cases. Swallowing exceptions means that your users will experience bad behaviour and you won't even know.

{{< kotlin >}}

fun getNameFromId(input: Int): String = "Har har"

fun main() {
//sampleStart
  val input: Any = 2.00
  // All cases covered, at least according to the compiler
  println( when (input) {
      is String -> "Hello $input"
      is Int -> "Hello ${getNameFromId(input)}"
      else -> throw IllegalArgumentException("The ${input::class.java} type isn't supported.")
  })
//sampleEnd
}

{{< /kotlin >}}

This means that when a new `greeting` is encountered you will definitely know about it.

This is our main goal: to **make sure that when expressions are forced to be exhaustive**. You can do this by:
1. passing in `when` as an argument. - `anyFunction(when(x){...})`
2. making sure `when` returns a value. - `val y = when(x){...}`
3. Creating an extension function to force exhaustiveness - `when(x){...}.exhaustive`

There are some [extension functions proposed in here](https://youtrack.jetbrains.com/issue/KT-12380) and [here](https://blog.karumi.com/kotlin-android-development-6-months-into-it/).

Back to our example which throws an IllegalArgumentException: This solution is still suboptimal, because if you have a new input or use-case it will be experienced at *runtime*, instead of at *compile time*.

So can we make it fail even faster?

## Using Enums / Sealed Classes to Fail Faster

I consider a `when` expression to be ideal if it is exhaustive, yet does not require an `else` branch. The only times that happens is if the `when` expression is used on an Enum/Sealed Class.

So we can therefore change our above example into this:

{{< kotlin >}}

fun getNameFromId(input: Int): String = "The Kingdom of Conscience is evil"

//sampleStart

sealed class Input {
    class StringInput(val value: String): Input()
    class IntInput(val value: Int): Input()
}

fun main() {
  val input: Input = Input.IntInput(451)

  println( when (input) {
      is Input.StringInput -> "Hello $input.value"
      is Input.IntInput -> "Hello ${getNameFromId(input.value)}"
  })
}
//sampleEnd

{{< /kotlin >}}

This adds a little bit of extra code, but it makes things safer. It means if you ever try to pass in a Double as an input, the compiler will stop you. Furthermore if you add a new case, you only have to add it in the `Input` sealed class, and you will automatically get an error on all the `when` expressions which evaluates that Input type.

{{< kotlin >}}

fun getNameFromId(input: Int): String = "The Kingdom of Conscience is evil"

//sampleStart

sealed class Input {
    class StringInput(val value: String): Input()
    class IntInput(val value: Int): Input()
    class DoubleInput(val value: Double): Input()
}

fun main() {
  val input: Input = Input.IntInput(451)

  // Compiler will throw a compile-time error here, because input isn't exhaustive.
  println( when (input) {
      is Input.StringInput -> "Hello $input.value"
      is Input.IntInput -> "Hello ${getNameFromId(input.value)}"
  })
}
//sampleEnd

{{< /kotlin >}}

And that's an ideal usage of the `when` keyword. Any changes to the input type will result in compile-time errors.

## "That's nice, but how does this help me in practice?"

**Using Sealed Classes with exhaustive `when` expressions makes your code safer.**

If you ever see yourself using a when statement, ask yourself this:
*Can I turn this input into a sensible enum or a sealed class?*

If the answer is yes, you should do it.

## A Library

Yes, I also made [a library](https://github.com/JacquesSmuts/ExhaustiveWhen) for this.

## Conclusion

Use `when` expressions with sealed classes.

In part 5 [We will have a brief look at using reified types in Kotlin]({{< ref "/post/compile_time_tests5" >}})
