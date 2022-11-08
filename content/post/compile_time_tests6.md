---
title: "Avoid `it` in Kotlin, Name Things"
author: "Jacques Smuts"
image: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "Generic", "labelled scope"]
date: 2023-04-02T22:07:55+02:00
draft: true
---

Using labelled scopes in Kotlin can help make your code safer.

<!--more-->

This article is Part 6 in the [series on making your code compile-time safer]({{< ref "/post/compile_time_tests" >}}).

## Labels & Scope

Kotlin has great type inference, which makes it easy to use the `it` and `this` keywords. However, strong inference does introduce risks.

In many circumstances, it's easy and sensible to replace `if (movieName != null) { println(movieName!!) }` with `movieName?.let { println(it) }`. Kotlin's scope functions are powerful and useful, but also risky. As explained in [the official documentation](https://kotlinlang.org/docs/reference/scope-functions.html), you want to avoid overuse. Specifically, risks are introduced because:
 - Nested scope functions can be difficult to parse
 - Chained scope functions can be difficult to parse
 - It can be difficult to immediately see what `this` or `it` refers to, especially if you combine several of them
 - Using `x?.let{}` is tempting to use to swallow errors instead of throwing exceptions

This isn't an inherent problem with scope functions. These problems also apply to nested lambdas, nested loops, and chained functions. It is possible to avoid some of these issues by using named variables and labels.

## Named Scope Variables

An example you might see is something like this. A scoping function, followed by a function that takes a lambda input. In both cases the type is inferred and the word `it` is used to refer to the innermost parameter:

{{< kotlin >}}

//sampleStart

fun printConvertedString(inputString: String?) {

    inputString?.let {
        processResultFromInput(it) {
            println(it)
        }
    } ?: println("InputString is null. This is fine. 🔥🐶🔥")

}
//sampleEnd


fun processResultFromInput(input: String, outputListener: (String) -> Unit) {
    outputListener(input + " " + input)
}

fun main() {
  printConvertedString("Get your ass to mars.")
}

{{< /kotlin >}}

Luckily, IntelliJ tells you that this is unsafe and suggests that you add a name to at least one of the parameters. If there is any chance of misunderstanding what `it` or `this` refers to, you should always add a name. The rule I like to follow is this:

**For a one-liner with zero nesting, you can use `it`/`this`. In all other cases, assign an explicit name.**

You'll end up with something like this:

{{< kotlin >}}

//sampleStart

fun printConvertedString(inputString: String?) {

    inputString?.let { input ->
        processResultFromInput(input) { output ->
            println(output)
        }
    } ?: println("InputString is null. This is fine. 🔥🐶🔥")

}
//sampleEnd


fun processResultFromInput(input: String, outputListener: (String) -> Unit) {
    outputListener(input + " " + input)
}

fun main() {
  printConvertedString("Open your mind.")
}

{{< /kotlin >}}

This seems acceptable, except that you can't tell at a glance what the `output` type is. The `input` type is pretty obvious, as it is inferred from the variable on the immediate left and above. If you happen to be looking at the code in an IDE you can hover to see the inferred type. If you are not in an IDE (because you're doing a code review on a pull request on GitHub/GitLab, for example), you'll have to go look at the functions involved to see what the types are. The `println` function takes any type as input so we don't know if it's a string being outputted there.

In cases like these, it's often a good idea to just declare a type explicitly. Yes, yes, I know you like type inference and not having to type of out everything like in Java, but code readability as well as our ongoing focus on Architecting to Fail Faster indicates that it's a good idea to specify some types explicitly, like so:

{{< kotlin >}}

//sampleStart

fun printConvertedString(inputString: String?) {

    inputString?.let { input ->
        processResultFromInput(input) { output: String ->
            println(output)
        }
    } ?: println("InputString is null. This is fine. 🔥🐶🔥")

}
//sampleEnd


fun processResultFromInput(input: String, outputListener: (Int) -> Unit) {
    outputListener(input + " " + input)
}

fun main() {
  printConvertedString("What if this is a dream.")
}

{{< /kotlin >}}

If you run the above code you'll see that we get a compilation error. It turns out the function was returning an Int, not a String as we expected. This is a rare event, but still important to consider. If the type of object being returned is likely to change or be misinterpreted, it's a good idea to specify the type explicitly.

## Library?

*Work in progress. Check this space.* It should be fairly easy to check if a function takes up more than one line, in which case require a named variable.

## Conclusion

Reduce usage of the `it` keyword. It makes your code more readable and easier to refactor.
