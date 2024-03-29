---
title: "Use More Named Arguments in Kotlin"
author: "Jacques Smuts"
image: "/images/compile_test_named_arg.png"
categories:
  - Android
  - Compile Time Safety
tags:
  - named parameters
  - architecture
date: 2022-11-07T11:33:09+02:00
draft: false
---
Using named arguments in Kotlin causes breaking changes to be caught faster. This article shows you how.
<!--more-->

This article is a follow-up on [Part 1 in this series,]({{< ref "/post/compile_time_tests" >}}) wherein I explain why the test pyramid is incomplete, and that you should architect so that breaking changes gets caught at compile-time if possible. 

Further context: I wrote this in 2020, but crashed out from pandemic-induced stress before I could finish the series. I'm now going back to finish it.

## What is a named argument?

When you pass an argument to a function or parameter, you can name the argument instead of just passing in the arguments in the right order. This is most powerful when you have optional arguments, but for now, let's assume you have a simple `User` class.

{{< kotlin >}}

fun main() {
//sampleStart
  data class User(
    val name: String,
    val surname: String,
    val age: Int = 0
  )

  // This is how you would normally pass arguments. Name, Surname, Age.
  val userNormal = User(
      "Kim",
      "Katsuragi",
      43
  )

  // This is how you use named arguments. You put in extra effort to specifically name the arguments you are passing in.
  val userNamed = User(
      name = "Klaasje",
      surname = "Amandou",
      age = 28
  )
//sampleEnd

  println("$userNormal is literally the best.")
}

{{< /kotlin >}}

## Why is a named argument?

Named arguments are usually used to differentiate between several optional parameters and pass in only the necessary arguments. That is a great use of them, however I contend that **you should almost always use named arguments.** Especially if you have more than two arguments.

The reason for this can be demonstrated simply. What if a developer was tasked to add a `height` field? It'd be very easy to make this mistake:

{{< kotlin >}}

fun main() {
//sampleStart

  // Added `height`
  data class User(
    val name: String,
    val surname: String,
    val height: Int, // height added here.
    val age: Int? = null
  )

  val userNormal = User(
      "Kim",
      "Katsuragi",
      43
  )

  val userNamed = User(
      name = "Klaasje",
      surname = "Amandou",
      age = 28
  )
//sampleEnd

  println("$userNamed is an extremely well written character. I refused to arrest her, despite knowing that she's manipulative.")
}

{{< /kotlin >}}

If you run the above code, you'll get a compiler error. Note that the error appears (correctly) for the `userNamed` variable, but not for `userNormal`. Now `Kim Katsuragi` has a height of 43 and an age of null.

 This might seem like a contrived example, because "everybody knows that you should put new variables at the end". I disagree with that notion, because it places the burden of knowledge on future unknown developers or circumstances. It's often that developers feel the need to change the order of arguments or variables. They usually don't, for fear of breaking things. However if you always use named arguments, the order of constructor inputs would not be a factor, and refactoring your constructor would carry less risk.

 I've run into various scenarios where I've had to change the arguments on a function/constructer where I had to manually go through every single instance where the function was called to make sure it was still being called correctly despite the change. Furthermore, I don't know what unit-test can be written to ensure that all instances of a constructor is being called correctly. However when I switched to named arguments, I knew that the calling code would always keep working if no change was required, and throw a compiler error if a change was required.

**Named arguments catches breaking changes from incorrect arguments immediately**, in a way which would be slower with unit-tests.

However this only works if you use named arguments everywhere. It's not forced.


## Named Arguments Sounds Great. Can I Force Their Use (a little bit)?

Some people would propose [this hacky workaround](https://stackoverflow.com/questions/37394266/how-can-i-force-calls-to-some-constructors-functions-to-use-named-arguments) that is as clever as it is unpleasant to implement. Don't do this.

Or you can add a lint check to your project which forces you to use named arguments.

I tried to write [this little library](https://github.com/JacquesSmuts/NamedArgsLint/blob/master/README.md) which does it for you. It forces all functions or constructors with more than two arguments to require named arguments, otherwise it tags it as an error.  Unfortunately, this library doesn't work reliably. I kept struggling with lint, and eventually gave up (see context at the start of this article).

However, you can see there is some demand for it in Kotlin itself, [here](https://youtrack.jetbrains.com/issue/KT-14934), as well as in [Detekt](https://github.com/detekt/detekt/issues/3534). You can use [this library](https://github.com/chao2zhang/RequireNamedArgument) to force any one function to have NamedArguments, but you can't enforce it project-wide with specific rulesets easily.

Regardless of lint libraries, you should use [this IntelliJ plugin](https://plugins.jetbrains.com/plugin/10942-kotlin-fill-class) to easily fill arguments or add names to arguments. I use it all of the time.

## Conclusion

Use named arguments. Not everywhere all the time, but use them regularly.
