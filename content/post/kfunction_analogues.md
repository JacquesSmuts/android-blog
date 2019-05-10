---
title: "The Problem with Kotlin Kfunction Receiver Functions"
author: "Jacques Smuts"
cover: "/images/generic_interfaces3.png"
tags: ["android", "kotlin", "KFunction", "KFunction1", "KSuspendFunction", "KSuspendFunction1", "Receiver Function"]
date: 2019-05-08T17:54:58+02:00
draft: false
---

KFunction can be written in more than one way. Here's a list of analogues, plus some criticism of the Reciever Function.

<!--more-->

After my [previous article on Reflection]({{% ref "/post/generic_interface_and_methods" %}}) was posted, I got a great comment from @pacoworks:

{{< tweet 1126057942724296704 >}}

I assumed that since the IDE told me that there was a type mismatch, that suspend functions weren't supported.

Turns out I was wrong; KSuspendFunction can be shortened. (click that thread to read more)  They're called Receiver Functions and they have since been added to the [Kotlin official documentation for higher-order-functions](https://kotlinlang.org/docs/reference/lambdas.html#function-types) with a nice explanation.

So, in order to help me remember the different ways to reference higher-order functions, I made this grid. It omits the list of passed interfaces for brevity.

{{% gist "JacquesSmuts" "f43755a820658b19f6c5fdd4fdb14eb4" %}}

| KFunction  | Analogue | ReceiverFunction |
| ------------- | ------------- | ------------- |
|  KFunction1\<Interface, Result\> |  (Interface) -> Result |  Interface.() -> Result |
|  KFunction2\<Interface, Input, Result\> | (Interface, Input) -> Result |  Interface.(Input) -> Result |
|  KFunction3\<Interface, In1, In2, Result\> |  (Interface, In1, In2) -> Result |  Interface.(In1, In2) -> Result |
|  KSuspendFunction1\<Interface, Result\> |  suspend (Interface) -> Result |  suspend Interface.() -> Result |
|  KSuspendFunction2\<Interface, Input, Result\> |  suspend (Interface, Input) -> Result |  suspend Interface.(Input) -> Result |

In all 3 cases you have to add `implementation "org.jetbrains.kotlin:kotlin-reflect:$kotlinVersion"` in your gradle.

However you only have to add the `import kotlin.reflect.KFunction1` import at the top of your file if you're using the explicit KFunction syntax.

I like the way ReceiverFunction is composed and I would like to use that, however I see a problem with it. If any new developer unused to this syntax works on this code, what should they google to find out more? `().->T` ? It's difficult to work with Reflection and Generics the first time, but if you see "KSuspendFunction1" you will be able to google it and find references to Reflection to understand what the code is doing.

The other thing to note is that if you use the ReceiverFunction notation and pass the wrong type of function, the IDE will always default to referring to the passed function argument as "KFunction".

{{% figure src="/images/kfunction_analogues2.png" alt="compiler error" title="I didn't call it a KSuspendFunction, but the IDE does. " width="90%"  class="zoomable" %}}

Which means the developer is likely to think that they are passing something entirely different than the required input, unless they know both the KFunction notation and the Receiver Function notation.

Since I like the Receiver Function syntax, I'm going to add a link to the Kotlin documentation in my code comments wherever I use the ReceiverFunction syntax. [This explanation on StackOverflow](https://stackoverflow.com/questions/45875491/what-is-a-receiver-in-kotlin) as well, since it explains the usage of receiver functions a bit better.

### Conclusion

Receiver Functions are nice, but there is no simple way for a developer to get to the right documentation from the code or IDE unless they already know what Receiver Functions are, or that they refer to KFunction.
In order to make it easier for people to understand your code, you need to choose one:

- Refer to KFunction explicitly; or
- Add comments every time you use Higher Order Receiver Functions

