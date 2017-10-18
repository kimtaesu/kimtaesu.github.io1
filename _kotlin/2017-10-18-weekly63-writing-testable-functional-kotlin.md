---
layout: post
title:  "Weekly 63 Writing testable functional Kotlin"
date:   2017-10-18 0:10:00
description: Writing testable functional Kotlin
tag:
 - weekly 63
 - kotlin
toc: true
---

# Writing testable functional Kotlin
[원문][source]

Dependency injection은 객체 지향 Java에서 매우 일반적으로 사용되는 패턴입니다.
Dagger, Spring 또는 Guice와 같은 프레임 워크를 사용하여 수행되는 경우가 있으며 때로는 클래스의 모든 종속성이 생성자를 통해 제공되는 것처럼 간단합니다.
dependency injection의 주요 장점 중 하나는 **mock 주입을 통해 Test 용 클래스를 분리 할 수 있다는 것**입니다.


{% highlight kotlin  %}
import java.io.PrintStream

class FizzBuzzRunner(
        val linePrinter: PrintStream // class dependency
) {
    fun fizzBuzz(val limit: Int) {
        for (i in 1..limit) {
            linePrinter.println(calculateValueForNumber(i))
        }
    }

    private fun calculateValueForNumber(number: Int): String {
        val sb = StringBuilder()
        if (number % 3 == 0) {
            sb.append("FIZZ")
        }
        if (number % 5 == 0) {
            sb.append("BUZZ")
        }
        if (sb.isEmpty()) {
            sb.append(number)
        }
        return sb.toString()
    }
}
{% endhighlight %}

함수는 다른 함수를 호출합니다. 
한 함수가 다른 함수를 호출 할 때 호출자 함수를 어떻게 테스트 할 수 있습니까?

FizzBuzz 코드:

{% highlight kotlin  %}
fun fizzBuzz(limit: Int) {
    for (i in 1..limit) {
        print(calculateValueForNumber(i))
    }
}

fun calculateValueForNumber(number: Int): String {
    val sb = StringBuilder()
    if (number % 3 == 0) {
        sb.append("FIZZ")
    }
    if (number % 5 == 0) {
        sb.append("BUZZ")
    }
    if (sb.isEmpty()) {
        sb.append(number)
    }
    return sb.toString()
}


fun print(string: String) {
    System.out.println(string)
}
{% endhighlight %}

`fizzBuzz` 함수안에 `print()` 를 Mocking할 방법이 없습니다. 
다른 한 가지 방법은 함수가 호출을 하도록 하고 component-style 로 Test를 하는 것 입니다.
그러나 데이터베이스 호출 I/O와 같이 시스템 경계에서 무언가를 호출하면 그렇게 잘 작동하지 않습니다. 
그래서 다른 접근법을 취했습니다. 

fizzBuzz 함수는 다음과 같이 보입니다.

{% highlight kotlin  %}
fun fizzBuzz(limit: Int, printer :(String) -> Unit = ::print) {
    for (i in 1..limit) {
        printer.invoke(calculateValueForNumber(i))
    }
}
{% endhighlight %}

Product Code 에서 똑같은 방식으로 호출 할 수 있습니다.

{% highlight kotlin  %}
fizzbuzz(4)
{% endhighlight %}
 

이런 테스트를 위해 격리 될 수 있습니다.

{% highlight kotlin  %}
fun test() {
    val outputList = ArrayList<String>()
    fizzBuzz(6, {outputList.add(it)})
    assert(outputList[2] == "FIZZ")
}
{% endhighlight %}

이 접근법의 장점은 개체 중심의 "edge" 코드에서 순수한 "core"기능을 개체에 주입하여 호출하는 것입니다.

{% highlight kotlin  %}
class FizzBuzzRunner(
        private val printer: (String) -> Unit = ::print,
        private val calculator: (Int) -> String = ::calculateForNumber) {
    
    fun fizzBuzz(limit: Int) {
        for (i in 1..limit) {
            printer.invoke(calculator.invoke(i))
        }
    }
}

fun print(string: String) {
    System.out.println(string)
}

fun calculateForNumber(number: Int): String {
    val sb = StringBuilder()
    if (number % 3 == 0) {
        sb.append("FIZZ")
    }
    if (number % 5 == 0) {
        sb.append("BUZZ")
    }
    if (sb.isEmpty()) {
        sb.append(number)
    }
    return sb.toString()
}
{% endhighlight %}

Test 할 수 있는 Functional Kotlin을 작성하는 것 중 간단한 패턴이며 다른 사람들에게 유용하기를 희망합니다.


  [source]: https://medium.com/@wakingrufus/writing-testable-functional-kotlin-2c06ec01dc02
  [github]: https://github.com/dmytrodanylyk/coroutine-recipes