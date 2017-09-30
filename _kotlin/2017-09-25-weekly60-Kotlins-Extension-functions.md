---
layout: post
title:  "Weekly 60 Getting to know Kotlin’s Extension functions: some caveats to keep in mind"
date:   2017-09-25 0:10:00
description: Getting to know Kotlins Extension functions some caveats to keep in mind
tag:
 - weekly 60
 - kotlin
toc: true
---

# Getting to know Kotlin’s Extension functions: some caveats to keep in mind
## [Article][source]

Kotlin을 쓰고 있다면, 
당신이 이미 알고 있거나 짐작할 수 있듯이, 확장 기능이라고하는 것과 같은 것이 있습니다. 
확장 기능이 클래스의 범위 밖에서 선언 될 수 있기 때문에 소스 코드로 이동할 필요없이 클래스에 동작을 추가 할 수 있습니다. 그렇게함으로써 코드의 가독성을 높일 수 있습니다. 굉장해!

확장 함수는 정말 간단한 트릭입니다.
확장 함수는 동작을 추가 할 클래스의 바이트 코드를 수정하지 않습니다. 
확장 함수는 메소드를 호출하는 클래스의 객체 인 수신기를 첫 번째 인수로 명시 적으로 지정한 
추가 매개 변수와 함께 취하는 메소드입니다. 
또한 최상위 함수로 선언 된 경우에는 바이트 코드에서 정적 메서드로 컴파일됩니다.

간단한 예제를 살펴봅시다. 
{% highlight kotlin  %}
fun Activity.toast(text: String) {
    ...
}
{% endhighlight %}

디 컴파일 된 Java 로 보면 아래와 같습니다. 
{% highlight kotlin  %}
public static final void toast(@NotNull Activity $receiver, @NotNull String text) {
    ...
}
{% endhighlight %}

## Static vs dynamic dispatching

확장 기능을 깊이 파고 들기 전에 정적 및 동적 디스 패칭 간의 차이점을 명심해야합니다. 
아시다시피 **Java는 정적으로 형식화 된 언어**이며 
모든 객체는 런타임뿐만 아니라 개발자가 명시 적으로 지정해야하는 컴파일 시간 유형 
(또는 Kotlin에서 유추 할 수 있음)을 가집니다.

Base base = new Extended()라고 말할 때,
* 컴파일 타임 타입 Base 
* 런타임 타입 Extended

base.foo ()를 호출하면 메서드가 동적으로 전달되므로 런타임 유형 (확장) 메서드가 호출됩니다.

{% highlight kotlin  %}

void foo(Base base) {
    println(base)
}
void foo(Extended extended) {
    println(extended)
}
public static void main(String[] args) {
    Base base = new Extended();
    foo(base);
}
{% endhighlight %}

println : `base`

# Extension functions are always dispatched statically

확장 기능은 항상 정적으로 전달됩니다.
리시버는 실제로 바이트 코드에서 컴파일 된 메소드의 매개 변수이기 때문에 오버로드 할 수는 있지만 오버라이드 할 수는 없습니다. 이것은 아마도 멤버와 확장 함수 사이의 가장 중요한 차이 일 것입니다. 전자가 동적으로 전달되는 반면 후자는 항상 정적입니다.


이해하기 쉽도록 아래 예제를 살펴보도록 하겠습니다. 


{% highlight kotlin  %}
open class Base
class Extended: Base()
fun Base.foo() = "Base!"
fun Extended.foo() = "Extended!"
fun main(args: Array<String>) {
    val instance: Base = Extended()
    val instance2 = Extended()
    println(instance.foo())
    // Base!
    println(instance2.foo())
    // Extended!
}
{% endhighlight %}

println : `Base!`
println : `Extended!`

컴파일 타임 유형 만 고려되므로 첫 번째 Base.foo () 
두 번째는 Extended.foo () 메서드가 호출됩니다.

# Extension functions inside a class

클래스 내부에 확장 함수를 선언하면 정적이 아니며 open로 선언 된 경우에도이를 재정의 할 수 있습니다. 
확장 기능이 클래스에서 선언되면 디스패치 수신기와 확장 수신기가 둘 다 있습니다.

{% highlight kotlin  %}
class A {
    fun B.foo() = "Hello!"
}
{% endhighlight %}

위의 경우 A는 디스패치 수신기이고 B는 확장 수신기입니다. 
확장 함수를 open으로 선언하면 dispatching은 디스패치 수신기와 관련해서만 동적 일 수 있으며 
확장 수신기는 항상 컴파일 타임에 해석됩니다.

위 개념을 예제로 이해해봅시다. 
{% highlight kotlin  %}
open class Base
class Extended : Base()
open class A {
    open fun Base.foo() {
        println("Base.foo in A")
    }
    open fun Extended.foo() {
        println("Extended.foo in A")
    }
    fun caller(base: Base) {
        base.foo()
    }
}
class B : A() {
    override fun Base.foo() {
        println("Base.foo in B")
    }
    override fun Extended.foo() {
        println("Extended.foo in B")
    }
}

fun main(args: Array<String>) {
    A().caller(Base())   
    // prints "Base.foo in A"
    B().caller(Base())  
    // prints "Base.foo in B" - dispatch receiver is resolved dynamically
    A().caller(Extended())  
    // prints "Base.foo in A" - extension receiver is resolved statically
}
{% endhighlight %}

아시다시피 `Extend` 확장 기능은 호출되지 않으며 
이 동작은 정적 디스패치 기간 동안 이전에 보았던 것과 동일합니다. 
실제로 변경되는 것은 호출자를 호출하는 클래스의 런타임 유형에 따라 
두 개의 기본 확장 함수 중 어느 것이 호출되는지입니다.

"Extended.foo in A" 를 호출하기 위해선 
caller의 parameter를 변경해야 한다. 
{% highlight kotlin  %}
    fun caller(base: Extended) {
        base.foo()
    }
{% endhighlight %}    


# Extension functions vs member functions

{% highlight kotlin  %}
class Person {
    fun walk() {
        println("walk in Person")
    }
    fun Person.walk() {
        println("Person.walk in Person")
    }
}
fun Person.walk() {
    println("Person.walk")
}
{% endhighlight %}

함수의 충돌로 컴파일되지 않을 것이라 예상했겠지만, 실제로 동작한다. 
첫 번째 함수만 사용되며 다른 함수는 무시된다. 

# Visibility

확장 기능에서 public 속성과 메서드에만 액세스 할 수 있습니다.

{% highlight kotlin  %}
class Hero(
   val name: String,
   protected val surname: String,
   private val nickname: String
)
fun Hero.getIdentity() = "$name $surname, $nickname"
{% endhighlight %}

Compile Error `$surname, $nickname"`

영웅 인스턴스를 Input으로 사용하는 메서드를 생성하므로 당연히 공개 필드에만 액세스 할 수있게 될 것이므로 큰 놀라움은 아닙니다. 

  [source]: https://medium.com/@quiro91/getting-to-know-kotlins-extension-functions-some-caveats-to-keep-in-mind-d14d734d108b
  
  
  