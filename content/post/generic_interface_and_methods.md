---
title: "Generics + Reflection + Type Inference + Reified Type = Kotlin Magic"
author: "Jacques Smuts"
cover: "/images/generic_interfaces3.png"
tags: ["android", "kotlin", "KFunction", "KFunction1", "KSuspendFunction", "KSuspendFunction1"]
date: 2019-05-01T08:30:00+02:00
<!-- publishdate: 2019-05-01T10:00:00+02:00 -->
draft: false
---

With Kotlin, it's easier than ever to code with reflection and generics. This post attempts to give one example.

<!--more-->

## Background: An unlikely scenario

I couldn't think of a simpler scenario to demonstrate KFunction and KSuspendFunction working well in tandem with Generics, so please bear with me when I present this unlikely scenario. 

Let's assume you have several different endpoints and you need to differentiate between them easily, even though they can change dynamically. If you're an Android developer, you're probably used to Retrofit and its associated Interfaces. You have several classes that look like this:

{{% gist "JacquesSmuts" "8374926ea79785b96aec53714d1dd44e" %}}

Each of these interfaces are used by RetroFit to generate the right api call you can call. And then you have a few classes which may or may not implement these interfaces. So if you have a list of unknown services and you want to get the AWS Repos, you have to iterate through your services, find the AwsService and then make the call. You can do something like this:

{{% gist "JacquesSmuts" "142f78c848c784119ca76bdc151ea01f" %}}

But this means that every single time you want to check your GitHubRepos, your AwsRepos, or any other api call, you have to manually write out this entire process from scratch. That's highly inefficient and boilerplatey. Can't you push them all through some central function?

**Yes, with Reflection + Generics you can**

## Kotlin Reflection

So first, you should read the [official kotlin documentation on reflection](https://kotlinlang.org/docs/tutorials/kotlin-for-py/member-references-and-reflection.html). It's a really good starting point, even if you already know Reflection from other languages. Even better, read [this great guide on Medium](https://medium.com/kotlin-thursdays/introduction-to-kotlin-generics-reified-generic-parameters-7643f53ba513) about both reflection and generics. But how does this relate to us?

We want to pass the AWS Repo function in the AWS interface and get the AWSRepo result. Later we'll make it generic, but first step is just reflection. So let's call that non-existent function. We'll call it `doAwsApiCall` and it has two input parameters: 

- The list of `services`; and 
- the `listAwsRepos` method from the `AwsCodeCommitService` interface.

```java
doAwsApiCall(services, AwsCodeCommitService::listAwsRepos)
```

But that function doesn't exist yet, so just ask IntelliJ/Android Studio to create it for you. (We're doing this because the IDE is smarter than us at figuring out the input parameters.)

{{% figure src="/images/generic_interfaces1.png" alt="compiler error" title="If you see this warning, you're headed in the right direction. " width="50%"  class="zoomable" %}}

And you'll get something like this:

{{% gist "JacquesSmuts" "10a7bab01fe4e8292b307955beac2da6" %}}

So the `KFunction1` here is reference to a specific function that can be called. To make a class run that function, you just have to pass the calling class into the function. Yes, that's hard to understand and a little bit backwards, but maybe it makes more sense in code.

{{% gist "JacquesSmuts" "53d6fb84c6ac3eebb6dde3d830bca61b" %}}

This is almost the same as our first implementation, except for the commented part. What this means is that we can pass in any function from the `AwsCodeCommitService` interface, and it will automatically be called inside the `doAwsApiCall` function.

But this means we have to write this code for each of our interfaces. One for `GitHubService`, one for `BitBucketService`, and so forth. That's a lot better, but still not good enough. If we wanta single function to handle all of these api calls, then we must answer this question:

- Can we write a function that takes **ANY** interface and call **ANY** function from that interface, and return **ANY** result, **and still be type safe**?

Yes, with Reflection and Generics you can!

## Kotlin Generics Plus Reflection

So, using the great explanation into inline functions and reified Generics in [this great guide on Medium](https://medium.com/kotlin-thursdays/introduction-to-kotlin-generics-reified-generic-parameters-7643f53ba513) as a basis, I'm going to just turn everything into a Generic and see what happens.

{{% gist "JacquesSmuts" "087ee57e4afca3fde8112c3b37dcbf4e" %}}

All I did was 

1. Added `<reified Service, Result>` to the beginning of the function, to pass in those types
2. Replace any reference to `AwsCodeCommitService` with the generic `Service`
3. Replaced any reference to `List<AwsRepo>` with the generic `Result`

And it works and accepts literally any `interface::method` pair which returns the expected Result type. I wanted to restrict it a bit, so I made sure that `Service` extends the `RetroService` interface I defined all the way at the top. Now it only accepts the right methods.

However...

{{% figure src="/images/generic_interfaces2.png" alt="compiler error" title="If you see this warning, you're headed in the right direction. " width="90%"  class="zoomable" %}}

The `GitHubRepos` api call takes a username as an input parameter. So now the IDE is telling you that `listGitHubRepos(username)` is a KFunction2. What's that? 

- KFunction1 is a function reference with zero input arguments, e.g. `listAwsRepos()`
- KFunction2 is a function reference with one input argument, e.g. `listGitHubRepos(username: String)`

So let's create another `doApiCall()`, except that it takes kFunction2 as an input.

{{% gist "JacquesSmuts" "499cab2b907298bc03de8d02d19c22c0" %}}

And this function works. All we had to do was add the `Input` Type as a Generic type, and pass the `input` into the `KFunction2`. This means in other words that we can pass in a list of unknown classes, a function and an input. If any class in that list of unknown classes is the correct type, the right function will be called on that class. 

The best part is that because of Kotlin's intensely awesome type inference, I never even had to pass in the `<Service, Input, Result>` types. It was inferred automatically.


## Suspend Functions

You may have noticed that little suspend function back in the beginning. I didn't forget about it. It's handled identically, but you have to create a new function for it unfortunately. Something like this:

{{% gist "JacquesSmuts" "96af861d1096ed8feb8c299d19a623b0" %}}

As you can see, all I did was replace KFunction1 and KFunction2 with KSuspendFunction1 and KSuspendFunction2, respectively. The reason for this is because the signature for a suspend function and normal function in Kotlin are not the same. Hopefully if you're using suspend functions you already know this though.

### Conclusion

I don't even know why anyone would land in this bizarre scenario of needing to iterate through dozens of dynamically changing classes with unpredictable interfaces. But if you do, it's very solveable with the techniques above.

- Reflection is great
- Generics are amazing
- Kotlin Generics + Reflection + Type Inference is mindblowing

If you can get your mind around higher-order functions, generics, and reflection, you will become way more efficient as a developer. I hope this shows an example why.
