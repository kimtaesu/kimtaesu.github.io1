---
layout: post
title:  "Android Clean Architecture"
date:   2017-10-04 0:10:00
description: Clean Architecture
tags:
- Clean Architecture
---

# [Android Clean Architecture](https://github.com/bufferapp/clean-architecture-components-boilerplate)


원래의 Clean Architecture Boilerplate를 fork 한 것입니다
지금 Repo 에서는
* Presentation Layer : MVP 접근 방식을 전환하여 ViewModels를 사용합니다.
* Caching Layer : Room을 사용합니다.

이 boilerplate가 다른 개발에게 도움이 될 뿐만 아니라, architecture에 교육의 도움이 됩니다.
우리 몇 가지 이유로 boilerplate 를 만들었습니다. 

1. 모듈화의 실험하기
2. Android Architecture 구성 요소로 실험하기 
3. Clean Architecture 공유, 특별히 이것에 대해 [많은 이야기를](https://academy.realm.io/posts/converting-an-app-to-use-clean-architecture/) 해 온 것
4. Clean Architecture 앞으로의 프로젝트에서 출발점으로 사용하기하기 

Kotlin으로 100% 작성되어 있습니다. 

## Disclaimer

Sample에서 clean architecture는 over-complicated 되어 보입니다.  
그러나 이렇게하면 상용구 코드의 양을 최소한으로 유지하면서 간단한 방식으로 접근 방식을 입증 할 수 있습니다.
클린 아키텍처는 모든 프로젝트에 적합하지 않으므로 필요에 맞게 결정해야합니다.

## Languages, libraries and tools used

* [Kotlin](https://kotlinlang.org/)
* [Room](https://developer.android.com/topic/libraries/architecture/room.html)
* [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)
* Android Support Libraries
* [RxJava2](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)
* [Dagger 2 (2.11)](https://github.com/google/dagger)
* [Glide](https://github.com/bumptech/glide)
* [Retrofit](http://square.github.io/retrofit/)
* [OkHttp](http://square.github.io/okhttp/)
* [Gson](https://github.com/google/gson)
* [Timber](https://github.com/JakeWharton/timber)
* [Mockito](http://site.mockito.org/)
* [Espresso](https://developer.android.com/training/testing/espresso/index.html)
* [Robolectric](http://robolectric.org/)


## Requirements

* JDK 1.8
* [Android SDK](https://developer.android.com/studio/index.html)
* Android O ([API 26](https://developer.android.com/preview/api-overview.html))
* Latest Android SDK Tools and build tools.


## Architecture

<img width="100%" src="https://github.com/bufferapp/clean-architecture-components-boilerplate/blob/master/art/architecture.png?raw=true">
<img width="100%" src="https://github.com/bufferapp/android-clean-architecture-boilerplate/blob/master/art/ui.png?raw=true">

## Dependencies Graph

<img width="100%" src="https://github.com/bufferapp/android-clean-architecture-boilerplate/blob/master/art/ui.png?raw=true">

## Reference
* [https://github.com/android10/Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)
* [https://github.com/googlesamples/android-architecture](https://github.com/googlesamples/android-architecture)