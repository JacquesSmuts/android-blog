---
title: "Architecting to Fail Faster in Kotlin: Labels,Scope and Litera"
author: "Jacques Smuts"
cover: "/images/compile_test1.png"
tags: ["Kotlin", "Android", "Generic", "labelled scope"]
date: 2020-04-02T22:07:55+02:00
draft: true
---

Using labelled scopes in Kotlin can help make your code safer.

<!--more-->

This article is Part 6 in the [series on making your code compile-time safer]({{< ref "/post/compile_time_tests" >}}).

Kotlin has great type inference and makes it easy to use the `it` and `this` keywords. However, strong inference does introduce risks. 

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
