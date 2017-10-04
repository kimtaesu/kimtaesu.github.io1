---
layout: post
title:  "Android Clean Architecture (2/2)"
date:   2017-10-04 0:10:00
description: Clean Architecture  (2/2)
tags:
- Clean Architecture
---

# [Android Clean Architecture  (2/2)](https://github.com/bufferapp/clean-architecture-components-boilerplate)

이전 [Article](Android-Clean-Architecture1.html)에서는 
Android Clean Architecture의 전반적인 Architecture 를 살펴보았습니다. 

앞으로 훝어볼 내용은 Test Code 기반의 각 모듈이 하고자 하는 것을 세부적으로 소개할 예정입니다.

# Test Code
먼저 살펴볼 것은 **cache, remote** 모듈 입니다. 

그 이유는 상위 Layer는 추상화를 포함하고 있기 때문에 하위 Layer의 Concreate를 먼저 파고들어가는 것이 이해하는데 훨씬 수월하기 때문입니다. 

## cache

