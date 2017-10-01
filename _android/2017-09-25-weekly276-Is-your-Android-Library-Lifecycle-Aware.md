---
layout: post
title:  "Weekly 276 Is your Android Library, Lifecycle-Aware?"
date:   2017-09-25 0:10:00
description: Is your Android Library, Lifecycle-Aware?
tag:
 - weekly 276
 - android
toc: true
---

# Is your Android Library, Lifecycle-Aware?
## [Article][source] 
## [Github][github]
생각지도 못했지만 대부분의 경우 코드베이스에서 메모리 누출의 주요 원인은 
앱의 액티비티 / 프래그먼트의 라이프 사이클과 동기화되지 않는 잘못된 상태에 액세스하기 때문입니다. 
이것이 의미하는 것은 Android 개발자로서 코드 상태가 Activity / Fragment의 라이프 사이클과 동기화되어 있는지 확인하여 많은 조건를 체크 해야 했습니다.

이 작업은 많이 지저분 해지기 시작합니다. 
안드로이드 라이브러리 개발자로서, Activity / Fragment onDestroy (), onResume (), onCreate ()의 라이프 사이클 중 특정 시점에서 다시 초기화 할 수 있도록 콜백 or 콜 정리 메소드 또는 특정 함수 등록 취소를 준수해야하는 다른 앱 개발자가 필요합니다.

우리는 개발자가 문서화를 거치지 않는 경향이 있다는 것을 알고 있습니다. (제품을 신속하게 만들고  Deadline을 지키기 위해선😅). 
이것이 의미하는 바는 때로는 정리 부분이 코드에서 올바른 위치에 놓이기 위해 제대로 인식되지 않거나 이해되지 않는 경우입니다. 
예를 들어 활동이 중지 된 후 콜백을받는 경우 메모리 누수로 인해 앱이 중단 될 가능성이 큽니다. 
앱 개발자가 안드로이드 라이브러리에서 cleanup () 메소드를 호출했지만 가능할 것으로 예상되는 경우, 
콜백을 등록 취소하는 것이 Activity가 파괴되기 바로 전에 필요합니다. 그들이 그것을 준수 할 수 있는지 확인할 수 없습니다.

## 해결책은 무엇입니까?
해결책은 안드로이드 라이브러리 Lifecycle-Aware를 만드는 것입니다. 
안드로이드 라이브러리가 Activity / Fragment의 라이프 사이클 상태를 인식하게되면, 
안드로이드 라이브러리 개발자는 콜백이 등록되지 않았는지, 콜백 메소드 호출 또는 재 초기화가 Activity / Fragment.
두 사람 모두를위한 윈 - 윈처럼 들립니다. 😎
동시에 앱 개발자가 안드로이드 라이브러리의 상태를 처리하기 위해 앱에 더 많은 코드와 수표를 사용하는 것에 따르는 것에 덜 의존하게됩니다. 

## 해결책과 관련하여 저의 의견으로 해석해 보겠습니다. 
View Code, 즉 Activity, Fragment Source 안에 Activity, Fragment Lifecycle에 의한 객체 초기화, 콜백 등록 작업 등을 
다른 Source로 분리함으로써 View Code와의 의존성을 낮췄다는 이야기로 해석하였습니다.
그러므로 View Code 가 아닌 객체의 초기화, 콜백 등록 등 작업에 집중할 수 있도록 하는 것이라고 해석하였습니다.

> 업데이트 : Arch 구성 요소 v1.0.0-alpha9-1의 출시로,
the core lifecycle artifacts (runtime, common)이 **안정 버전 v1.0.0**에 도달했습니다.
그러나 The lifecycle compiler and extensions 은 여전히 v1.0.0-alpha9-1 버전입니다.

# 이제 소스를 살펴보도록 하겠습니다. 

> ### preparation
> IDE : Android Studio 3.0 

awesomelib module을 app 모듈에 대한 종속성으로 추가하여 
app 모듈이 awesomelib 라이브러리 모듈에 포함 된 공용 클래스에 액세스 할 수 있도록합니다. 

[app] build.gradle 
{% highlight gradle  %}
implementation project(':awesomelib')
{% endhighlight %}

awesomelib 모듈에서 사용하기 위한 Lifecycle Arch Components의 의존성을 추가 합니다.
 
[awesomelib] build.gradle 
{% highlight gradle  %}
implementation "android.arch.lifecycle:runtime:1.0.0"
annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha9-1"
{% endhighlight %}


AwesomeLibMain.java 를 유심히 살펴보십시오. 
 * implements LifecycleObserver
 * @OnLifecycleEvent(Lifecycle.Event)

첫 번째로 LifecycleObserver를 implements 해야합니다. 

그리고 두 번째로 Trigger가 될 Lifecycle의 Event를 Annotation 기반으로 선언해야합니다. 

{% highlight java  %}
import android.arch.lifecycle.Lifecycle;
import android.arch.lifecycle.LifecycleObserver;
import android.arch.lifecycle.OnLifecycleEvent;

public class AwesomeLibMain implements LifecycleObserver {
  static final AwesomeLibMain ourInstance = new AwesomeLibMain();

  public static AwesomeLibMain getInstance() {
    return ourInstance;
  }

  private AwesomeLibMain() {
  }
    
  @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
  public void init() {
    System.out.println(
        "Called From AwesomeLibMain Class, called onCreate() of Activity >>>>>> init()");
  }

  @OnLifecycleEvent(Lifecycle.Event.ON_START)
  public void LibOnStart() {
    System.out.println(
        "Called From AwesomeLibMain Class, called onStart() of Activity >>>>>> LibOnStart()");
  }

  @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
  public void LibOnStop() {
    System.out.println(
        "Called From AwesomeLibMain Class, called onStop() of Activity >>>>>> LibOnStop()");
  }

  @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
  public void LibOnResume() {
    System.out.println(
        "Called From AwesomeLibMain Class, called onResume() of Activity >>>>>> LibOnResume()");
  }

  @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
  public void LibOnPause() {
    System.out.println(
        "Called From AwesomeLibMain Class, called onPause() of Activity >>>>>> LibOnPause()");
  }

  @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
  public void cleanup() {
    System.out.println(
        "Called From AwesomeLibMain Class, called onDestroy() of Activity >>>>>> cleanup()");
  }
}
{% endhighlight %}


이제 사용하는 MainActivity.java를 살펴보도록 하겠습니다. 
 * getLifecycle().addObserver
 * getLifecycle().removeObserver

getLifecycle() addObserver / removeObserver 함수를 통하여  AwesomeLibMain의 Lifecycle을 등록 / 해제 하도록 합니다. 


{% highlight java  %}
public class MainActivity extends AppCompatActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // Add Lifecycle Observer
    getLifecycle().addObserver(AwesomeLibMain.getInstance());
  }

  @Override
  protected void onDestroy() {
    super.onDestroy();

    // Remove Lifecycle Observer
    getLifecycle().removeObserver(AwesomeLibMain.getInstance());
  }
}
{% endhighlight %}

> **Note**:
우리는 "com.android.support:appcompat-v7:$26.1.0" 의 library를 사용합니다. 
AppCompatActivity implementing LifecycleOwner로 구현되어 있으나, 
appcompat-v7의 26.1.0 이하 버전에서는  LifecycleOwner를 명시해야 합니다.
LifecycleOwner는 android.arch.lifecycle:extensions library안에 포함되어 있습니다. 
implementation "android.arch.lifecycle:extensions:1.0.0-alpha9-1

  [source]: https://jankotlin.wordpress.com/2017/09/18/di-for-lazy-kotliniers/
  [github]: https://github.com/nisrulz/android-examples/tree/develop/LifeCycleCompForLib
  
  
  