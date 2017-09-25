---
layout: post
title:  "Weekly #60 1. Spek your reactive stream using PublishSubjects"
date:   2017-09-25 0:10:00
description: Python 내장 모듈 contextmanager
tags:
- Python
- contextmanager
toc: true
---

# KOTLIN REIFIED TYPES IN INLINE FUNCTIONS
## [Article][source]


## Starting situation
{% highlight kotlin  %}
fun <T> myGenericFun(c: Class<T>)
{% endhighlight %}

`myGenericFun` 는 자바와 동일하게 T type은 접근할 수 업습니다. 

**Runtime 시에는 지워지므로 Compile time**에만 접근할 수 있기 때문입니다.

반면에
{% highlight kotlin  %}

inline fun <reified T> myGenericFun()

{% endhighlight %}

__reified T__ 는 접근이 가능합니다. 

> **Note**
> inline function에서만 접근이 가능합니다.
> inline 함수는 컴파일러가 함수의 바이트 코드를 함수가 호출되는 모든 위치에 복사하기 때문입니다. 
 


아래는 소스는 컴파일 에러가 발생합니다. 
* inline 함수가 아님 
* reified generic type 이 아님

{% highlight kotlin  %}
fun <T> getTNameOriginal(t: T): String {
//    Compile Error 
    return T::class.java.name
}
{% endhighlight %}

reified keyword 없이 컴파일이 가능합니다. 

{% highlight kotlin  %}
fun <T> getTNameWithOutReified(t: Class<T>): String {
    return t.name
}

{% endhighlight %}

그러나 Class<> 라는 지저분한 코드가 포함됩니다.

결과적으로 reified 로 사용한 코드입니다. 

{% highlight kotlin  %}
inline fun <reified T> getTName(t: T): String {
    return T::class.java.name
}
{% endhighlight %}



  [source]: https://simpleprogrammer.com/products/learn-anything/