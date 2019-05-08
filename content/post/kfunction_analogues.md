---
title: "Kotlin Kfunction Analogues"
author: "Jacques Smuts"
cover: "/images/generic_interfaces3.png"
tags: ["android", "kotlin", "KFunction", "KFunction1", "KSuspendFunction", "KSuspendFunction1"]
date: 2019-05-08T17:54:58+02:00
draft: true
---

KFunction can be written in more than one way. Here's a list of analogues.

<!--more-->

After my [previous article on Reflection]({{% ref "/post/generic_interface_and_methods" %}}) was posted, I got a great comment from @pacoworks:

{{< tweet 1126057942724296704 >}}

Turns out I was wrong (click that thread to read more) and KSuspendFunction can be shortened. So, in order to help me remember the different ways to reference functions, I made this grid. It omits the list of passed interfaces for brevity.


| Function  | Analogue1 | Analogue2 |
| ------------- | ------------- | ------------- |
| do(function: KFunction1\<Interface, Result\>){} | do(function: (Interface) -> Result){} | do(function: Interface.() -> Result){} |
| do(function: KFunction2\<Interface, In, Result\>, input: In){} | do(function: (Interface, In)->Result, input: In){} | do(function: Interface.(In)->Result, input: In){} |
| do(function: KFunction3\<Interface, In1, In2, Result\>, in1: In1, in2: In2){} | do(function: (Interface, In1, In2)->Result, in1: In1, in2: In2){} | do(function: Interface.(In1, In2)->Result, in1: In1, in2: In2){} |
| do(function: KSuspendFunction1\<Interface, Result\>){} | do(function: suspend (Interface) -> Result){} | do(function: suspend Interface.() -> Result){} |
| do(function: KSuspendFunction2\<Interface, Input, Result\>){} | do(function: suspend (Interface, Input) -> Result){} | do(function: suspend Interface.(Input) -> Result){} |

Right now, I don't know if there is any difference between these syntaxes.