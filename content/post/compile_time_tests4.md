---
title: "Always use Exhaustive `when` Expressions"
author: "Jacques Smuts"
image: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "When Expression"]
date: 2022-03-09T18:50:56+02:00
draft: true
---

Using the `when` expression, you can structure your code to catch new unhandled scenarios.
https://gist.github.com/AniketSK/5d7bc1b0cb379d2ed7812cb4a8603b75

<!--more-->

This article is a follow-up to [Part 2 in this series,]({{< ref "/post/compile_time_tests2" >}}) wherein I explain why named arguments are a good way to pick up potentially breaking changes at compile time.

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

This is fine. Except that any alternate `input`s are ignored. Furthermore, if any new `input`s are introduced, there's nothing telling you that this `input` is now unhandled. To add to that, it's difficult to structure your code in such a way that your tests automatically cover new `greeting`s when you add them.

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
  println(
    when (input) {
      is String -> "Hello $input"
      is Int -> "Hello ${getNameFromId(input)}"
      else -> throw IllegalArgumentException("The ${input::class.java} type isn't supported.")
    }
  )
//sampleEnd
}

{{< /kotlin >}}

This means that when a new `input` is encountered you will definitely know about it.

This is our main goal: to **make sure that when expressions are forced to be exhaustive**. You can do this by:
1. passing in `when` as an argument. - `anyFunction(when(x){...})`
2. making sure `when` returns a value. - `val y = when(x){...}`
3. Creating an extension function to force exhaustiveness - `when(x){...}.exhaustive`

There are some [extension functions proposed in here](https://youtrack.jetbrains.com/issue/KT-12380) and [here](https://blog.karumi.com/kotlin-android-development-6-months-into-it/), though I personally prefer [this one](https://gist.github.com/AniketSK/5d7bc1b0cb379d2ed7812cb4a8603b75).

Back to our example which throws an IllegalArgumentException: This solution is still suboptimal, because if you have a new input or use-case it will be experienced at *runtime*, instead of at *compile time*.

So can we make it fail even faster?

## Using Enum/Sealed Classes For The Perfect When Expression

I'd like to propose the idea of *The Perfect When Expression*. A when expression is *perfect* if it is exhaustive and does not have an `else` branch. As far as I know, that requires the use of an Enum/Sealed Class.

A Perfect When Expression will always fail if you add subclasses to the enum or sealed class. Using a perfect when expression is ideal for stability, because
1. you can limit a function's inputs or outputs to a specific subset of possibilities
2. future changes to that subset will result in a compilation error at each instance where a perfect when expression is used

So to bring this back to our `greeting` function, a perfect when expression would look like this:

{{< kotlin >}}

fun getNameFromId(input: Int): String = "The Kingdom of Conscience is evil"

//sampleStart

sealed class GreetingInput
data class StringInput(val value: String): GreetingInput()
data class IntInput(val value: Int): GreetingInput()

fun greeting(input: GreetingInput) {
  println(
    when (input) {
      is StringInput -> "Hello $input.value"
      is IntInput -> "Hello ${getNameFromId(input.value)}"
    }
  )
}

//sampleEnd

fun main() {
  val input: Input = Input.IntInput(451)
  greeting(input)
}
{{< /kotlin >}}

This adds extra code and makes things more restrictive, however it limits the possibility for wrongful use. It means if you ever try to pass in a Double as an input, the compiler will stop you. Furthermore if you add a new Input Type, you only have to add it in the `Input` sealed class, and you will automatically get an error on all the `when` expressions which evaluates that Input type.

For example, if we add a Double input type, we'd immediately get a compiler error before any tests have to run or be written, like so:

{{< kotlin >}}

fun getNameFromId(input: Int): String = "The Kingdom of Conscience is evil"

//sampleStart

sealed class GreetingInput
data class StringInput(val value: String): GreetingInput()
data class IntInput(val value: Int): GreetingInput()
data class DoubleInput(val value: Double): GreetingInput() // We added a new input type

fun greeting(input: GreetingInput) {

  println(
    when (input) { // Compiler will throw a compile-time error here, because input isn't exhaustive.
      is StringInput -> "Hello $input.value"
      is IntInput -> "Hello ${getNameFromId(input.value)}"
    }
  )
}

//sampleEnd

fun main() {
  val input: Input = Input.IntInput(451)
  greeting(input)
}
{{< /kotlin >}}

And that's an ideal usage of the `when` keyword. Any changes to the input type will result in compile-time errors.

## "That's nice, but how does this help me in practice?"

**Using Sealed Classes with exhaustive `when` expressions makes your code safer,** by detecting breaking changes faster.

If you ever see yourself using a when statement, ask yourself this:
*Can I turn this input into a sensible enum/sealed class?*

If the answer is yes, you should do it. I use the word "sensible" here, because I don't know how to articulate under what circumstances it's a good idea to turn an input into an enum or sealed class, so this is left as an exercise to the reader.

## A Library

Yes, I also made [a library](https://github.com/JacquesSmuts/ExhaustiveWhen) for this. It detects whether or not your `when` statements are exhaustive and provides a warning if they are not. If you don't want to add the word "exhaustive" to your `when` statements, or you don't want to wait for JetBrains to add exhaustive when statement config settings, I suggest you use this library.

## Conclusion

Use `when` expressions with sealed classes and make them exhaustive.

In part 5 We will have a brief look at using reified types in Kotlin
