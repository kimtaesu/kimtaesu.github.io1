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


# contextmanager with 
ContextManager의 Usage : 

{% highlight python  %}
    Typical usage:

@contextmanager
def some_generator(<arguments>):
    <setup>
    try:
        yield <value>
    finally:
        <cleanup>
{% endhighlight %}


Exmaple 
{% highlight python  %}
@contextmanager
def log_level(level, name):
    logger = logging.getLogger(name)
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield logger
    finally:
        logger.setLevel(old_level)

with log_level(logging.DEBUG, 'my-log') as logger:
    logger.debug('This is my message!')
    logging.debug('This will not print')
    
logger = logging.getLogger('my-log')
logger.debug('Debug will not print')
logger.error('Error will print')

{% endhighlight %}

Result
```
ERROR:my-log:Error will print
```
