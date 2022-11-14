---
title: "Coroutines Are More Maintainable"
author: "Jacques Smuts"
image: "/images/compile_test1.png"
tags: ["Kotlin", "Android"]
date: 2023-06-17T10:20:03+02:00
draft: true
---

Kotlin Coroutines are the best asynchronous solution. This post explains why.

This article is Part 8 in the [series on making your code compile-time safer]({{< ref "/post/compile_time_tests" >}}).

<!--more-->

## Other Asynchronous solutions

There are many ways to handle asynchronous events and/or threading on Android/Java.

- You can use **runOnUiThread(Runnable)** just to switch to the main thread.
- You can be a leet haxor or [shoot yourself in the foot](https://twitter.com/ErikHellman/status/1262322194182438912?s=20) with **java.lang.Thread/android.os.Handler.**
- You can use the now deprecated [android.os.AsyncTask](https://developer.android.com/reference/android/os/AsyncTask).
- You can use LiveData's [PostValue function](https://developer.android.com/reference/androidx/lifecycle/LiveData#postValue(T)) to keep things on the main thread.
- You can go full functional-reactive programming and use [RxJava](https://github.com/ReactiveX/RxJava).
- You can choose the correct answer and use [Kotlin Coroutines](https://kotlinlang.org/docs/reference/coroutines-overview.html).

Each of these have their own place and there are various discussions to be had on these. However this is a series on making your code [safe at **compile time**]({{< ref "/post/compile_time_tests" >}}), so our focus will be entirely on that single aspect.

## Kotlin Coroutines are uniquely safe

How so? Well, with many of the above examples, there is nothing guaranteeing that a function can only be called on a certain thread. Deciding on which thread/dispatcher to use is usually passed off to the calling class. For example, by attempting to make a network call, it's easy for someone to run into the classic [NetworkOnMainThreadException](https://stackoverflow.com/questions/6343166/how-to-fix-android-os-networkonmainthreadexception). That is an important and well-named exception, but it occurs only at runtime. Similarly, if you use `liveData.setValue` instead of `liveData.postValue`, there is no compiler error telling you that this will cause a [CalledFromWrongThreadException](https://stackoverflow.com/questions/5161951/android-only-the-original-thread-that-created-a-view-hierarchy-can-touch-its-vi). The same applies to RxJava (which I love otherwise). If you observe an RxJava observable, you don't get compile-time errors about which thread you're on, or any indication that thread-safety should be considered before proceeding. You get runtime errors, same as any other threading solution.

Furthermore, LiveData and RxJava Observers need to be cleaned up. LiveData only allows you to observe from a lifecycle, so that means LiveData is restrictive but inherently safe in that regard. RxJava easily allows you to observe forever, which can lead to memory leaks. 

On the other hand, Coroutines introduces this wonderful magical solution:

{{< figure src="/images/compile_test6.png" alt="Za Warudo" title="『S U S P E N D』" width="50%" >}}

Which results in this beautiful error.

{{< figure src="/images/compile_test7.png" alt="Za Warudo" title="Coroutine Error" width="60%" >}}

Which means that the moment you start to use coroutines, certain parts of your code are locked off unless you launch a coroutine and explicitly define a scope (and dispatcher) for that coroutine. You can still shoot yourself in the foot, but this restricts your access and makes it more difficult to access specific functions unless you are calling it in the right context. That's pretty great, but there's a second part here which brings everything together perfectly.

## Telling you what thread to use

When you call a function, that function requires the right inputs to work and you get a compiler error if you don't pass the right inputs. The function handles everything else. It has its own responsibilities and when you call that function you shouldn't have to know what goes on inside the function to get the right result.

This logic does not hold true for threading. A function runs on whatever thread it's called on, and the best you can do is to tag a function with an annotation to let the next developer know to call it on the right thread. In Android that would involve putting `@WorkerThread` or `@MainThread` above the function declaration. But this does not throw a compiler error and it doesn't actually do any thread switching for you, [as some would expect.](https://stackoverflow.com/questions/33649994/how-use-workerthread-annotation-in-android-project)

However, with Kotlin Coroutines, you can use the `withContext(ioDispatcher)` function to switch threads easily.

TODO: add kotlin code to show an api call function, first normally, then with suspend, then with withContext().

## Why is this better?

1. The function can only be called with a strictly defined coroutine scope, so the
1. The function can be called from any thread, and it will be returned on that thread.
