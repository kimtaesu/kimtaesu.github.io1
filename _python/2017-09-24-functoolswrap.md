---
layout: post
title:  "Python functools.wrap"
date:   2017-09-24 0:10:00
description: Python 내장 모듈 functools.wrap
tags:
- Python
- functools.wrap
toc: true
---

# Functools.wrap 함수 데코레이터

데코레이터는 감싸고 있는 함수를 호출하기 전이나 후에 추가로 코드를 실행하는 기능
* 시맨틱 강조
* 디버깅
* 함수 등록 

{% highlight python  %}

def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print('%s(%r, %r) -> %r' %
              (func.__name__, args, kwargs, result))
        return result
    return wrapper


@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))


def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))

fibonacci = trace(fibonacci)


fibonacci(3)
# fibonacci((1,), {}) -> 1
# fibonacci((0,), {}) -> 0
# fibonacci((1,), {}) -> 1
# fibonacci((2,), {}) -> 1
# fibonacci((3,), {}) -> 2

print(fibonacci)
# <function trace.<locals>.wrapper at 0x1023e9f28>
{% endhighlight %}

이 코드는 잘 동작하지만 의도하지 않은 부작용을 일으킨다.
   
앞에서 호출한 함수의 이름이 **fibonacci**가 아니다.
 
원인 : trace 함수는 그 안에 정의된 wrapper 를 반환한다. 

해결 방법 : 내장 모듈 functools의 wraps 헬퍼 함수를 사용하는 것

{% highlight python  %}

from functools import wraps
def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print('%s(%r, %r) -> %r' %
              (func.__name__, args, kwargs, result))
        return result
    return wrapper

@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) +
            fibonacci(n - 1))

print(fibonacci)
# <function **fibonacci** at 0x102518730>
{% endhighlight %}
