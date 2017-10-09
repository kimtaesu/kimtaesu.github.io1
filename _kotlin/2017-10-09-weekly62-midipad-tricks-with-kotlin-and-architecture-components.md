---
layout: post
title:  "Weekly 62 MidiPad – Tricks With Kotlin And Architecture Components"
date:   2017-10-09 0:10:00
description: MidiPad – Tricks With Kotlin And Architecture Components
tag:
 - weekly 62
 - kotlin
toc: true
---

# MidiPad – Tricks With Kotlin And Architecture Components
## [Article][source]
## [Github][github]

Musical Instrument Digital Interface (MIDI)는 전자 악기와 기타 장치가 서로 통신 할 수있는 표준입니다. 
1980 년대 초반부터 사용되어 왔으며 기본 사양은 거의 변경되지 않았습니다. 
Marshmallow (V6.0 - API 23)에서 안드로이드는 실제로 좋은 MIDI 지원을 받았으며
이 시리즈에서는 MIDI 컨트롤러 앱을 만드는 방법을 살펴볼 것입니다. 
이 Article에서는 Kotlin과 Architecture Components의 힘을 최대한 활용할 수있는 유용한 기술을 살펴 보겠습니다.


최초 사용 가능한 모든 기기를 찾아야 하며 Spinner에 표시합니다. 
그런 다음 우리는 외부 장치가 연결되거나 연결이 끊어져 있음을 감지하여 목록을 업데이트 할 수 있도록 새로운 장치를 검색해야 합니다.
다른 MIDI 기기와의 연결은 앱의 전체 라이프 사이클 동안 유지되어야 할 필요가 있는 항목이므로 안드로이드 아키텍처 구성 요소는 연결을 잃지 않고 
기기 회전에서 쉽게 생존 할 수 있도록 도와줍니다. 
이 시리즈의 아키텍처 구성 요소에 대해서는 자세히 설명하지 않겠지만 익숙하지 않은 경우 먼저 이 [시리즈][series]를 확인하십시오.

우선 MainActivity를 살펴봅시다  
{% highlight kotlin  %}
class MainActivity : AppCompatActivity() {
    private val lifecycleRegistry: LifecycleRegistry by lazyFast { LifecycleRegistry(this) }
    override fun getLifecycle(): LifecycleRegistry = lifecycleRegistry
 
    private val midiController: MidiController by viewModelProvider {
        MidiController(application)
    }
 
    private val deviceAdapter: DeviceAdapter by lazyFast {
        DeviceAdapter(this, { it.inputPortCount > 0 })
    }
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        ...
 
        midiController.observeDevices(this, deviceAdapter)
        
        ...
    }
 
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.main_menu, menu)
        menu?.findItem(R.id.app_bar_selector)?.actionView?.apply {
            findViewById<Spinner>(R.id.output_selector)?.apply {
                adapter = deviceAdapter
                onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
                    override fun onNothingSelected(parent: AdapterView<*>?) {
                        midiController.closeAll()
                    }
 
                    override fun onItemSelected(parent: AdapterView<*>?, view: View?, position: Int, id: Long) {
                        deviceAdapter[position].apply {
                            midiController.open(this)
                        }
                    }
                }
            }
        }
        return true
    }
 
    override fun onDestroy() {
        midiController.closeAll()
        super.onDestroy()
    }
}
{% endhighlight %}

MidiController는 모든 MIDI 기능을 처리하는 ViewModel 입니다. 
역활은 사용 가능한 MIDI 장치 목록이 변경 될 때마다 콜백을 수신 할 것입니다. 
이러한 콜백은 사용 가능한 각 MIDI 장치를 나타내는 MidiDeviceInfo 개체의 목록을 수신하며,
이 콜백을 처리하는 것은 deviceAdapter 인스턴스입니다 (잠시 후에 살펴 보겠습니다).


MainActivity 코드의 대부분은 사용자가 MIDI 이벤트를 보낼 사용자를 선택할 수 있도록 
사용 가능한 장치 목록을 포함하는 메뉴 항목 및 Spinner의 설정입니다. 
그러나 여기서 약간 흥미로운 몇 가지 트릭이 있습니다.

### 첫 번째 트릭은 **lazyFast**를 우리의 lazy 초기화에 사용하는 것 입니다.

{% highlight kotlin  %}
val object: Object by lazy { Object() }
{% endhighlight %}
 
한가지 issue가 있습니다.
default lazy 초기화는 Thread safe라는 것입니다. 
이 모든 것이 주 스레드에서 수행된다는 것을 알게되면 약간의 과장입니다.
우리는 ThreadSafe가 필요하지 않다는 것을 알고있는 곳에서 ThreadSafe를 하지 않도록 해서 성능을 쉽게 향상시킬 수 있습니다. 

{% highlight kotlin  %}
val object: Object by lazy(LazyThreadSafetyMode.NONE) { Object() }
{% endhighlight %}

lazyFast라는 함수를 만들어서 지원하도록 합니다. 

{% highlight kotlin  %} 
fun <T> lazyFast(operation: () -> T): Lazy<T> = lazy(LazyThreadSafetyMode.NONE) {
    operation()
}
{% endhighlight %}

lazyFast를 호출하여 ThreadSafe를 하지 않도록 하는 것이 가능합니다.  
{% highlight kotlin  %}
val object: Object by lazyFast { Object() }
{% endhighlight %}

###  다음 트릭은 어댑터 자체가 구성되는 방법입니다.

{% highlight kotlin  %}
class DeviceAdapter(private val context: Context,
                    private val filter: (MidiDeviceInfo) -> Boolean = { true },
                    private val items: MutableList<MidiDeviceInfo> = mutableListOf(),
                    private val adapter: ArrayAdapter<String> =
                        ArrayAdapter(context, android.R.layout.simple_spinner_item, mutableListOf())) :
        SpinnerAdapter by adapter,
        Observer<List<MidiDeviceInfo>> {
 
    init {
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
    }
 
    override fun onChanged(updatedItems: List<MidiDeviceInfo>?) {
        with(items) {
            clear()
            updatedItems?.also {
                addAll(it.filter(filter))
            }
            updateAdapter()
        }
    }
 
    private fun updateAdapter() =
            with(adapter) {
                clear()
                addAll(items.map {
                    context.getString(R.string.device_name,
                            it.properties.getString(MidiDeviceInfo.PROPERTY_MANUFACTURER),
                            it.properties.getString(MidiDeviceInfo.PROPERTY_PRODUCT))
                })
            }
 
    operator fun get(index: Int) = items[index]
 
}
{% endhighlight %}


`onChanged()`는 사용 가능한 MIDI 장치 목록이 변경 될 때마다 호출되며 필터를 적용하는 MidiDeviceInfo 개체의 내부 목록을 업데이트합니다. 
이전에 전달 된 필터가 주어지면 우리는 MIDI 입력을 처리합니다. 
그런 다음 각 MidiDeviceInfo를 장치 Manufacturer 및 Product 이름이 들어있는 String으로 변환하는 ArrayAdapter의 데이터를 업데이트합니다.

`get()` 연산자를 사용하면 Spinner의 항목 인덱스에서 특정 MidiDeviceInfo 항목을 조회 할 수 있습니다. 

결과적으로 SpinnerAdapter는 LiveData 객체로부터 콜백을 수신 할 때마다 자동으로 데이터를 업데이트하며 상대적으로 적은 양의 코드를 사용하여 수행됩니다.

### 마지막 트릭은 ViewModel 인스턴스를 얻는 방법입니다.

{% highlight kotlin  %}
private val midiController: MidiController by viewModelProvider {
    MidiController(application)
}
{% endhighlight %}

```kotlin
inline fun <reified VM : ViewModel> FragmentActivity.viewModelProvider(crossinline provider: () -> VM) = lazyFast {
    object : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: Class<T>) =
                provider() as T
    }.let {
        ViewModelProviders.of(this, it).get(VM::class.java)
    }
}
```

FragmentActivity의 Extension function으로 
어느 곳에서든 호출 될 수 있으며
적절한 ViewModel 객체를 반환하거나 이미 존재하지 않는 경우 새 인스턴스를 생성합니다.

viewModelProvider가 하는 것은
Provider의 Lambda로부터 요구 된 객체의 인스턴스를 생성 할 수있는 팩토리를 제공하면서,
이 클래스의 ViewModelProvider의 인스턴스를 취득하는 것입니다. 
그런 다음 ViewModelProvider는 캐시 된 인스턴스를 반환하거나 필요한 경우 새 인스턴스를 만듭니다.

  [source]: https://blog.stylingandroid.com/midipad-tricks-with-kotlin-and-architecture-components/
  [github]: https://github.com/StylingAndroid/MidiPad    
  [series]: https://blog.stylingandroid.com/architecture-components-lifecycle/ 