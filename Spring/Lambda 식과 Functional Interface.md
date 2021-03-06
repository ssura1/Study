<h1>Lambda식과 Functional Interface</h1>

<h2>Standard Functional Interfaces</h2>

* Functional Interface들은 `java.util.function` 패키지 내에 위치해 있다.   
  이 패키지는 개발자가 Lambda 식과 Method Reference를 사용하기 쉽게 해준다.

* 이 패키지에 있는 모든 인터페이스들은 General 타입을 수용하며, 추상적으로 되어 있어   
  사실상 거의 모든 Lambda식을 수용할 수 있게 해준다.

* `Foo`라는 인터페이스가 있다고 가정해보자.
```java
@FunctionalInterface
public interface Foo {
    String method(String string);
}
```

* 그리고 `UseFoo`라는 클래스의 `add()` 메소드에서 위의 인터페이스를 파라미터로 가진다고 하자.
```java
public class UseFoo {
   public String add(String string, Foo foo) {
       return foo.method(string);
   }
}
```

* 위의 코드를 실행하려면 아래와 같이 작성할 것이다.
```java
// Foo 인터페이스의 method()로 작동할 Lambda식 정의
Foo foo = parameter -> parameter = " from lambda.";
String result = useFoo.add("Message", foo);
```

* 코드를 보면, `Foo`는 단지 하나의 인자를 받고, 새로운 값을 반환하는 함수에 불과하다.   
  Java 8은 이러한 경우를 위해 `java.util.function` 패키지에 `Function<T, R>`라는 인터페이스를 제공한다.

* 위의 패키지를 이용해서 `Foo`를 사용하지 않고, 코드를 작성하면 아래와 같다.
```java
public String add(String string, Function<String, String> fn) {
    return fn.apply(string);
}
```

* 이제 위 코드를 실행하는 코드를 작성해보자.
```java
// 위의 fn.apply()에서 사용될 apply 메소드를 Lambda식으로 정의
Function<String, String> fn = parameter -> parameter + " from lambda.";
String result = useFoo.add("Message", fn);
```
<hr/>

<h2>@FunctionalInterface의 사용</h2>

* Functional Interface들은 `@FuntionalInterface` 어노테이션을 적용하여 만들 수 있다.   
  처음에는 이 어노테이션을 적용하는 것이 무쓸모해 보일 수도 있다.   
  이 어노테이션이 없더라도, 해당 Functional Interface는 하나의 추상 메소드를 가진   
  인터페이스로 취급될 것이다.

* 하지만 프로젝트가 커지며, 이러한 Functional Interface들이 많아지면, 하나 하나 관리하기가 그만큼   
  어려워질 것이다. 하지만 `@FunctionalInterface` 어노테이션을 적용하면 만약 코드의 다른 부분에서   
  개발자가 실수로 기존에 정의된 Funtional Interface를 수정하려고 한다면 컴파일러가 에러를 띄운다.   
  따라서 이는 다른 개발자들과 협업할 때 매우 유용하게 사용할 수 있는 도구이기도 하다.

* 따라서, 아래와 같이 작성하도록 하자.
```java
@FunctionalInterface
public interface Foo {
    String method();
}
```
<hr/>

<h2>Functional Interface의 기본 메소드 과도하게 사용하지 않기</h2>

* 개발자는 Functional Interface의 default 메소드를 쉽게 추가할 수 있다.   
  하지만 이는 추상 메소드의 선언이 단 1개만 되어있을 때 `Functional Interface Contract`를 만족한다.
```java
@FunctionalInterface
public interface Foo {
    String method(String string);
    default void defaultMethod() {}
}
```

* Functional Interface는 추상 메소드가 동일한 시그니처(Signature)를 가질 때에만   
  다른 Functional Interface에 의해 상속될 수 있다. 아래는 예시이다.
```java
@FunctionalInterface
public interface FooExtended extends Baz, Bar {}

@FunctionalInterface
public interface Baz {
    String method(String string);
    default String defaultBaz() {}
}

@FunctionalInterface
public interface Bar {
    String method(String string);
    default String defaultBar() {}
}
```

* 위 Functional Interface 중 `Baz#method(String string)`와 `Bar#method(String string)`는   
  동일한 시그니처를 가지기 때문에 함께 `FooExtended`가 상속할 수 있는 것이다.

* 일반적인 인터페이스들과 마찬가지로, 동일한 default 메소드를 가진 Functional Interface들을   
  함께 상속하는 것은 허용되지 않는다.
```java
@FunctionalInterface
public interface Baz {
    String method(String string);
    default String defaultBaz() {}
    default String defaultCommon() {}
}

@FunctionalInterface
public interface Bar {
    String method(String string);
    default String defaultBar() {}
    default String defaultCommon() {}
}
```

* 위 코드에서 `Baz`와 `Bar`는 `defaultCommon()`이라는 동일한 default 메소드를 가지기 때문에   
  두 인터페이스를 다른 인터페이스가 함께 다중 상속하는 것은 컴파일 에러를 일으킨다.
```
interface FooExtended inherits unrelated defaults for defaultCommon() from types Baz and Bar...
```

* 이를 해결하기 위해 `defaultCommon()` 메소드는 `FooExtended` 인터페이스에서 오버라이딩이 되어야 한다.   
  물론 사용 시에 원하는대로 구현할 수도 있다. 하지만 인터페이스에서 바로 오버라이딩하면,   
  `Bar` 또는 `Baz`에서 구현한 `defaultCommon()`을 바로 사용할 수 있다.

* 간단히 말해, 동일한 함수를 가진 인터페이스들을 다중 상속받는 인터페이스에 오버라이딩을 한다면,   
  해당 메소드를 정의한 부모 인터페이스의 구현체를 사용할 수 있다는 것이다.
```java
// 오버라이딩을 통해 Bar#defaultCommon()을 호출한다.
@FunctionalInterface
public interface FooExtended extends Baz, Bar {
    @Override
    default String defaultCommon() {
        return Bar.super.defaultCommon();
    }
}
```

* 하지만 너무 많은 Default 메소드를 인터페이스에 정의하는 것은 아키텍쳐의 관점에서는 좋지 않다는 점을 명심하자.
<hr/>

<h2>Functional Interface를 Lambda식으로 인스턴스화하기</h2>

* 컴파일러는 Inner class를 통해 Functional Interface를 인스턴스화 하는 것을 허용한다.   
  하지만 이는 코드가 복잡해질 수 있으므로, Lambda식을 활용하면 간단하게 구현할 수 있다.
```java
// Lambda식 사용 시
Foo foo = parameter -> parameter + " from Foo.";

// Inner class 사용 시
Foo fooImplemented = new Foo() {
    @Override
    public String method(String string) {
        return string + " from Foo.";
    }
}
```

* Lambda식을 사용한 인터페이스의 인스턴스화는 모든 라이브러리(예전 라이브러리)에 대해 가능하다.   
  이는 곧 `Runnable`, `Comparator` 등에도 적용 가능함을 의미한다.
<hr/>

<h2>Functional Inteface를 파라미터로 취급해서 메소드를 오버로딩하는 것은 좋지 않다.</h2>

* 메소드를 다른 이름으로 만들어서 충돌을 방지해야 한다. 아래 예시를 보자.
```java
public interface Processor {
    String process(Callable<String> c) throws Exception;
    String process(Supplier<String> s);
}

public class ProcessorImpl implements Processor {
    @Override
    public String process(Callable<String> c) throws Exception {
        // 구현
    }

    @Override
    public String process(Supplier<String> s) {
        // 구현
    }
}
```

* 그리고 위의 `Processor` 인터페이스를 람다함수를 통해 인스턴스화 해보자.
```java
String result = processor.process(() -> "abc");
```

* 위 코드는 동작할 것 처럼 보이지만, `ProcessorImpl`의 메소드 중 어느것을 실행하려고 하면 실패한다.
```
reference to process is ambiguous
both method process(java.util.concurrent.Callable<java.lang.String>) 
in com.sangwoo.java8.lambda.ProcessorImpl 
and method process(java.util.function.Supplier<java.lang.String>) 
in com.sangwoo.java8.lambda.ProcessorImpl match
```

* 이를 해결하는 방법은 두 가지가 있는데, 메소드 네이밍을 달리 하거나,   
  명시적으로 casting을 지정하는 것이다.
```java
// 메소드 네이밍 변경
public interface Processor {
    String processWithCallable(Callable<String> c) throws Exception;
    String processWithSupplier(Supplier<String> s);
}

// 호출 시 명시적으로 casting 적용
String result = processor.process((Supplier<String>) () -> "abc");
```
<hr/>

<a href="https://www.baeldung.com/java-8-lambda-expressions-tips">참고 링크</a>