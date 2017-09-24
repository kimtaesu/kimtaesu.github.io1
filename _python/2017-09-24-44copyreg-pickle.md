---
layout: post
title:  "Python contextmanager"
date:   2017-09-24 0:10:00
description: Python 내장 모듈 contextmanager
tags:
- Python
- contextmanager
toc: true
---


# Copyreg

pickle 로 직렬화 / 역직렬화를 통한 상태 저장은 위험할 수 있다.

{% highlight python  %}
class Exmaple(object):
    def __init__(self, a = 0):
        self.a = a

copyreg.pickle(ob_Type, pickle_function)

{% endhighlight %}


