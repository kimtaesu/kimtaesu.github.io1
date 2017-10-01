---
layout: post
title:  "Weekly 61 Inlining Kotlin Properties"
date:   2017-09-25 0:10:00
description: Inlining Kotlin Properties
tag:
 - weekly 61
 - kotlin
toc: true
---

# Inlining Kotlin Properties
## [Article][source]


Kotlin에서 [속성을 인라인][inline] 할 수 있다는 것을 알고 계셨습니까? 
Kotlin 문서를 탐색하면서이 작은 기능을 우연히 발견했으며 짧은 기사의 가치가 있다고 생각했습니다.
 
## 그렇다면 왜 Kotlin 에서 속성을 인라인 할 수 있는 것이 유용할까요?

간단한 예제를 살펴보도록 합시다

{% highlight kotlin  %}

val View.isVisible  
  get() = visibility == View.VISIBLE
  
if (button.isVisible) {  
    Toast.makeText(context, "I'm a button!", Toast.LENGTH_SHORT).show()
}
{% endhighlight %}

Tools -> Kotlin -> Show Kotlin Bytecode 로 ByteCode를 확인할 수 있습니다. 

위 코드의 ByteCode를 확인하면 아래와 같은 결과가 나옵니다. 

`isVisible`은 실제로 extension property 입니다.  

extension function와 같이 첫 번째 메서드 매개 변수는 확장 함수에서 Receiver가 됩니다. 

[Weekly#60 Kotlin’s Extension functions][extension]에서 Kotlin의 Extension Function은 정적 메소드로만 동작한다는 것을 확인한 사실이 있습니다.

 
{% highlight kotlin  %} 
if(ViewKt.isVisible((View)button)) {  
    Toast.makeText(CheckNowView.this.getContext(), (CharSequence)"I'm a button!", 0).show();
}
{% endhighlight %}

예상대로 코드는 첫 번째 인수로 button을 전달하여 ViewKt 클래스의 static isVisible () 메서드 
(확장 속성이있는 파일은 view.kt)를 호출합니다. 
속성의 getter를 인라인으로 표시하고 변경 내용을 확인해 봅시다.

{% highlight kotlin  %}

val View.isVisible  
  inline get() = visibility == View.VISIBLE
{% endhighlight %}

Decompiled:

{% highlight kotlin  %}
View $receiver$iv = (View)button;  
if($receiver$iv.getVisibility() == 0) {  
  Toast.makeText(CheckNowView.this.getContext(), (CharSequence)"I'm a button!", 0).show();
}
{% endhighlight %}

**인라인되어 정적 메서드 호출이 제거**되었습니다. 이 코드는 Java로 작성하는 것처럼 보이지만 Kotlin의 코드는 훨씬 더 깔끔하게 보입니다.

  [source]: https://blog.egorand.me/inlining-kotlin-properties/
  [inline]: https://kotlinlang.org/docs/reference/inline-functions.html#inline-properties
  [extension]: /kotlin/2017-09-25-weekly60-Kotlins-Extension-functions/  
  
  