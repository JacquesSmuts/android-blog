---
title: "Add Overlays to Your Launcher Icon Using Layer-List"
author: "Jacques"
image: "/images/layerlist3.png"
tags: ["Android", "ic_launcher.png", "Launcher Icon", "Layer-List as launcher"]
date: 2020-02-16T14:54:38+02:00
draft: false
---

You can use a <layer-list> drawable to easily differentiate between flavors and variants. This will show you how.

<!--more-->
## Why?

Ever wanted to have different icons for each flavor and variant in your Android app? Something like a little bug to indicate the debug version of your app, like so:

{{< figure src="/images/layerlist4.png" alt="An inordinate fondness for beetles" title="" width="40%" >}}

Sure, but it's effort. And you have to ask a designer to make a bunch of different icons. Or, even worse, you have to create a bunch of different icons yourself. That's wasteful, so instead you should use a layer-list. It has a default icon, and then it can dynamically insert an overlay image on top of your launcher icon, based upon your flavor or build variant. That way, you don't need a bunch of different icons.

## How?

Assuming you have `mipmap/ic_launcher` icon already, you should

1.) Create an `ic_launcher_overlaid.xml` file in your `main/drawable` folder, with the following content:

{{< gist "JacquesSmuts" "3541b17741c40e3c24453830d0ee102a" >}}

How does this work? The layer-list puts the `launcher_overlay` on top of the normal `ic_launcher` image, then puts 100px padding to the top and left, so that the `launcher_overlay` sits in the bottom right corner. That might take some adjustment to get things right.

{{< figure src="/images/layerlist1.png" alt="Default Launcher Icon" title="drawable/ic_launcher." width="10%" class="zoomable" >}}

2.) update your `AndroidManifest.xml` file to point to the new launcher icon.

{{< gist "JacquesSmuts" "fcd8ba8abc4df58909c7c624d53d9127" >}}

The only thing left is:

3.) Create `drawable/launcher_overlay` for each variant/flavor you need.

You can create a single transparent pixel in `main/drawable` as the default. Then add the appropriate images you need in the `flavorVariant/drawable` folders. I added this image to my debug folder:

{{< figure src="/images/layerlist2.png" alt="An inordinate fondness for beetles" title="Beetle" width="10%" class="zoomable" >}}

 If all goes well, you'll end up with a launcher icon that looks like this:

 {{< figure src="/images/layerlist3.png" alt="An inordinate fondness for beetles" title="" width="10%" class="zoomable" >}}

 If you're wondering about Adaptive Icons, all you have to know is that the adaptive icon references a foreground drawable. In the same way that you can replace the `ic_launcher` with a layer-list, you can also replace `ic_launcher_foreground` with a layer-list.


 ---

 I'd like to thank my colleague, Laurie Scheepers, for suggesting we try xml drawables as app icons.
