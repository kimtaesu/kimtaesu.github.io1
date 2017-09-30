---
layout: post
title:  "Weekly #60 3. Spek your reactive stream using PublishSubjects"
date:   2017-09-25 0:10:00
description: Spek 를 활용한 Rx Test code 
toc: true
---

# Spek your reactive stream using PublishSubjects
## [Article][source]
## [Github][github]
Spek 프레임 워크를 활용하여 RxJava 코드가 어떻게 구체화 될 수 있는지에 대한 간단한 예를 제공하고 

PublishSubjects를 활용하여 작가와 독자 모두에게 즐거운 단위 테스트를 제공하는 것을 목표로 합니다.


## 시나리오
DistancesProvider는 한 위치에서 다른 위치에 킬로미터와 마일 간격을 제공합니다.

{% highlight kotlin  %}

fun distances() : Observable<List<Int>> // [kilometres, miles]

fun unit() : Observable<Int> // 0 -> kilometres, 1 -> miles
{% endhighlight %}

위 두 Provider를 사용하여 실시간으로 거리를 방출하는 GPS 시스템을 만들수 있습니다.

{% highlight kotlin  %}

fun distances(): Observable<Int> {
    return Observable.combineLatest(
            distancesProvider.distances(),
            unitProvider.unit(),
            { distances, unit -> distances[unit] })
            // distances[unit] // 0 -> kilometres, 1 -> miles
}

{% endhighlight %}

Spek DSL을 사용하는 가능한 사양은 다음과 같습니다. 
30km의 거리 업데이트시 GPS 시스템에서 값 30을 방출해야합니다.

{% highlight kotlin  %}

given("a GPS system") {

    on("a distance update of 30 km") {
        
        it("should emit the value 30") {
            
        }
    }
}
{% endhighlight %}

Spek는 실제로 테스트를 실행하기 위해 _beforeEachTest_ 및 _afterEachTest_를 사용하여 사전 및 사후 조건을 정의 할 수 있습니다.

이는 Junit의 @Before @After Annotation과 같습니다.

DistancesProvider와 UnitProvider를 쉽게 테스트 할 수 있는 *DistancesUseCase*를 만듭니다. 

{% highlight kotlin  %}

val distancesProvider: DistancesProvider = mock()
val unitProvider: UnitProvider = mock()

val tested = DistancesUseCase(distancesProvider, unitProvider)
whenever(distancesProvider.distances()).thenReturn(...)
whenever(unitProvider.unit()).thenReturn(...)

{% endhighlight %}

{% highlight kotlin  %}
class DistancesUseCase(val distancesProvider: DistancesProvider,
                       val unitProvider: UnitProvider) {
    fun distances(): Observable<Int> {
        return Observable.combineLatest(
                distancesProvider.distances(),
                unitProvider.unit(),
                BiFunction({ distances, unit -> distances[unit] })
        )
    }
}
{% endhighlight %}

아래는 Article보다는 나의 개인적인 생각을 정리했다.  

명확히 테스트를 하고자하는 것이 무엇인지 정의가 필요하다. 
DistancesProvider, UnitProvide는 잘 작동한다는 가정하에 DistancesUseCase를 테스트하는 것이다.  
DistancesUseCase만 실제 객체이며, DistancesProvider, UnitProvide는  Mock 객체로 만들었다.  

  

{% highlight kotlin  %}

val distancesProvider: DistancesProvider = mock()
val unitProvider: UnitProvider = mock()
    
var distanceSub: PublishSubject<List<Int>> = PublishSubject.create()
var unitSub: PublishSubject<Int> = PublishSubject.create()
whenever(distancesProvider.distances()).thenReturn(distanceSub)
whenever(unitProvider.unit()).thenReturn(unitSub)
{% endhighlight %}


일반적으로 테스트에 따라 특정 값을 반환합니다. 예를 들어,
* distance () 메소드의 Observable.just (listOf (30, 19))
* Unit () 메소드의 Observable.just (0)

{% highlight kotlin  %}

given("a GPS system") {
    beforeEachTest { ... }
    afterEachTest { ... }
    on("a distance update of 30 km") {
        distanceSub.onNext(listOf(30, 19)) // [30km, 19miles]
        unitSub.onNext(0) // km
}
{% endhighlight %}


아래는 Article보다는 나의 개인적인 생각을 정리했다.

Test의 흐름 순서 : given -> when -> then 

`tested.distances().test()` 를 호출 함으로써 **given** 환경의 사전 설정을 끝내도록 한다.
 
**then**에서는 `testSubscriber`로 검증할 수 있도록 한다.  

{% highlight kotlin  %}

...
var testSubscriber = TestObserver<Int>()
given("a GPS system") {
  beforeEachTest {
    ...
    testSubscriber = tested.distances().test()
  }

 {% endhighlight %}


이제 우리는 GPS 시스템을 테스트하는 데 필요한 모든 재료를 갖추고 있습니다. 
위의 사양에 따라 거리 업데이트 30km가 오면 시스템에서 값 30을 출력하는지 확인하고자합니다. 
distanceSub 및 unitSub을 사용하면 이 작업을 매우 쉽게 수행 할 수 있습니다.

[Spek][spek] 문서에서 알 수 있듯이, **on** 은 **(given, context, describe)**에 의하여 반복될 수 있다.
{% highlight kotlin  %}

var testSubscriber = TestObserver<Int>()
    given("a GPS system") {

       
        beforeEachTest { ... }
        afterEachTest { ... }

        on("a distance update of 30 km") {
            distanceSub.onNext(listOf(30, 19)) // [30km, 19miles]
            unitSub.onNext(0) // km

            it("should emit the value 30") {
                testSubscriber.assertValue(30)
            }
        }

 {% endhighlight %}

"should not complete" 테스트는 unitSub는 completed 되었을 때
testSubscriber는 complete 가 되지 않아야 한다는 assert를 보여준다.
  
이를 이해하기 위해선 [combineLatest][combinelatest]를 이해해야 한다. 

{% highlight kotlin  %}

on("a distance update of 30 km, no more units and more distances") {
        distanceSub.onNext(listOf(30, 19)) // [30km, 19miles]
        unitSub.onNext(0) // km
        unitSub.onCompleted()
        distanceSub.onNext(listOf(20, 12)) // [20km, 12miles]
        it("should emit the value 30 followed by value 20") {
            testSubscriber.assertValues(30, 20)
        }
        it("should not complete") {
             testSubscriber.assertNotCompleted()
        }
}
{% endhighlight %}

다음 테스트 케이스를 보자. 


아래 테스트는 실패한다. 

{% highlight kotlin  %}

on("a distance update of 30 km, miles, no units, more distances") {
        distanceSub.onNext(listOf(30, 19)) // [30km, 19miles]
        unitSub.onNext(0) // km
        unitSub.onNext(1) // miles
        unitSub.onCompleted()
        distanceSub.onNext(listOf(20, 12)) // [20km, 12miles]
        it("should emit the value 30 followed by value 12") {
            testSubscriber.assertValues(30, 12)
        }
}

java.lang.AssertionError: 
Number of items does not match. Provided: 2  Actual: 3.
Provided values: [30, 12]
Actual values: [30, 19, 12]

{% endhighlight %}
 
combineLatest는 각 스트림의 마지막 값을 가장 최근의 값과 혼합합니다. 
이 경우 unitSub.onNext (1)는 이전 값인 distanceSub.onNext (listOf (30, 19))와 결합됩니다. 
그리고 그 후에야 마일이 방출되는 마지막 거리 업데이트를보고 결과를 (30, 19, 12)로 정합니다.

on 액션 내부의 이전 이벤트 시퀀스 ( "30km, 마일, 단위 없음, 더 먼 거리 업데이트")는 
매우 혼란스럽고 가독성이 떨어집니다. 
하지만 다음과 같이 Spek의 중첩 그룹을 사용하여 동일한 사양을 작성할 수 있습니다.

{% highlight kotlin  %}

context("distance update of 30 km") {
    beforeEachTest {
        distanceSub.onNext(listOf(30, 19))
        unitSub.onNext(0)
    }

    it("should emit the value 30") {
        testSubscriber.assertValue(30)
    }

    context("unit update to miles") {
        beforeEachTest {
            unitSub.onNext(1)
        }

        it("should emit 19 as second value") {
            testSubscriber.assertValues(30, 19)
        }

        context("no more units") {
            beforeEachTest {
                unitSub.onCompleted()
            }

            it("should not complete") {
                testSubscriber.assertNotCompleted()
            }
        }
    }
}
{% endhighlight %}

  [source]: https://proandroiddev.com/spec-your-reactive-stream-using-publishsubjects-7b6a9951c319
  [github]: https://github.com/kimtaesu/kotlin.weekly/blob/master/src/test/kotlin/weekly/a60/2_Spek_your_reactive_stream_using_PublishSubjectsTest.kt
  [spek]: http://spekframework.org/docs/latest/#_writing_specifications
  [combinelatest]: http://reactivex.io/documentation/operators/combinelatest.html