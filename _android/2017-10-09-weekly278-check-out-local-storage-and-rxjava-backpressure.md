---
layout: post
title:  "Weekly 278 Want to optimize network usage? Check out local storage and RxJava backpressure"
date:   2017-10-14 0:10:00
description: Want to optimize network usage? Check out local storage and RxJava backpressure
tag:
 - weekly 278
 - android
toc: true
---

# Want to optimize network usage? Check out local storage and RxJava backpressure
[원문][source] 
[Switchman][switchman]

사용자는 빠르고 반응이 빠른 앱을 좋아합니다. 
API 호출에 시간이 얼마나 걸리는지 관심이 없으며 즉시 업데이트를 보고 싶어합니다. 
사용자에게 그 기대를 충족하면 사용자는 앱과 더 많이 상호 작용하기 시작하는 것이 쉽습니다. 

또한 불필요한 작업은 네트워크와 배터리 사용량이 증가합니다. 
따라서 네트워크 호출 횟수를 최소화하는 것이 모든 사람에게 가장 중요합니다.

이 Post에서는 **RxJava**를 사용하여 그러한 사례가 어떻게 해결할 수 있는지 알아보겠습니다.


## The Problem
Task는 사용자가 특정 목록에서 항목을 __추가, 제거__ 할 수 있는 기능을 개발하는 것입니다. 특정 목록은 백엔드 서버에 저장됩니다.  
많은 앱이 이러한 문제에 직면 해 있습니다. 
 * Gmail에서 전자 메일을 중요하게 표시
 * Spotify에서 즐겨 찾기에 노래를 추가
 * 이 Post를 추천하는 것 (😉)

<img class="col" src="https://cdn-images-1.medium.com/max/2000/1*xYpu_fK_B6LgK6q_XpZdGw.jpeg"/>



Problem 은 간단하게 들리지만 네트워크 연결 지연 및 네트워크 오류와 같은 사항이 고려 될 때 문제가 발생합니다.

구현은 다음 요구 사항을 충족해야합니다.
 * **사용자 인터페이스는 사용자의 동작에 즉시 반응합니다.** 사용자는 자신의 행동 결과를 즉시보고 싶어합니다. 변경 사항을 동기화 할 수없는 경우 사용자에게 알리고 이전 상태로 롤백해야합니다.
 * **여러 장치의 상호 작용이 지원됩니다.** 이것은 우리가 실시간으로 변화를 지원할 필요가 있다는 것을 의미하지는 않습니다. 그러나 우리는 수시로 전체 컬렉션을 가져와야합니다. 또한 Google의 백 엔드는 추가 및 제거를위한 API 끝점을 제공하며 더 나은 동기화를 지원하기 위해 사용 해야합니다.
 * **데이터의 무결성이 보장됩니다** 동기화 호출이 실패 할 때마다 Google의 앱이 오류에서 정상적으로 복구되어야합니다. 
 
## Formal definition

다음 인터페이스를 개발해야합니다 : 
{% highlight kotlin  %}
interface ItemRepository {
    Single<List<? extends Item<ItemId>>> getItemList();
    Single<Response> addItem(ItemId id);
    Single<Response> removeItem(ItemId id);
    Observable<Integer> getCounter();
    boolean hasItem(ItemId id);
} 
{% endhighlight %}


**addItem()** 및 **removeItem()** 메서드는 모든 인수로 임의의 순서로 호출 할 수 있습니다. 
불필요한 정보를 전달하지 않으려면 두 메소드 모두 ItemId 만 필요하고 **getItemList()**는 전체 항목을 반환합니다.

**getCounter()**는 ItemRepository가 변경 될 때마다 컬렉션의 항목을 방출하는 Observable을 반환한다는 점을 고려하십시오. 
즉, 예를 들어 백엔드와의 동기화가 아직 완료되지 않은 경우에도 새 항목이 추가 될 때마다 카운터가 증가합니다.

Backend API는 다음과 같습니다 : 
{% highlight kotlin  %}
public interface Api {
    Single<List<? extends Item<ItemId>>> getItemList();
    Single<ApiResponse> addItem(ItemId id);
    Single<ApiResponse> removeItem(ItemId id);
}
{% endhighlight %}

Item id로 아이템을 추가하거나 제거하고 전체 컬렉션을 가져올 수 있습니다.

## Solution

<img class="col" src="https://cdn-images-1.medium.com/max/2000/1*9lAeafTyXVoDx_-YwqQQ5g.jpeg"/>

기본 목록의 로컬 사본과 함께 로컬 저장소에는 두 개의 추가 목록이 있습니다. 
1. 하나는 진행중인 추가 용이고 
2. 다른 하나는 진행중인 제거 용 목록입니다. 

그들은 그것이 부정적인 경우에서 회복하는 것을 돕습니다. 자세한 내용을 보려면 [How to leverage Local Storage to build lightning-fast apps][localstorage]을 확인하십시오.

## Optimization Strategy

한 번에 백엔드에 대한 모든 요청을 시작해서는 안됩니다. 
예를 들어 동일한 항목을 동시에 추가 및 제거하면 결과를 예측할 수 없습니다. 
어떤 요청이 처음으로 끝날지는 알 수 없습니다. 
따라서 단일 항목에 대한 작업을 하나씩 대기열에 넣고 시작해야합니다.
여러 항목을 처리하는 요청은 문제없이 병렬로 처리 할 수 있습니다.

<img class="col" src="https://cdn-images-1.medium.com/max/2000/1*N1wPQLD6kwD9FmTcvGaYLQ.jpeg"/>

큐는 병렬로 실행됩니다. 각 큐는 특정 항목 추가 및 제거 요청으로 구성됩니다. 그 중 하나를 살펴 보겠습니다.

<img class="col" src="https://cdn-images-1.medium.com/max/2000/1*lVwJj6yje110MmUb793eFg.jpeg"/>

가장 중요한 요청은 **최신 요청과 마지막 요청**이 시작된 것입니다.

예를 들어 두 작업 모두 "add"라면 전체 큐를 건너 뛰고 아무 일도하지 않을 수 있습니다. 
서로 다르면 현재 요청이 완료되면 최신 요청을 시작하고 가운데에있는 모든 것을 건너 뛸 수 있습니다.

또한 ItemRepository는 새로운 데이터가 생기자마자 지연 없이 백엔드와 동기화를 시작해야합니다. 
버퍼링 및 일괄 처리를 사용하면이 솔루션을 더 향상시킬 수 있지만
이 문서의 범위를 벗어납니다. 여기서 언급하고 있으므로 솔루션의 동작에 놀라지 않을 것입니다.

## Here comes RxJava

RxJava는 데이터 스트림을 조작하는 강력한 메커니즘을 제공합니다. 우리의 이점에 그것들을 사용합시다.
1. 사용자 의도를 나타내는 이벤트 (추가 및 제거)를 만듭니다. 이제 Subject 또는 Relay를 사용하여 해당 이벤트의 입력 스트림을 만들 수 있습니다.
2. 이 스트림을 동일한 itemId로 그룹화 된 이벤트의 여러 하위 스트림으로 분할하여 대기열을 구현합니다. 이렇게하려면 GroupedObservable의 Observable을 반환하는 groupBy 연산자를 사용할 수 있습니다. 각 GroupedObservable은 최적화 전략의 큐입니다.

<img class="col" src="https://cdn-images-1.medium.com/max/1600/1*rMWYkMeQAYqCgzJKv54EAg.png"/>

## Optimization

마지막 단계는 최적화입니다. 
시작하기 전에 최적화 전략을 한 번 더 확인하십시오. 
가장 최근의 이벤트를 실제로 네트워크 호출을 유발 한 마지막 이벤트와 함께 유지해야 합니다.  

아쉽지만, 각 백엔드 요청의 소요 시간이나 새 이벤트 발생 빈도는 알 수 없습니다. 
이것이 의미는 하는 것은 필요한 동작을 구현하기 위해 `window`, `buffer`, `throttle`또는 `debounce`을 사용할 수 없음을 의미합니다. 이를 사용하려면 미리 타이밍을 알아야합니다.

따라서 우리는 `onBackpressureLatest(Subscriber)` 생성해야합니다. `Subscriber.request()` 메서드가 호출 될 때만 방출되는 마지막 이벤트를 제외한 모든 이벤트를 삭제합니다.

<img class="col" src="https://cdn-images-1.medium.com/max/2000/1*gEFqpGv6ZjLaeyuf5fZHJw.png"/>
Sample은 [여기서][backpressure-sample] 확인할 수 있습니다. 

이제 `Subscriber`에서 백엔드 호출을 수행하여 `request()`를 호출 할 시점을 알아 보겠습니다. 
이벤트 간의 비교도 처리 할 수 있습니다. 동일하면, 마지막 것을 건너 뜁니다. 그렇지 않으면 백엔드에 대한 호출을 시작합니다. 결과는 출력 스트림에 게시됩니다.

`ItemRepository.addItem()` 및 `ItemRepository.removeItem()`은 `Single`을 반환하므로 API의 각 호출자에게 적절한 응답을 제공하기 위해이 스트림을 필터링해야합니다. 

## The third state of the request
요청을 건너 뛰거나 작성하는 것과는 별도로 ItemRepository의 소비자는 응답을 기대합니다. 
즉, 일부 요청을 건너 뛰더라도 `addItem()` 및 `removeItem()`에 대한 호출 만큼 많은 응답을 반환해야합니다. 

이것을 처리 할 수있는 두 가지 방법이 있습니다.

첫째,  consumer의 모든 최적화를 숨기고 모든 요청이 실행 된 것처럼 행동 할 수 있습니다. 최적화를 수행 한 후에는 consumer의 코드를 변경할 필요가 없으므로 캡슐화에 좋습니다.

그러나 경우에 따라서는 더 많은 복잡성을 의미합니다. 
예를 들어 각 응답마다 snackbar를 표시하고 최종 사용자가 여러 번 클릭하여 동일한 항목을 추가 및 제거하면 알림으로 스팸 메일을 보냅니다. 
이 문제를 해결하기 위해 우리는 이러한 경우를 다루기 위한 몇 가지 논리를 구현해야합니다.

아니면 우리는 두 번째 접근법에 갈 수 있습니다. 
우리는 API의 consumer에게 정직 할 수 있으며 
응답의 세 번째 "skipped"상태를 추가 할 수 있습니다.
경험에 비추어 볼 때, 간단하게 만들 수 있다고 말할 수 있습니다. 위에 예에서 skipped 모든 요청에 대해서는 알림을 표시하지 않습니다.

## Coding time

{% highlight kotlin  %}
inputStream.groupBy(Event::getItemId)
    .subscribe(substream -> 
        substream.onBackpressureLatest()
               .subscribe(new Subscriber<Event>() {
                   public void onNext(Event event) {
                      if (shouldSkipEvent(event)) {
                          outputStream.onNext(Response.createSkippedResponse());
                          request(1);
                      } else {
                          saveLastLaunchedEvent(event);
                          launchNetworkCall(event).doOnSuccess(outputStream::onNext)
                                        .subscribe(ignored -> request(1));
                      }
                  }
              }
          }));
{% endhighlight %}

가장 중요한 부분을 살펴 보겠습니다.
* 1 행 : ID별로 inputStream을 그룹화하고 하위 스트림을 얻었습니다.
* 3 행 : 우리는 Backpressure를 적용했다. 마지막 이벤트를 제외한 모든 항목이이 단계에서 삭제됩니다.
* 6 행 : 이벤트에 따라 건너 뛰고 건너 뛴 응답을 반환하거나 네트워크 호출을 시작하여 나중에 비교할 수 있도록 저장합니다.

위의 코드는 컴파일되지 않습니다. [repo][switchman]에서 재생할 실제 코드와 작은 Android 앱을 찾을 수 있습니다.

## Conclusion

컬렉션에 항목을 제거하고 항목을 추가하는 것과 같은 간단한 작업조차도 최적화, 동기화 및 탁월한 사용자 경험에 있어서는 어려움이됩니다. 
RxJava와 같은 훌륭한 도구는 코드를 간결하고 표현력있게 유지하는 데 도움이 될 수 있습니다.



  [source]: https://medium.freecodecamp.org/want-to-optimize-network-usage-check-out-local-storage-and-rxjava-backpressure-8b91b1db298a
  [switchman]: https://github.com/NikitaKozlov/Switchman
  [backpressure-sample]: https://github.com/kimtaesu/android-weekly/tree/master/Weekly278-Check-out-local-storage-and-RxJava-backpressure
  [localstorage]: https://medium.freecodecamp.org/how-leverage-local-storage-to-build-lightning-fast-apps-4e8218134e0c