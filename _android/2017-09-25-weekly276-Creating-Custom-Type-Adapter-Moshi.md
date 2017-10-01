---
layout: post
title:  "Weekly 276 Creating a Custom Type Adapter for Moshi"
date:   2017-09-25 0:10:00
description: Creating a Custom Type Adapter for Moshi
tag:
 - weekly 276
 - android
toc: true
---

# Creating a Custom Type Adapter for Moshi
## [Article][source] 
## [Github][github]

프로세스를 통해 사용자를 단계별로 안내하는 앱을 제작할 때 우리는 아마 그것을 표현하기 위해 열거 형을 사용합니다.

예를 들어, `NOT_STARTED`에서 `IN_PROGRESS`를 거쳐 `REJECTED` 또는 `COMPLETED` 중 하나로 이동하는 프로세스를 생각해보십시오

{% highlight java  %}
class enum Stage {
  NOT_STARTED,
  IN_PROGRESS,
  REJECTED,
  COMPLETED
}
{% endhighlight %}


Stage mapping 이 클라이언트와 서버에서 1 : 1이 아닌 경우에는 어떻게해야합니까? 
서버가 실제로 IN_PROGRESS 부분에서 여러 가지 작업을 수행한다면 어떻게 될까요? 
우리는 많은 관련성없는 열거 형이 생길 것입니다. 

{% highlight java  %}
class enum Stage {
  ...
  IN_PROGRESS_STEP1,
  IN_PROGRESS_STEP2, 
  ...
}
{% endhighlight %}

[Moshi][moshi]를 통해 이 문제를 쉽게 해결할 수 있습니다. 

먼저 열거 형에 @Json을 달아야합니다. 
따라서 Moshi는 열거 형으로 deserialising 할 때 원하는 값을 알고 있습니다.

{% highlight java  %}
class enum Stage {
  @Json(name = "not-started") NOT_STARTED,
  @Json(name = "in-progress") IN_PROGRESS,
  @Json(name = "rejected") REJECTED,
  @Json(name = "completed") COMPLETED
}
{% endhighlight %}

이제 deserialisation을위한 로직을 포함 할 StageAdapter를 생성하고, 
Moshi 인스턴스를 생성 할 때이를 적용해야합니다. 
어댑터는 클래스 일 뿐이며 상속받을 것은 없습니다. 

{% highlight java  %}
Moshi.Builder()
  .add(KotlinJsonAdapterFactory())
  .add(StageAdapter())
  .build()
{% endhighlight %}  

Moshi는 일반적으로 진행중인 문자열을 IN_PROGRESS로 암시 적으로 변환합니다. 
우리는 그 행동을 유지하기를 원하지만, 진행중인 문자열이 일치하는지 확인하기를 원합니다. 
일치하는 경우 IN_PROGRESS도 사용합니다.
Moshi 가 그것을 사용하기 위해 @FromJson가 달린 메소드를 정의할 필요가 있습니다.

{% highlight java  %} 
class StageAdapter {
  @FromJson fun fromJson(jsonReader: JsonReader, delegate: JsonAdapter<Stage>): Stage? {
    val value = jsonReader.nextString()
    return if (value.startsWith("in-progress")) Stage.IN_PROGRESS else delegate.fromJsonValue(value)
  }
}


{% endhighlight %}
두 개의 매개 변수를 사용하는 방법입니다. (jsonReader: JsonReader, delegate: JsonAdapter<Stage>)
JsonReader는 인코딩 된 토큰 스트림을 읽을 수있는 클래스이므로 여기에서 열거 형 값을 나타내는 원시 문자열을 가져올 수 있습니다.
JsonReader([text=:"in-progress"}])
JsonAdapter는 Kotlin 값을 Json으로 또는 Json에서 변환하는 역할을합니다.이 값을 사용하여 다른 enum 값을 deserialize하는 기존 동작을 유지할 수 있습니다.

  
  [source]: https://medium.com/@emmaguy/creating-a-custom-type-adapter-for-moshi-ae7e1cf469a9
  [github]: https://github.com/kimtaesu/android-weekly/blob/master/Weekly276-CreatingACustomTypeAdapterforMoshi/app/src/test/java/com/hucet/weekly276/StageDataTest.kt
  [moshi]: https://github.com/square/moshi
  
  