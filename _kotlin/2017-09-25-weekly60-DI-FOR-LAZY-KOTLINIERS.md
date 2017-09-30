---
layout: post
title:  "Weekly 60 DI FOR LAZY KOTLINIERS"
date:   2017-09-25 0:10:00
description: Kotlin DI FOR LAZY KOTLINIERS
tag:
 - weekly 60
 - kotlin
toc: true
---

# DI FOR LAZY KOTLINIERS
## [Article][source]


Acticle에서 사용되는 Framework는 Android 이나,  편의상 Kotlin으로 대체하여 설명하도록 하겠습니다. 

{% highlight kotlin  %}

interface Config {
    val properties : Map<String, Any?>
}
{% endhighlight %}

Dagger2와 같이 config의 기본 위치를 Application 개체에 적용한다. 
 
{% highlight kotlin  %}

class App() : /* Application() */{
    companion object {
        var app: App? = null

        fun getConfig() = hashMapOf(
                "myName" to "Jan Kotlin",
                "imageLoader" to object : ImageLoader {
                    override fun loadImage(url: String, view: String) {
                        println(view)
                    }
                }
        )
    }
}

{% endhighlight %}


{% highlight kotlin  %}

interface ImageLoader {
    fun loadImage(url: String, view: String)
}
{% endhighlight %}

{% highlight kotlin  %}

class ConfigDelegate : Config {
    override val properties: Map<String, Any?> by lazy {
        App.getConfig()
    }
}

{% endhighlight %}


{% highlight kotlin  %}

class MainActivity : Config by ConfigDelegate() {

    private val myName: String by properties
    
    private val imageLoader: ImageLoader by properties


    fun onCreate() {
        println("${myName}")
        
        imageLoader.loadImage("http://www.ourhenhouse.org/wp-content/uploads/2013/01/kitten_lily.jpg", "ImageView")
        
    }
}
{% endhighlight %}



결과 : 
```
Jan Kotlin
ImageView
```

# 차근차근 하나하나씩 해체해보자

### 1. "myName", "imageLoader" by properties 는 **Map delegation** 이다.

{% highlight kotlin  %}

private val myName: String by properties
private val imageLoader: ImageLoader by properties

{% endhighlight %}

[Map Delegation][kotlin 1]

<img width="720" height="700" class="margin-top: auto; margin-bottom: auto;" src="/images/kotlin/StoringPropertiesInaMap.png"/>


### 2. by ConfigDelegate() 는 **class delegation** 이다.
 
{% highlight kotlin  %}

class MainActivity : Config **by ConfigDelegate()** {
}

{% endhighlight %}

[Class Delegation][kotlin 2] 

Class Delagation은 Design Pattern 중 Decorater에서 소개한 적이 있다.
 
[Github][decorator]
 
  [source]: https://jankotlin.wordpress.com/2017/09/18/di-for-lazy-kotliniers/
  [kotlin 1]: https://kotlinlang.org/docs/reference/delegated-properties.html
  [kotlin 2]: https://kotlinlang.org/docs/reference/delegation.html
  [decorator]: https://github.com/kimtaesu/Design-Patterns-In-Kotlin/blob/master/src/main/kotlin/Decorator.kt
  
  
  