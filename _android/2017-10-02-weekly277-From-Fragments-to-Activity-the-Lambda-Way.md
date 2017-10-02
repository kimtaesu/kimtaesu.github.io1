---
layout: post
title:  "Weekly 277 From Fragments to Activity: the Lambda Way"
date:   2017-09-25 0:10:00
description: From Fragments to Activity the Lambda Way
tag:
 - weekly 277
 - android
toc: true
---

# From Fragments to Activity: the Lambda Way
## [Article][source] 
## [Github][github]

Fragment와 Activity 사이의 통신은 Activity가 구현할 수있는 Fragment에 Listener를 전달하는 것처럼 간단하지 않습니다. 
Activity가 곧 Rotate하고 Fragment는 죽어서 유출 된 Activity에 대한 참조를 유지합니다. 
Google은 목표를 달성하고 누출 위험을 줄이기 위해 ["Good old pattern"][goodoldpattern]을 권장합니다. 
이 Article은 우리는 람다가 더 나은 솔루션이고 표현력이 풍부하며 유지 보수가 더 쉽다는 것을 입증 할 것입니다.

# The Good Old Way

공식적인 방법은 Activity 안에 Fragment가 Activity와 통신하도록 하는 interface 작성하고 구현하는 것 입니다.
 
{% highlight java  %}

public static class MainActivity extends Activity
        implements HeadlinesFragment.HeadlineListener{
    ...
  public void onArticleSelected(int position) {
   // The user selected the headline of an article from the HeadlinesFragment
   // Do something here to display that article
  }
}
{% endhighlight %}

Fragment는 위험한 캐스트를 사용하여 Acitivty에 대한 참조를 얻고 통신 인터페이스를 통해 Activity로 전달합니다.

{% highlight java  %}

public class HeadlinesFragment extends ListFragment {
    HeadlineListener mCallback;

  // Container Activity must implement this interface
  public interface HeadlineListener {
    public void onArticleSelected(int position);
  }

  @Override
  public void onAttach(Activity activity) {
   super.onAttach(activity);

   // Makes sure that the container activity has implemented
   // the callback interface. If not, it throws an exception
     try {
      mCallback = (HeadlineListener) activity;
     } catch (ClassCastException e) {
       throw new ClassCastException(activity.toString()
           + " must implement HeadlineListener");
     }
   }

   @Override
   public void onListItemClick(ListView l, View v, int p, long i) {
     // Send the event to the host activity
     mCallback.onArticleSelected(p);
   }
{% endhighlight %}


이 방법에는 몇 가지 문제가 있습니다. 
* the casting은 런타임에 실패할 수 있습니다. 
* Fragment와 Activity가 어떻게 연결되어 있는지 명확하게 알 수 없습니다.
 
# The Lambda Way

우리는 Activity 클래스가 Fragment와 통신을 위한 인터페이스를 더 이상 구현하고 싶지 않습니다. 

{% highlight java  %}
public static class MainActivity extends Activity {
  public void onArticleSelected(int position) {
  }
}
{% endhighlight %}

우리의 통신 인터페이스는 모든 Activity을 사용하도록 수정되었습니다. 
따라서 Activity 인스턴스와 독립적이되어 Leak이 발생하지 않습니다. 
{% highlight java  %}
public interface HeadlineListener<T extends Activity> extends Serializable {
  public void onArticleSelected(T activity, int position);
}
{% endhighlight %}


Fragment는 현재 호스팅중인 Activity 인스턴스를 전달하고자 할 때 리스너에게 전달합니다.
{% highlight java  %}
public class HeadlinesFragment extends ListFragment {
    HeadlineListener mCallback;

  public <T> void setHeadlineListener(HeadlineListener<T> listener) {
   this.mCallBack = listener;
  }

   @Override
   public void onListItemClick(ListView l, View v, int p, long i) {
     // Send the event to the host activity
     mCallback.onArticleSelected(getActivity(), p);
   }
}
{% endhighlight %}

이 부분이 핵심입니다.  **이제 Activity는 간단한 메소드 참조를 사용하여 Fragment에 리스너를 설정합니다.**

{% highlight java  %}
public static class MainActivity extends Activity {
  public void bindFragment(HeadlinesFragment fragment) {
    fragment.setHeadlineListener(MainActivity::onArticleSelected);
  }
  public void onArticleSelected(int position) {
    ...
  }
}
{% endhighlight %}

## What is this method reference : MainActivity::onArticleSelected ?
* 정적 메소드에 대한 참조 
* 특정 유형의 객체에 대한 정적이 아닌 메소드에 대한 참조

## MainActivity :: onArticleSelected는 무엇을 참조합니까 ??

MainActivity, int의 두 매개 변수를 사용하는 메서드를 가리 킵니다.
이제 MainActivity 인스턴스와 독립적이므로 그것에 대한 참조가 Activity instance의 leak이 발생하지 않습니다!

{% highlight java  %}
void onArticleSelected(T activity, int position);
{% endhighlight %}

Lambda 관점에서 보면,이 메소드 참조의 의미는 약간 다릅니다. 
실제로 MainActivity를 받아들이고 이 Acrtivity에서 onArticleSelected 메소드를 호출하는 메소드입니다.
 
{% highlight java  %}
activity -> ((MainActivity)activity).onArticleSelected(position)
{% endhighlight %}


# Disambiguating MainActivity::onArticleSelected

메서드 참조는 Java 컴파일러에서 잘못 판단되어 정적 메소드 참조로 이해할 수 있습니다. 
이를 명확히하기 위해 인터페이스 HeadlineListener와 메소드 HeaderFragment # setHeadlineListener는 **제네릭**을 사용하여 대상 유형 유추를 트리거합니다.
{% highlight java  %}
public interface HeadlineListener<T extends Activity> extends Serializable {
  public void onArticleSelected(T activity, int position);
}
{% endhighlight %}

{% highlight java  %}
  public class HeadLineFragment extends Fragment {
    public <T> void setHeadlineListener(HeadlineListener<T> listener) {
      ...
    }
  }

{% endhighlight %}

위에 두 generics 덕분에 컴파일러는 메서드 참조가 setHeadlineListener에 전달되어 **특정 유형의 개체에 대한 정적 메서드가 아닌 참조를 전달한다는 것을 이해하고 Activity를 확장하는 클래스 내부의 메서드를 가리키는 참조**만 받아들입니다.
 
{% highlight java  %}
  public class HeadLineFragment extends ListFragment {
   @Override
   public void onListItemClick(ListView l, View v, int p, long i) {
     // Send the event to the host activity
     // we should probably check if the activity is finishing|destroyed|rotating
     mCallback.onArticleSelected(getActivity(), p);
   }
  } 
{% endhighlight %}  

결론
* Activity, Fragment 간의 통신은 이제 명시적입니다. Good Old Way 인터페이스를 구현하는 것과는 반대로 IDE에서 쉽게 추적 할 수 있습니다.
* Good Old Way의 activity를 casting 제거
* 우리는 상속에 대한 구성을 선호합니다.
* 코드는 현대 Rx 체인의 나머지 부분과 매우 비슷하게 보입니다.
* 이러한 API를 사용하여 Fragment를 사용하는 경우 리스너 인터페이스를 사용하는 것은 메소드 참조를 사용하는 것만 큼 쉽고 솔루션의 복잡성은 대부분 장면 뒤에서 발생합니다.

  [source]: https://medium.com/groupon-eng/from-fragments-to-activity-the-lambda-way-32c768c72aa9
  [github]: https://github.com/stephanenicolas/activtity-fragment-lambda
  [goodoldpattern]: https://developer.android.com/training/basics/fragments/communicating.html
  
  