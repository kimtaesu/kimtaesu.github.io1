---
layout: post
title:  "Weekly 61 Infix function in Kotlin"
date:   2017-09-25 0:10:00
description: Infix function in Kotlin
tag:
 - weekly 61
 - kotlin
toc: true
---

# Infix function in Kotlin
[참조](https://medium.com/makingtuenti/infix-functions-in-kotlin-2db3d3142dd2)

한국어로는 중위 표기법이라고 하는데 Kotlin은 이 중위 표기법(Infix notation)을 지원합니다.

"."를 객체 내 함수를 호출하는 것이 아니고 infix 선언된 함수명을 통하여 접근이 가능합니다. 

infix는 더 자연스럽게 읽고 가독성있게 만드는데도 도움이됩니다.

예제를 살펴보도록 합시다. 

{% highlight kotlin  %}
enum class Suit {
    HEARTS,
    SPADES,
    CLUBS,
    DIAMONDS
}
{% endhighlight %}

infix로 `of` 라는 함수가 선언되어 있습니다. 

{% highlight kotlin  %}
enum class Rank {
    TWO, THREE, FOUR, FIVE,
    SIX, SEVEN, EIGHT, NINE,
    TEN, JACK, QUEEN, KING, ACE;

    infix fun of(suit: Suit) = Card(this, suit)
}

data class Card(val rank: Rank, val suit: Suit)
{% endhighlight %}

실제로 사용하는 측에서 보면 `of`라는 표기할 수 있도록 합니다. 

{% highlight kotlin  %}
fun main(args: Array<String>) {
    val card = Rank.QUEEN of Suit.HEARTS
}

{% endhighlight %}
