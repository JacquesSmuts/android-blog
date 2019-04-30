---
title: "Don't Pass Around Your Room/Database Entity"
author: "Jacques Smuts"
cover: "/images/modularization_room1.png"
tags: ["Android", "Room", "Modularization", "Kotlin"]
date: 2019-04-25T10:35:54+02:00
publishdate: 2019-04-29T10:00:00+02:00
draft: false
---

If you're thinking of modularizing your Android App and you use Android Room, you should not pass around your Room Entity.

<!--more-->

## Room: Only In Your Database Module

Room is great, and so is modularization. If you're using Room, you probably have several [Room Entity Objects](https://developer.android.com/training/data-storage/room/defining-data) that look kinda like this:

{{% gist "JacquesSmuts" "0fc7ced6f167b435f41882c36e9a80cf" %}}

Chances are you're passing this model around everywhere. Every time you want to access your `User` class, or save it, or display the data to the UI, you use the same `User` class, which doubles as your Room `Entity`.

For a one-module proof of concept app, this is probably fine. But the moment your app expands and you begin to modularize, you'll run into issues. Specifically, Gradle will tell you that you have `unresolved Supertypes`.

```python
Supertypes of the following classes cannot be resolved. Please make sure you have the required dependencies in the classpath:
com.example.persistence.AppDatabase, unresolved supertypes: androidx.room.RoomDatabase
```

This happens because you're passing the `Entity` from your database module to other modules which don't implement the room compiler. So then you add the room compiler to your other modules. [Just like StackOverflow suggests](https://stackoverflow.com/questions/53152796/androidx-room-unresolved-supertypes-roomdatabase).

{{% gist "JacquesSmuts" "af7ffc45a38232998cf48ec0d3722855" %}}

**Don't do this.** Please. It breaks the core concept of [Information Hiding](https://en.wikipedia.org/wiki/Information_hiding) as would be required by a project that implements a proper Separation of Concerns.

### Map Your Entity

Instead, you should have two distinct objects. You should one object that represents the database Entity, and one object for passing around the pure data. Your database module should be literally the only module which imports the Room library. If any other module imports those implementation details of the database module, then you are not `Information Hiding` as you should. This is especially true if you may want to change out your Room database for something like [SqlDelight](https://github.com/square/sqldelight). 

Instead, map your data class to a new `Entity`, like so:

{{% gist "JacquesSmuts" "b2419911d125845cdc701166491421e5" %}}

Perhaps you have some other way of mapping to and from your `User` to your `UserEntity` class. You may think it'd be better if it happened automatically via a Serializer or the like. Normally I'd agree that automated serialization is easier in the long term, but this way, any changes to your `User` class results in a compiler error in your `UserEntity` class, which shows you:

1. That your database needs to go up a version; and
2. what migration steps to take, based on what changes the mapper requires.

I suggest you turn all your database classes into `internal` class, [as I've discussed before]({{< ref "/post/modularization" >}}). This is a bit of a tedious thing to do sometimes, but you will be super-happy in the long term. Future work and refactoring will be very easy.

There is one other thing though.

### LiveData + Room, But In Another Module

One of the nice things about LiveData and Room is how easily they interact.

But then you have this problem in your Repository

{{% figure src="/images/modularization_room2.png" alt="compiler error" title="If you see this warning, you're headed in the right direction." width="80%"  class="zoomable" %}}

Turns out you can't use LiveData to hook directly on to your database, because that exposes internal database entities outside the module. You want LiveData but you still want a proper separation of concerns. How? You use the LiveData Transformation function.

{{% gist "JacquesSmuts" "edde89d887a97e2dafe7da164f39ed5d" %}}

This means you have a direct LiveData pipeline from your ViewModel/View to your database, without actually exposing your internal database components. You get to have your module and access it too.

### Conclusion

I showed you why you shouldn't pass around your Room @Entity, and how to separate your modules without losing functionality.
