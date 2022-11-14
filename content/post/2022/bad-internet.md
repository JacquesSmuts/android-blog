---
title: Further Reading for Bad Internet, Old Phones
description: This is an accompanying post for my presentation on handling Bad Internet and Old Phones
slug: bad-internet
date: 2022-11-07 00:00:00+0000
image: images/bad_internet_header.png
categories:
  - Android
  - Presentations
tags:
  - architecture
  - bad internet
  - old phones
draft: true
---

## Google Tips

[This Google Architecture Recommendation Guide](https://developer.android.com/topic/architecture/recommendations) is the best starting point for building the architecture that allows you to more easily adapt for low connectivity and old phones:

Once you have the basic architecture set up, the Google [Build for Billions Guide](https://developer.android.com/docs/quality-guidelines/build-for-billions/connectivity
) and the [Google Performance Guidelines](https://developer.android.com/topic/performance) both have a myriad of great tips.

I found that [this recent article](https://medium.com/androiddevelopers/inspecting-performance-95b76477a3d7) by Ben Weiss, as well as [this series of articles](https://android-developers.googleblog.com/2022/09/optimize-for-android-go-lessons-from-google-apps-part-1.html) by Niharika Arora on optimizing for Android Go, are both good supplements to the above.

## Other Reading

The Http client standard is [hosted here](https://www.rfc-editor.org/rfc/rfc9110.html#name-idempotent-methods), with a direct link to how they define idempotent methods.

[This medium article](https://medium.com/inloopx/okhttp-is-quietly-retrying-requests-is-your-api-ready-19489ef35ace) has a good warning about how OkHttp default retry works.

## Video

{{< youtube "jaZ2gLMGUsM" >}}
This video explains the complex logic behind offline-first apps, in an extremely simple and straightforward way. I consider offline first architecture to be the most important way to support low connectivity environments.

## Also

{{< youtube "aAzW12BcvGk" >}}

This video helped me a lot to build the skills to understand and implement a lot of the above, and I just like recommending it.

