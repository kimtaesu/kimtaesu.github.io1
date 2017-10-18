---
layout: post
title:  "Weekly 63 Programmer dictionary Parameter vs Argument, Type parameter vs Type argument"
date:   2017-10-18 0:10:00
description: Programmer dictionary Parameter vs Argument, Type parameter vs Type argument
tag:
 - weekly 63
 - kotlin
toc: true
---

# Programmer dictionary: Parameter vs Argument, Type parameter vs Type argument
[원문][source]

**parameter**와 **argument**는 혼동스럽지만 완전히 다른 개념입니다. 
그 차이점에 대해 논의하며 타입 인자와 **type parameter**와 **type argument**가 무엇인지를 이해할 것이다.

## Parameter vs Argument

**parameter**는 함수 정의에 정의 된 변수이며 **argument**는 함수에 전달 된 실제 값입니다. 
차이점을 이해하기 위해 먼저 예제 함수와 그 사용법을 살펴 보겠습니다.


{% highlight kotlin  %}

fun randomString(length: Int): String {
    // ....
}
randomString(10)
{% endhighlight %}

위 예제에서 `length`는 **parameter**이고 `10`은  **argument**입니다. 

> 일반적인 정의 : parameter는 함수 선언에 정의 된 변수입니다. argument는 함수에 전달되는 변수의 실제 값입니다.

## Type parameter vs Type argument

example : 

{% highlight kotlin  %}
class Box<T>

val a: Box<Int> = Box()
{% endhighlight %}

`Box`는 `T` **type parameter**를 정의하는 generic class입니다. 
`Int`로 지정한 것은 **type argument**입니다. 

> 일반적인 정의 : type parameter 는 generic으로 선언 된 blueprint 또는 placeholder 입니다. type argument는 generic type parameter에 사용되는 실제 유형입니다.

  [source]: https://medium.com/kotlin-academy/programmer-dictionary-parameter-vs-argument-type-parameter-vs-type-argument-b965d2cc6929