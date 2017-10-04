---
layout: post
title:  "Weekly 277 Android Architecture Components Testing your ViewModel LiveData"
date:   2017-09-25 0:10:00
description: Android Architecture Components Testing your ViewModel LiveData
tag:
 - weekly 277
 - android
toc: true
---

# Android Architecture Components Testing your ViewModel LiveData"
## [Article][source] 
## [Github][github]

지난 주에 Buffer Android Boilerplate의 포크를 공개 저장소에 푸시했습니다.
이 포크의 차이점은 새로운 아키텍처 구성 요소 (Presentation Layer의 ViewModels 및 Cache Layer에서 Room을 사용한다는 점입니다. 

LiveData가 Update되는 지 확인할 수 있는 간단한 방법을 간단히 살펴 보겠습니다. 

{% highlight java  %}
open class BrowseBufferoosViewModel @Inject internal constructor(
        private val getBufferoos: GetBufferoos,
        private val bufferooMapper: BufferooMapper) : ViewModel() {

    private val bufferoosLiveData: MutableLiveData<Resource<List<BufferooView>>> =
            MutableLiveData()

    override fun onCleared() {
        getBufferoos.dispose()
        super.onCleared()
    }

    fun getBufferoos(): LiveData<Resource<List<BufferooView>>> {
        return bufferoosLiveData
    }

    fun fetchBufferoos() {
        bufferoosLiveData.postValue(Resource(ResourceState.LOADING, null, null))
        return getBufferoos.execute(BufferooSubscriber())
    }

    inner class BufferooSubscriber: DisposableSubscriber<List<Bufferoo>>() {

        override fun onComplete() { }

        override fun onNext(t: List<Bufferoo>) {
            bufferoosLiveData.postValue(Resource(ResourceState.SUCCESS,
                    t.map { bufferooMapper.mapToView(it) }, null))
        }

        override fun onError(exception: Throwable) {
            bufferoosLiveData.postValue(Resource(ResourceState.ERROR, null, exception.message))
        }

    }

}
{% endhighlight %}  

ViewModel 인스턴스에 **GetBufferoos**라는 Use Case 클래스가 있음을 볼 수 있습니다. 
**ViewModel**을 사용하는 일부 프로젝트는 Repository와 직접 통신합니다.

ViewModel이 뷰의 올바른 이벤트를 triggering 하고 있음을 확인할 수 있는 두 가지 방법이 있습니다.


## Verifying Observer onChanged() events

첫 번째 방법은 mockito를 사용하여 
ViewModel 내에서 postValue () 메소드를 트리거해야 할 때 
onObserver onChanged ()가 호출되는지 확인하는 것입니다. 
이 예제에서는 위 코드 **fetchBufferoos()** 메서드를 사용합니다.

{% highlight java  %}
@Test
fun fetchBufferoosTriggersLoadingState() {
    bufferoosViewModel.getBufferoos().observeForever(observer)
    bufferoosViewModel.fetchBufferoos()

    verify(observer).onChanged(Resource(ResourceState.LOADING))
}
{% endhighlight %}

이 테스트는 fetchBufferoos() 메서드를 호출 할 때 
LiveData가 ResourceState 값이 LOADING 인 change 이벤트로 업데이트되는지 확인하는 간단한 작업을 수행합니다. 

{% highlight java  %}
open class BrowseBufferoosViewModel @Inject internal constructor(
...
fun fetchBufferoos() {
        bufferoosLiveData.postValue(Resource(ResourceState.LOADING, null, null))
        return getBufferoos.execute(BufferooSubscriber())
    }
...

{% endhighlight %}

## Asserting LiveData values

우리의 테스트에서 특정 데이터가 설정되었다고 주장하고 싶다면 위와 비슷한 방법으로 할 수 있습니다. 
솔직히 말해서, 이러한 접근 방식은 꽤 유사하므로 개인적인 선호도에 따라 결정됩니다. 
우리의 테스트 방법은 약간만 변경됩니다.

{% highlight java  %}
@Test
fun fetchBufferoosTriggersLoadingState() {
    bufferoosViewModel.getBufferoos().observeForever(observer)
    bufferoosViewModel.fetchBufferoos()

    assert(bufferoosViewModel.getBufferoos().value.status ==     
        ResourceState.LOADING)
}
{% endhighlight %}

여기서 수행하는 작업은 ViewModel에서 livedata 인스턴스를 가져 와서 값을 검색하고 원하는 값과 일치하는지 확인하는 것입니다.

이 두 접근법의 차이점의 주요 열쇠는 테스트 방법의 초점입니다.

예를 들어 fetchBufferoosReturnsBufferoos () 테스트 메소드가 있고 데이터가 다음과 같이 변경되었다고 가정했습니다.
{% highlight java  %}
verify(observer).onChanged(
        Resource(ResourceState.SUCCESS, bufferoos))
{% endhighlight %}

대부분의 경우이 확인을 할 수 있지만 일부 개발자는 이 테스트를 두 가지 별도의 메서드로 나누는 것을 선호 할 수 있습니다.

fetchBufferoosReturnsSuccessState() - 결과를 다음과 같이 표시 
{% highlight java  %}
assert(bufferoosViewModel.getBufferoos().value.status ==     
        ResourceState.SUCCESS)
{% endhighlight %}

fetchBufferoosReturnsBufferoos () - 결과를 다음과 같이 표시 
{% highlight java  %}
assert(bufferoosViewModel.getBufferoos().value.data == bufferoos)
{% endhighlight %}

위의 메서드는 ViewModel 메서드에 대한 단위 테스트를 작성하는 방법에 대한 간단한 소개입니다.


> 저의 성향적으로 보면 Test Case를 별도의 메서드로 나누는 것을 선호합니다. 그래야만 Test Case가 모호해지지 않고 명확해지기 때문입니다. 
그러나 가장 중요한 것은 **"무엇을 테스트하고자 하는가"** 입니다.
  
  [source]: https://medium.com/exploring-android/android-architecture-components-testing-your-viewmodel-livedata-70177af89c6e
  [github]: https://github.com/bufferapp/clean-architecture-components-boilerplate
  
  