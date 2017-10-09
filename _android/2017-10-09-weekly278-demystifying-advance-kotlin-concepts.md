---
layout: post
title:  "Weekly 278 Demystifying Advanced Kotlin Concepts"
date:   2017-10-09 0:10:00
description: Demystifying Advanced Kotlin Concepts
tag:
 - weekly 278
 - android
toc: true
---

# Demystifying Advanced Kotlin Concepts
## [Article][source] 

Article에서 총 7개의 주제가 있지만, 제가 궁금해했던 것이나 몰랐던 사실들에 대해서 정리를 하도록 하겠습니다.   
* 01. Local Functions
* 03. Anonymous Functions
* 04.Inline Functions

### 01. Local Functions

```kotlin
fun OuterFunction(param:Int){
  val outerVar = 11
  fun localFunction(){
    println(param)
    println(outerVar)
  }
}
```
지역 함수는 함수 내부의 함수입니다. 
로컬 함수는 외부 함수의 매개 변수 및 해당 로컬 변수 (즉, [클로저][closure])에 액세스 할 수 있습니다.

용도는 무엇인가?
이는 코드 재사용을 원할 때 유용합니다. 즉, 최상위 함수를 만들지 않겠습니다. 또는 원하지 않습니다.
클래스 외부에서 멤버 함수를 만들 수 있습니다.이 방법으로 그룹화하는 것이 좋습니다.

> **Note** 외부에서는 outerFunction의 localFunctions에 액세스 할 수 없습니다.


### 03. Anonymous Functions

람다식이 코드 블록으로 전달되는 `higher-order functions`를 본적이 있습니다. 

그러나 `Anonymous Functions`은 약간 다릅니다. 

op라는 함수가 있다고 가정합시다.
```kotlin
fun op(x:Int,op:(Int) -> Int):Int{
  return op(x)
}
```

lambdas으로 표현하면 : 

```kotlin
op(3,{it*it})
//here {it*it} is lambda expression
```

Anonymous으로 표현하면 : 
```kotlin
//can have multiple returns

op(3,fun(x):Int{
         if(x>10) return 0
         else return x*x
        })
```

`Anonymous Functions`는 일반 함수의 전체 본문을 갖지만 이름은 없습니다.

### 04.Inline Functions

kotlin의 lambda 표현식은 java의 익명 클래스에 대한 방법으로 제공합니다.
> 람다 표현식에 클로저가 있으면 인스턴스가 생성되어 더 많은 메모리의 오버 헤드가 추가됩니다.
> 또한 이러한 모든 람다식이 호출 스택에 영향을줍니다.


**NonInline**

```kotlin
fun op(op:()->Unit){
    println("This is before lambda")
    op()
    println("this is after lambda")
}

fun main(args: Array<String>) {
    op({println("this is the actual function")})
}
```

**NonInline kotlin bytecode**
```java
import kotlin.Metadata;
import kotlin.jvm.functions.Function0;
import kotlin.jvm.internal.Intrinsics;
import org.jetbrains.annotations.NotNull;

@Metadata(
   mv = {1, 1, 7},
   bv = {1, 0, 2},
   k = 2,
   d1 = {"\u0000\u001a\n\u0000\n\u0002\u0010\u0002\n\u0000\n\u0002\u0010\u0011\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0018\u0002\n\u0000\u001a\u0019\u0010\u0000\u001a\u00020\u00012\f\u0010\u0002\u001a\b\u0012\u0004\u0012\u00020\u00040\u0003¢\u0006\u0002\u0010\u0005\u001a\u0014\u0010\u0006\u001a\u00020\u00012\f\u0010\u0006\u001a\b\u0012\u0004\u0012\u00020\u00010\u0007¨\u0006\b"},
   d2 = {"main", "", "args", "", "", "([Ljava/lang/String;)V", "op", "Lkotlin/Function0;", "production sources for module IdeaProjects-random_main"}
)
public final class InlineFunctionKt {
   public static final void op(@NotNull Function0 op) {
      Intrinsics.checkParameterIsNotNull(op, "op");
      String var1 = "This is before lambda";
      System.out.println(var1);
      op.invoke();
      var1 = "this is after lambda";
      System.out.println(var1);
   }

   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      op((Function0)null.INSTANCE);
   }
}
```

**Inline**
```kotlin
inline fun op(op:()->Unit){
    println("This is before lambda")
    op()
    println("this is after lambda")
}

fun main(args: Array<String>) {
    op({println("this is the actual function")})
}
```

**Inline kotlin bytecode**
```java

import kotlin.Metadata;
import kotlin.jvm.functions.Function0;
import kotlin.jvm.internal.Intrinsics;
import org.jetbrains.annotations.NotNull;

@Metadata(
   mv = {1, 1, 7},
   bv = {1, 0, 2},
   k = 2,
   d1 = {"\u0000\u001a\n\u0000\n\u0002\u0010\u0002\n\u0000\n\u0002\u0010\u0011\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0018\u0002\n\u0000\u001a\u0019\u0010\u0000\u001a\u00020\u00012\f\u0010\u0002\u001a\b\u0012\u0004\u0012\u00020\u00040\u0003¢\u0006\u0002\u0010\u0005\u001a\u0017\u0010\u0006\u001a\u00020\u00012\f\u0010\u0006\u001a\b\u0012\u0004\u0012\u00020\u00010\u0007H\u0086\b¨\u0006\b"},
   d2 = {"main", "", "args", "", "", "([Ljava/lang/String;)V", "op", "Lkotlin/Function0;", "production sources for module IdeaProjects-random_main"}
)
public final class InlineFunctionKt {
   public static final void op(@NotNull Function0 op) {
      Intrinsics.checkParameterIsNotNull(op, "op");
      String var2 = "This is before lambda";
      System.out.println(var2);
      op.invoke();
      var2 = "this is after lambda";
      System.out.println(var2);
   }

   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      String var1 = "This is before lambda";
      System.out.println(var1);
      String var2 = "this is the actual function";
      System.out.println(var2);
      var1 = "this is after lambda";
      System.out.println(var1);
   }
}
```
**차이점은 Inline 함수 전체 코드를 해당 함수를 호출하는 곳으로 복사하고 전달됩니다.**

### 05. Returns And Local Returns

```kotlin
fun ContainingFunction(){
  val nums=1..100
  numbers.forEach{
    if(it%5==0){
      return
      }
    }
  println("Hello!")
}
```

**Hello!**는 콘솔에 찍히지 않는다.

**Solve : labels** 

```kotlin
...
  if(it%5==0){
    return@forEach //i.e. name of the higher order function
  }
...
//or you could define your own labels
numbers.forEach myLabel@{
          if(it%5==0){
            return@myLabel  
         }
   ....
```

  [source]: https://dev.to/praveenkajla/demystifying-advance-kotlin-concepts-a97
  [closure]: https://ko.wikipedia.org/wiki/%ED%81%B4%EB%A1%9C%EC%A0%80_(%EC%BB%B4%ED%93%A8%ED%84%B0_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
  [infix]: /2017-10-01-weekly61-infix-in-kotlin/