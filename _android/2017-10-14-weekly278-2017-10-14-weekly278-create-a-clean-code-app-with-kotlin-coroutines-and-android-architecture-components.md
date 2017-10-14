---
layout: post
title:  "Weekly 278 Create a Clean-Code App with Kotlin Coroutines and Android Architecture Components"
date:   2017-10-09 0:10:00
description: Create a Clean-Code App with Kotlin Coroutines and Android Architecture Components
tag:
 - weekly 278
 - android
toc: true
---

# Create a Clean-Code App with Kotlin Coroutines and Android Architecture Components
[원문][source] 

Android 개발은 빠르게 발전하고 있습니다. 
많은 개발자와 회사가 일반적인 문제를 해결하고 앱 구성 방식을 완전히 바꿀 수있는 훌륭한 도구 나 라이브러리를 만들려고 노력하고 있습니다.

새로운 프로그래밍 스타일의 혜택을 받기 위해 앱을 다시 작성할 시간을 찾기는 어렵습니다. 
그러나 실제로 새로운 프로젝트를 시작한다면 어떨까요? 
어떤 획기적인 아이디어를 채용 할 것인가? 
어떤 솔루션이 충분히 안정적입니까? 
RxJava를 광범위하게 사용하고 반응 우선 사고 방식으로 앱을 구성해야합니까?

Rx는 highly composable하며 큰 잠재력을 가지고 있지만 일반적인 객체 지향 프로그래밍 스타일과는 너무나 다르기 때문에 RxJava 경험이 없는 개발자는 이해하기 어려울 것입니다.
  
새 프로젝트를 시작하기 전에 더 많은 질문을해야합니다.
* Java 대신 Kotlin을 사용해야합니까?  (실제로 여기서 대답은 간단합니다 : 예)
* experimental Kotlin Coroutines을 사용해야할까요? (다시, 완전히 새로운 프로그래밍 스타일을 촉진)
* Google의 새로운 실험 라이브러리를 사용해야합니까? Android 아키텍처 구성 요소 (MVVM, Room)?

결정을 내리기 위해서는 Sample에서 모든 것을 먼저 시도해야합니다. 
이것이 바로 제가 한 일로, 프로세스에서 유용한 통찰력을 얻었습니다.

## About [The App][github]

이 aim of the experiment은 사용자가 선택한 도시의 날씨 데이터를 다운로드 한 다음 앱의 차트를 그래픽 차트 (그리고 멋진 애니메이션)로 표시하는 앱을 만드는 것이 었습니다. 
간단하지만 아직 Android 프로젝트의 일반적인 기능이 대부분 포함되어 있습니다.

**coroutines**과 **architecture components**가 실제로 잘 작동하며 문제를 효과적으로 분리하여 깨끗한 앱 아키텍처를 제공합니다. 
coroutines은 자연스럽고 간결한 방식으로 아이디어를 표현합니다. 
Suspendable 함수는 비록 비동기 호출을 중간에해야 할 필요가 있더라도 마음에 든 정확한 논리를 한 줄씩 코딩하려는 경우 유용합니다.

이 예제 애플리케이션에서는 **RxJava**를 사용할 필요를 완전히 제거했습니다. 
suspendable 함수는 일부 RxJava 연산자 체인보다 읽기 쉽고 이해하기 쉽습니다.
이러한 체인은 너무 빠르게 작동 할 수 있습니다. ;-)

> RxJava가 모든 Use Case에서 coroutines으로 대체 될 수 있다고 나는 생각하지 않는다. 
Observables는 일시적인 기능에 일대일로 매핑 할 수없는 다른 종류의 표현력을 우리에게 제공합니다. 
특히 한 번 생성 된 관찰 가능 연쇄 체인을 통해 여러 이벤트가 전달 될 수 있으며 suspendable은 호출 당 한 번만 재개됩니다.

다음은 사용자의 편의를 위해 텍스트 참조 번호가 추가 된 일반적인 아키텍처 다이어그램입니다.

나는 특히 **green elements**에 중점을 둘 것이다  suspendable functions 와 actors (actors는 정말 유용한 종류의 coroutines builder 입니다.)

<img class="col" src="https://cdn-images-1.medium.com/max/1600/1*DL--eDRDLPPCDR1nsAmILg.png"/>

## 01 Weather Service

이 Service는 Open Weather Map REST API에서 특정 도시의 일기 예보를 다운로드합니다.

<img class="col" src="https://cdn-images-1.medium.com/max/1600/1*QGvoMVNbR_nHjmn0WCCFsw.png"/>

여기서 주목해야 할 중요한 점은 Retrofit에서 생성 된 함수의 Return type이 **Call** 이라는 것 입니다.
실제로 Call.enqueue 를 사용하여 Open Weather Map을 호출합니다.

## 02 Utils
이것은 우리가 coroutines 세계에 들어가는 곳입니다 : 우리는 Call 객체를 감싸는 일시 중단 가능한 함수를 만들고 싶습니다.

> 적어도 당신은 coroutines의 기초를 알고 있다고 가정합니다. [courutine-guide][Coroutines Guide]의 첫 번째 장 (Roman Elizarov 작성)을 읽으십시오.

suspend fun Call<T>.await() extension function은 Call.enqueue (...)를 호출하고 일시 중단 한 후 Response가 돌아올 때 까지 기다립니다. 
<img class="col" src="https://cdn-images-1.medium.com/max/1600/1*T6QT9tRQbqOS9pKJfyh0og.png"/> 

## 03 Repository
리포지토리 개체는 앱에 표시되는 데이터 (차트)의 소스입니다.
<img class="col" src="https://cdn-images-1.medium.com/max/1600/1*rie-ith-AXP8-ajuBiNdzw.png"/>

여기에는 suspend fun Call<T>.await() 를 적용하여 만든 weather service 의 함수들은 private suspendable functions 이 있습니다. 
이 방법으로 모두 Call 대신 Forecast 데이터를 사용할 준비가 되었습니다.

> 중요 사항 : suspendable functions 만이 다른 suspendable functions을 호출 할 수 있습니다.

MVVM 내용은 생략하도록 하겠습니다. 

## 정리 (저의 생각) 

Architecture와 Coroutines의 두 가지로 정리하였습니다. 
 
### Architecture
본문 서론에서 새로운 프로그래밍 스타일을 어떻게 적용하는 것이 좋은지에 대한 좋은 견해가 있었습니다. 
Post에서는 Sample앱을 만들어보고 시도해보고 통찰력을 얻는 것을 원하였지만
현재 `Kotlin coroutines`, `Android Architecture Components` 는 실험적인 단계이거나 Alpha version 입니다. 
득과 실 관계를 따져본다면
우선, 득이 되는 케이스는 새로운 프로그래밍 스타일로 간결하고 명확한 코드를 얻을 순 있습니다.  
실이 되는 케이스는 안정적이지 않으며 update시 마다 migration이 되어야 합니다. 

저의 견해는 **조직의 문화가 가장 우선적으로 고려**되어야 하며 
두번째는 실험적인 단계이거나 Alpha version을 상용화 제품에 적용하는 것보다는 안정적으로 비즈니스 로직을 진행해 나가는 것이 좋을 것 같습니다. 
비즈니스 로직 단위를 작은 단위로 모듈화 시킨 다면, 실험적으로 특정 모듈만 적용도 가능 할 것 같습니다. 

### Coroutines 
파이썬에서 Coroutines을 사용했던 경험은 대용량 데이터들을 효율적인 메모리로 fetch 할 경우에 사용한다고 생각했었습니다.
그러나 Post에서는 네트워크 통신에서의 Call 함수를 주제로 다룬 것이 저에게는 흥미롭게 느꼈으며, 많은 것을 배웠습니다.  

  [source]: http://blog.evernote.com/tech/2017/10/06/announcing-android-job-library-1-2-0/
  [github]: https://github.com/elpassion/crweather
  [courutine-guide]: https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md
  
  