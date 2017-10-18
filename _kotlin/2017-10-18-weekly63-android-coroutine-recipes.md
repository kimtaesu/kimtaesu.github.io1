---
layout: post
title:  "Weekly 63 Android Coroutine Recipes"
date:   2017-10-18 0:10:00
description: Android Coroutine Recipes
tag:
 - weekly 63
 - kotlin
toc: true
---

# Android Coroutine Recipes
[원문][source]

[Source Code][github]는 여기에서 확인 할 수 있습니다.


## How to launch a coroutine
`kotlinx.coroutines` 라이브러리에서 `launch` or `async` 함수를 사용하여 새로운 동시 루틴을 시작할 수 있습니다.

개념적으로 `async` 는 `launch`와 같습니다. 


차이점은 `launch`는 `Job`을 반환하고 
아무런 결과 값도 전달하지 않는 반면
 
`async`는 `Defered`를 반환합니다. 
`Defered` 은   light-weight non-blocking future로 결과를 전달하며, 
`.await()`을 사용하면 최종 결과를 얻을 수 있습니다. `Deferred`는 또한 `Job`이기 때문에 필요할 경우 취소 할 수 있습니다. 
 
`launch` 블럭에서 예외로 종료되면 스레드에서 캐치되지 않는 예외처럼 처리되어 Android 응용 프로그램이 충돌합니다.
 
`async` 코드 내의 포착되지 않은 예외는 결과 Deferred 내부에 저장되며 다른 곳에서는 전달되지 않고 처리되지 않으면 자동으로 삭제됩니다. 

## Coroutine context
Android에서는 보통 두 가지 상황을 사용합니다.
* `uiContext` : 부모 코루틴의 경우 Android 기본 UI 스레드로 실행을 전달합니다.
* `bgContext` : 백그라운드 스레드 (자식 코루틴의 경우)에서 실행을 전달합니다.

{% highlight kotlin  %}
// dispatches execution onto the Android main UI thread
private val uiContext: CoroutineContext = UI
 
// represents a common pool of shared threads as the coroutine dispatcher
private val bgContext: CoroutineContext = CommonPool
{% endhighlight %}

다음 예에서는 `bgContext`에 `CommonPool`을 사용하여 `Runtime.getRuntime.availableProcessors() - 1` 값과 병렬로 실행되는 스레드의 수를 제한합니다
따라서 coroutine 작업이 예약되었지만 모든 코어가 점유되면 대기열에 들어갑니다.

`newFixedThreadPoolContext` 또는 캐시 된 스레드 풀 구현을 고려할 수 있습니다.

## launch + async (execute task)

Parent coroutine은 `UI Context`와 함께 `launch` 함수를 통해 시작됩니다.

Child coroutine은 `CommonPool Context`와 함께 `async` 함수를 통해 실행됩니다.

> **Note** Parent coroutine은 항상 모든 하위 항목의 완료를 기다립니다.

> **Note** 검사되지 않은 예외가 발생하면 응용 프로그램이 중단됩니다.

{% highlight kotlin  %}
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    val task = async(bgContext) { dataProvider.loadData("Task") }
    val result = task.await() // non ui thread, suspend until finished
 
    view.showData(result) // ui thread
}
{% endhighlight %}

## launch + async + async (execute two tasks sequentially)

Parent coroutine은 `UI Context`와 함께 `launch` 함수를 통해 시작됩니다.

Child coroutine은 `CommonPool Context`와 함께 `async` 함수를 통해 실행됩니다.

> **Note** task1과 task2가 순차적으로 실행됩니다.

> **Note** 검사되지 않은 예외가 발생하면 응용 프로그램이 중단됩니다.


{% highlight kotlin  %}
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    // non ui thread, suspend until task is finished
    val result1 = async(bgContext) { dataProvider.loadData("Task 1") }.await()
 
    // non ui thread, suspend until task is finished
    val result2 = async(bgContext) { dataProvider.loadData("Task 2") }.await()
 
    val result = "$result1 $result2" // ui thread
 
    view.showData(result) // ui thread
}
{% endhighlight %}

launch + async + async (execute two tasks parallel)

Parent coroutine은 `UI Context`와 함께 `launch` 함수를 통해 시작됩니다.

Child coroutine은 `CommonPool Context`와 함께 `async` 함수를 통해 실행됩니다.

> **Note** task1과 task2가 병렬적으로 실행됩니다.

> **Note** 검사되지 않은 예외가 발생하면 응용 프로그램이 중단됩니다.

{% highlight kotlin  %}
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    val task1 = async(bgContext) { dataProvider.loadData("Task 1") }
    val task2 = async(bgContext) { dataProvider.loadData("Task 2") }
 
    val result = "${task1.await()} ${task2.await()}" // non ui thread, suspend until finished
 
    view.showData(result) // ui thread
}
{% endhighlight %}

## How to launch a coroutine with a timeout

If you want to set a timeout for a coroutine job, wrap the suspended function with the withTimeoutOrNull function which will return null in case of timeout.

coroutine job에 대한 timeout을 설정하려면 
suspended function 함수를 withTimeoutOrNull 함수로 감싸야 합니다.
이 함수는 timeout이 발생할 경우 null을 반환합니다.

{% highlight kotlin  %}
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    val task = async(bgContext) { dataProvider.loadData("Task") }
 
    // non ui thread, suspend until the task is finished or return null in 2 sec
    val result = withTimeoutOrNull(2, TimeUnit.SECONDS) { task.await() }
 
    view.showData(result) // ui thread
}
{% endhighlight %}

## How to cancel a coroutine

`loadData` 함수는 취소 될 수 있는 `Job` 객체를 반환합니다. Parent coroutine이 취소되면 모든 자식도 재귀적으로 취소됩니다.

`dataProvider.loadData`가 아직 진행 중일 때 `stopPresenting` 함수가 호출 된 경우 `view.showData` 함수는 호출되지 않습니다.

{% highlight kotlin  %}
var job: Job? = null
 
fun startPresenting() {
    job = loadData()
}
 
fun stopPresenting() {
    job?.cancel()
}
 
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    val task = async(bgContext) { dataProvider.loadData("Task") }
    val result = task.await() // non ui thread, suspend until finished
 
    view.showData(result) // ui thread
}
{% endhighlight %}


## How to handle exceptions with coroutines

Parent coroutine은 `UI Context`와 함께 `launch` 함수를 통해 시작됩니다.

Child coroutine은 `CommonPool Context`와 함께 `async` 함수를 통해 실행됩니다.

> **Note** try-catch 블록을 사용하여 예외를 catch하고 처리 할 수 있습니다.

{% highlight kotlin  %}
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    try {
        val task = async(bgContext) { dataProvider.loadData("Task") }
        val result = task.await() // non ui thread, suspend until task is finished
 
        view.showData(result) // ui thread
    } catch (e: RuntimeException) {
        e.printStackTrace()
    }
}
{% endhighlight %}

Presenter Class 에서 try-catch를 피하려면 `dataProvider.loadData` 함수에서 예외를 처리하고 일반 `Result` Class를 반환하는 것이 좋습니다.

{% highlight kotlin  %}
data class Result<out T>(val success: T? = null, val error: Throwable? = null)
 
private fun loadData() = launch(uiContext) {
    view.showLoading() // ui thread
 
    val task = async(bgContext) { dataProvider.loadData("Task") }
    val result: Result<String> = task.await() // non ui thread, suspend until the task is finished
 
    if (result.success != null) {
        view.showData(result.success) // ui thread
    } else if (result.error != null) {
        result.error.printStackTrace()
    }
}
{% endhighlight %}


## async + async

Parent coroutine은 `UI Context`와 함께 `launch` 함수를 통해 시작됩니다.

Child coroutine은 `CommonPool Context`와 함께 `async` 함수를 통해 실행됩니다.

> **Note** Exception을 무시하려면 async 기능으로 Parent coroutine을 실행하십시오.

{% highlight kotlin  %}
private fun loadData() = async(uiContext) {
    view.showLoading() // ui thread
 
    val task = async(bgContext) { dataProvider.loadData("Task") }
    val result = task.await() // non ui thread, suspend until the task is finished
 
    view.showData(result) // ui thread
}
{% endhighlight %}

이 경우 예외는 Job 개체에 저장됩니다. 이를 검색하기 위해 invokeOnCompletion 함수를 사용할 수 있습니다.

{% highlight kotlin  %}
var job: Job? = null
 
fun startPresenting() {
    job = loadData()
    job?.invokeOnCompletion { it: Throwable? ->
        it?.printStackTrace() // (1)
        // or
        job?.getCompletionException()?.printStackTrace() // (2)
 
 
        // difference between (1) and (2) is that (1) will NOT contain CancellationException
        // in case if job was cancelled
    }
}
{% endhighlight %}

## launch + coroutine exception handler

Parent coroutine은 `UI Context`와 함께 `launch` 함수를 통해 시작됩니다.

Child coroutine은 `CommonPool Context`와 함께 `async` 함수를 통해 실행됩니다.

> **Note** Parent coroutine 컨텍스트에 CoroutineExceptionHandler를 추가하여 Exception을 catch하고 처리 할 수 있습니다.

{% highlight kotlin  %}
val exceptionHandler: CoroutineContext = CoroutineExceptionHandler { _, throwable -> throwable.printStackTrace() }
 
private fun loadData() = launch(uiContext + exceptionHandler) {
    view.showLoading() // ui thread
 
    val task = async(bgContext) { dataProvider.loadData("Task") }
    val result = task.await() // non ui thread, suspend until the task is finished
 
    view.showData(result) // ui thread
}
{% endhighlight %}


## How to test coroutines

MainPresenter Class에 대한 Unit Test를 작성하려면 UI 및 Background 실행에 대한 Coroutine 컨텍스트를 지정할 수 있어야합니다.

가장 쉬운 방법은 MainPresenter 생성자에 두 개의 매개 변수를 추가하는 것입니다 
1. 기본값 UI 인 `uiContext`
2. 기본값 인 CommonPool이 있는 `ioContext`

{% highlight kotlin  %}
class MainPresenter(private val view: MainView,
                    private val dataProvider: DataProviderAPI
                    private val uiContext: CoroutineContext = UI,
                    private val ioContext: CoroutineContext = CommonPool) {
 
    private fun loadData() = launch(uiContext) { // use the provided uiContext (UI)
        view.showLoading()
 
        // use the provided ioContext (CommonPool)
        val task = async(bgContext) { dataProvider.loadData("Task") }
        val result = task.await()
 
        view.showData(result)
    } 
}
{% endhighlight %}

이제는 현재 스레드에서 코드를 실행하는 EmptyCoroutineContext를 제공하여 MainPresenter 클래스를 쉽게 테스트 할 수 있습니다.


{% highlight kotlin  %}
@Test
fun test() {
    val dataProvider = Mockito.mock(DataProviderAPI::class.java)
    val mockView = Mockito.mock(MainView::class.java)
 
    val presenter = MainPresenter(mockView, dataProvider, EmptyCoroutineContext, EmptyCoroutineContext)
    presenter.startPresenting()
    ...
}
{% endhighlight %}



  [source]: https://proandroiddev.com/android-coroutine-recipes-33467a4302e9
  [github]: https://github.com/dmytrodanylyk/coroutine-recipes