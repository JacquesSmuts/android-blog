---
title: Further Reading for Bad Internet, Old Phones
description: This is an accompanying post for my presentation on handling Bad Internet and Old Phones
slug: bad-internet
date: 2022-11-21 00:00:00+0000
image: images/bad_internet_header.png
categories:
  - Android
  - Presentations
tags:
  - architecture
  - bad internet
  - old phones
---

## Slides

{{< rawhtml >}}
<iframe src="https://slides.com/jacquessmuts/deck/embed?style=hidden" width="640" height="360" title="bob" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
{{< /rawhtml >}}

## TL;DW

The point of my talk is that all the best practices that apply under normal circumstances are doubly necessary under difficult circumstances, such as building for bad internet, old phones, small screens, or technically inexperienced users.

There is no shortcut to learning best practices, but here are some good sources for further reading.

## Bad Internet Box

If you want to get your own Bad Internet Box, I suggest you contact [PJH Technologies](https://pjhtechnologies.com/).

## Google Tips

I feel that [this Google Architecture Recommendation Guide](https://developer.android.com/topic/architecture/recommendations) is right now the best starting point for building the architecture that allows you to more easily adapt for low connectivity and old phones.

Once you have the basic architecture set up, the Google [Build for Billions Guide](https://developer.android.com/docs/quality-guidelines/build-for-billions/connectivity
) and the [Google Performance Guidelines](https://developer.android.com/topic/performance) both have a myriad of great tips.

[This recent article](https://medium.com/androiddevelopers/inspecting-performance-95b76477a3d7) by Ben Weiss, as well as [this series of articles](https://android-developers.googleblog.com/2022/09/optimize-for-android-go-lessons-from-google-apps-part-1.html) by Niharika Arora on optimizing for Android Go, are both good supplements to the above.

## Other Reading

The Http client standard is [hosted here](https://www.rfc-editor.org/rfc/rfc9110.html#name-idempotent-methods), with a direct link to how they define idempotent methods.

[This medium article](https://medium.com/inloopx/okhttp-is-quietly-retrying-requests-is-your-api-ready-19489ef35ace) has a good warning about how OkHttp default retry works.

## Video

{{< youtube "jaZ2gLMGUsM" >}}
This video explains the complex logic behind offline-first apps, in an extremely simple and straightforward way. I consider offline first architecture to be the most important first step towards supporting low connectivity environments. However, it is part of a larger [series of videos on modern android development](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc_L3n1j4ajHjJ6QccFUvW1u) which I also recommend.

## Also

{{< youtube "aAzW12BcvGk" >}}

This video helped me a lot to build the skills to understand and implement a lot of the above, and I just like recommending it because it's a timeless and important presentation.

