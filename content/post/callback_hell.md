---
title: Escaping Callback Hell with Generic SuspendCoroutine Infix Extension Functions
author: Jacques Smuts
image: /images/callback_hell4.png
slug: callback-hell
categories:
    - Kotlin 
tags: 
    - android
    - kotlin
    - suspendCoroutine
    - coroutines
    - callback hell
    - infix notation
date: 2019-05-20T23:12:36+02:00
draft: false
---

Callback hell happens all the time in Android. Luckily, with coroutines, there's an easy way out.

<!--more-->

So you're hopefully aware of [callback hell](http://callbackhell.com/), which makes your code difficult to read, and also makes the sequence of events difficult to understand. This is something we want to avoid, but it's not always easy. If you're using any sort of third-party library, (like Firebase) you're probably forced into this pattern regularly. A simple callback would look like this:

{{< gist "JacquesSmuts" "6cdad47d20e6c4f2a92ebee4f1ff8640" >}}

As you can see, we're one level into callbacks. At a glance, it would seem like the result would be handled immediately after sending out the `saveUsername` function. But it in fact creates an asynchronous task (most likely on the IO thread). Which means the order of events quickly becomes unclear, especially to a new developer climbing on the project. An experienced dev would know that the order of events isn't clear, but it'll take a few minutes to fully comprehend what the actual order is.

However, if your operation is suitable for coroutines (such as API calls, disk writes or background processes) we can "flatten" the callback hell fairly easily.

### SuspendCoroutine

I'm assuming you're aware of coroutines and suspendFunctions already. If not, it's time to [start using them](https://kotlinlang.org/docs/reference/coroutines-overview.html). First, I need to introduce suspendCoroutine. The [official documentation on suspendCoroutine](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/suspend-coroutine.html) doesn't make it particularly clear. But [this StackOverflow answer](https://stackoverflow.com/questions/50315802/correct-way-to-suspend-coroutine-until-taskt-is-complete) provides a good idea of how to use it. So if we were to write a suspendCoroutine version of our function, it would look like this:

{{< gist "JacquesSmuts" "40462adab3f657da91d25dfdba84ef99" >}}

As you can see, the saveUsername function has turned into a function which returns the result in the same line, but can only be called from a coroutineContext/suspendFunction. This is perfect for making your code look cleaner and run in sequence with more confidence.

However, it requires you to write a suspendCoroutine function for each function you want to flatten in this way. This adds a lot of boilerplate. So is there some way we can use generics to avoid this boilerplate? Of course!

### Generic SuspendCoroutine

If you read my [first article on kotlin generics]({{< ref "/post/generic_interface_and_methods" >}}) you'll know that the IntelliJ IDE is smarter than us, so we'll use it to generate our function. We'll call our function `suspendAsync()` and pass in our method, along with the input, and see what happens. Remember you can pass a method reference with `instance::method`

{{< figure src="/images/callback_hell1.png" alt="compiler error" title="If you see this warning, you're headed in the right direction. " width="80%"  class="zoomable" >}}

Then IntelliJ creates a function which looks like this:

{{< figure src="/images/callback_hell2.png" alt="compiler result" title="Not generic yet. " width="80%"  class="zoomable" >}}

And through a little bit of Generic reworking, as well as [my knowledge of KFunction analogues]({{< ref "/post/kfunction_analogues" >}}), I replaced the `<String>` input and output types with generic types, resulting in this function:

{{< gist "JacquesSmuts" "8ae63a3da81d38708c0c35a300280a86" >}}

As you can see, it's about the same as our previous function. The difference is, you can pass in any function which takes a single input and a function callback, and it becomes a one-liner instead of a callback.

If you want to pass in two inputs, it's slightly trickier, but still very doable:

{{< gist "JacquesSmuts" "9d1c10544e72c4d88e978ca65c5f07ff" >}}

And there you have it. You can reduce any qualifying callback with these functions. Other variations can be covered with a few more generic functions. However, it doesn't feel very nice calling a function in this way. First you write `suspendAsync`, then you pass in the function you actually want to call? Not the best, because we're used to referencing the function we want to call first.

So maybe we can do better?

### Generic Infix SuspendCoroutine Extension Function

I like the [infix notation in Kotlin](https://kotlinlang.org/docs/reference/functions.html#infix-notation), though I usually avoid it because it's a little dangerous and it's not easy for new developers to discover it's usage in any given codebase. In this case the increased readability and ease of use may be worth it.

So using the same method of letting the IDE generate our function for us, we write out the infix function we want. I call it `suspendAndInvokeWith`, because we are invoking a function, turning it into a suspend function, and passing in arguments.

{{< figure src="/images/callback_hell3.png" alt="compiler result" title="If it wasn't for compiler suggestions I'd be useless. " width="80%"  class="zoomable" >}}

Which we clean up and genericify a little bit into our result:

{{< gist "JacquesSmuts" "d099eff5e66aa6b1d5de924f87c9e967" >}}

This feels a lot more natural. You start out by writing the function you would normally write, using the infix notation and passing in your input.

Infix functions require only a single input, so if you want to pass in more than one argument into a function, you need to do it by using Pairs or Triples, like so:

{{< gist "JacquesSmuts" "b6b45f506d001005bc8914e0925986cc" >}}

And there you have it. This won't work on everything, but it's a start to reducing boilerplate.

### Source

At [Flat Circle](https://flatcircle.io/), we're trying to make our code as readable as possible, and found that using coroutines and avoiding callback hell is a good way to go about doing this. So we built these infix notations (and a few other utility functions) into a little [CoroutineHelper Library](https://github.com/flatcircle/coroutinehelper) to make our lives easier. Please go ahead and use that library or copy the code to help make your code as readable as possible.

### Conclusion

Callback hell isn't the end of the world, but sometimes it can add just a little bit too much confusion to an already complex lifecycle. Hopefully with Coroutines, coupled with the techniques and Generic Utility functions above, you can reduce callback hell a little bit in your code.
