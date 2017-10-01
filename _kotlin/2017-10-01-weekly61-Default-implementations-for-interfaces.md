---
layout: post
title:  "Weekly 61 Default implementations for interfaces"
date:   2017-09-25 0:10:00
description: Default implementations for interfaces
tag:
 - weekly 61
 - kotlin
toc: true
---

# Default implementations for interfaces

## [Article][source]

예제를 통해 살펴봅시다.

View라는 Simple한 interface가 있습니다. 

{% highlight kotlin  %}
interface View {
    fun showToast(text: String)
}
{% endhighlight %}

View를 상속받는 Interface로 `showToast` Default implementation을 만들 수 있습니다.

{% highlight kotlin  %}
interface DefaultView : View {
    val context: Context

    override fun showToast(text: String) {
        Toast.makeText(context, text, Toast.LENGTH_SHORT).show()
    }
}
{% endhighlight %}


DefaultView라는 별개의 인터페이스를 만들었습니다. 
보기에는 구현이 포함될 수도 있지만 테스트가 더 어려워집니다. 
다른 모든 View 인터페이스는 View를 확장 할 수 있지만 
이를 구현하는 모든 클래스는 다음과 같이 DefaultView를 직접 구현할 수 있습니다.

{% highlight kotlin  %}
abstract class BaseActivity : Activity(), DefaultView {
    override val context: Context
        get() = this
}
{% endhighlight %}


# How does this work?

Bytecode를 살펴봅시다.

interface : `DefaultView`
 * static showToast 를 가진 `DefaultImpls` static class가 생성됨
 * 첫 번째 Parameter로 DefaultView 인스턴스가 전달됨
   - 이렇게하면 인터페이스에서 다른 기능이나 속성에 액세스 할 수 있습니다.
 
{% highlight kotlin  %}
public interface DefaultView extends View {
    @NotNull
    Context getContext();

    void showToast(@NotNull String var1);
    
    public static final class DefaultImpls {
        public static void showToast(@NotNull DefaultView $this, String text) {
            Intrinsics.checkParameterIsNotNull(text, "text");
            Toast.makeText($this.getContext(), (CharSequence)text, 0).show();
        }
    }
}
{% endhighlight %}


class : `BaseActivity`

`DefaultView.DefaultImpls.showToast` static method를 호출합니다. 

{% highlight kotlin  %}
public abstract class BaseActivity extends Activity implements DefaultView {

    @NotNull
    public Context getContext() {
        return (Context)this;
    }

    public void showToast(@NotNull String text) {
        Intrinsics.checkParameterIsNotNull(text, "text");
        DefaultView.DefaultImpls.showToast(this, text);
    }
}
{% endhighlight %}


## Default Implements 는 몇 가지 제약사항이 있습니다. 
* BaseActivity는 Kotlin으로 작성해야합니다. 그렇지 않으면 DefaultView의 모든 함수를 구현해야 합니다.
* 특성의 이름 지정에 주의 해야 합니다. 위의 예제에서 context 속성은 Fragment의 getContext () 메서드와 충돌합니다.
 
  [source]: https://medium.com/a-problem-like-maria/default-implementations-for-interfaces-afd0b9316a8a
  
  