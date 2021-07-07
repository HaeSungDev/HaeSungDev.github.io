---
title: "Effective Java 열거 타입과 애너테이션"
search: true
categories: 
  - Java
last_modified_at: 2021-07-06T21:00:00
---

**이 글은 프로그래밍인사이트의 Effective Java 3/E를 읽고 요약 정리한 글입니다.**

**책 내용이 훨씬 정확하니 책을 사서 보세요**

**문제 시에 글을 내리겠습니다**

# 이펙티브 자바 - 열거 타입과 애너테이션

상수를 그룹으로 묶어서 표현해주기 위한 열거 타입과 메타 정보를 표현하기 위한 인터페이스의 일종인 에너테이션을 올바르게 사용하는 방법을 알아보겠습니다.

## Item 34. int 상수 대신 열거 타입을 사용하라

열거 타입은 상수 값들을 정의한 후 정의 된 상수 값 이외의 값은 허용하지 않는 타입입니다. 열거 타입 전에는 정수 열거 패턴을 사용했었는데 단점이 많습니다. 

1. int형 타입을 사용하므로 상수에 대한 타입 보장이 안되고 전혀 다른 그룹의 상수와 비교해도 아무런 오류를 발생시키지 않습니다.
2. 상수 값이 바뀌면 클라이언트도 컴파일을 다시 해줘야하고 컴파일 하지 않고 실행이 되도 버그가 발생하기 쉽습니다.
3. 상수명을 문자열로 출력하는 것이 불가능합니다.
4. 그룹에 속한 모든 상수를 순회하는 방법도 없습니다.

위에서 말한 단점은 열거 타입을 사용하면 모두 해결됩니다.

```
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
```

위 선언처럼 Apple이라는 열거형 그룹에 속한 상수들을 묶어서 표현할 수 있습니다. 또한 Apple 참조형에는 Apple에 속한 상수만 할당할 수 있으므로 타입 안전합니다.

```
Apple apple = Orange.NAVEL; // compile error
```

열거형 타입에서 toString을 사용하면 상수의 이름 문자열을 얻을 수 있습니다.

```
Apple.FUJI.toString(); // "FUJI"
```

values 메소드를 사용하면 모든 상수를 담고있는 배열을 얻을 수 있습니다.

```
Apple.values(); // 모든 상수를 포함한 Apple[]
```

이 외에도 Comparable, Serializable을 기본으로 구현하고 있고 열거형 상수에 필요한 메소드 기능 추가 등 다양한 장점이 있습니다.

```
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;

  public double apply(double x, double y) {
    switch (this) {
      case PLUS: return x + y;
      case MINUS: return x - y;
      case TIMES: return x * y;
      case DIVIDE: return x / y;
    }
    throw new AssertionError("unreachable operation: " + this);
  }
}
```

위 처럼 열거형 상수마다 메소드를 분기해서 다른 동작을 제공할 수 있습니다. 다만 위 처럼 switch문으로 분기하다보면 실수할 수 도 있고 연산자가 늘어나면 관리가 힘드므로 다음과 같이 추상 메소드를 활용할 수 있습니다.

```
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y) { return x + y; }
  },
  MINUS("-") {
    public double apply(double x, double y) { return x - y; }
  },
  TIMES("*") {
    public double apply(double x, double y) { return x * y; }
  },
  DIVIDE("/") {
    public double apply(double x, double y) { return x / y; }
  };

  private final String symbol;

  Operation(String symbol) { this.symbol = symbol; }

  @Override
  public String toString() { return symbol; };

  public abstract double apply(double x, double y);
}
```

위 처럼 추상 메소드를 이용해서 상수마다 다른 메소드를 재정의할 수 있고 상수마다 필드값도 관리해 줄 수 있습니다. 위와 같은 방식에도 단점은 있는데 상수마다 공통으로 사용되는 로직을 관리해주기 힘들다는 단점이 있습니다. 이럴 떄는 전략 열거 타입 패턴을 사용하면 됩니다.

```
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y) {
      stategy.commonLogic();
      return x + y;
    }
  },
  MINUS("-") {
    public double apply(double x, double y) { return x - y; }
  },
  TIMES("*") {
    public double apply(double x, double y) { return x * y; }
  },
  DIVIDE("/") {
    public double apply(double x, double y) { return x / y; }
  };

  private final String symbol;
  private final Strategy strategy;

  Operation(String symbol, Strategy strategy) {
    this.symbol = symbol;
    this.strategy = strategy;
  }

  @Override
  public String toString() { return symbol; };

  public abstract double apply(double x, double y);

  enum Strategy {
    STRATEGY1 { ... }
    STRATEGY2 { ... }

    public abstract void commonLogic();
  }
}
```

위와 같이 전략 열거 타입 패턴을 사용하면 공통 로직을 관리하기 용이합니다.

## Item 35. ordinal 메소드 대신 인스턴스 필드를 사용하라

enum 타입의 ordinal 메소드는 EnumSet, EnumMap과 같이 열거 타입 기반의 범용 자료구조를 개발하는데 사용하도록 설계됐습니다. ordinal 메소드는 정의 순서에 따라 값이 바뀌기 때문에 실수를 할 가능성이 높습니다. 따라서 ordinal 메소드는 사용하지 않고 인스턴스 필드를 사용하는 것이 안전합니다.

## Item 36. 비트 필드 대신 EnumSet을 사용하라

상수의 집합을 비트의 OR 연산으로 묶으면 정수형 자료형 하나로 상수 집합을 표현할 수 있다. 하지만 이 방식은 개발자가 값의 의미를 파악하기 쉽지않고 상수 값이 변경됐을 때 실수할 가능성이 높다. 따라서 내부적으로 비트 필드를 관리해주는 EnumSet을 사용하면 성능 저하가 거의 없이 상수의 집합을 표현할 수 있다.

## Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

ordinal 값을 배열 인덱스로 사용해서 key-value 값을 관리하는 것은 실수할 여지도 많고 타입 안전하지도 않다. 따라서 내부적으로 ordinal을 활용해서 key-value를 관리해주는 EnumMap을 사용하는 것이 훨씬 타입 안전하고 실수의 여지도 줄여준다.

## Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

열거 타입은 확장할 수 없다는 단점을 가지고 있다. 따라서 클라이언트가 값을 확장할 수 있도록 인터페이스를 구현한 열거형을 사용하는 것이 좋다. 열거 타입이 해당 인터페이스를 구현한 다음 클라이언트에게 제공하는 메소드의 파라미터에서는 인터페이스로 값을 주고 받도록 한다. 이렇게 되면 클라이언트가 인터페이스를 확장한 자기 자신만의 열거 타입을 만들 수 있고 확장성이 생긴다.

추후에 확장이 필요한 열거 타입이라면 인터페이스를 사용하도록 하자

## Item 39. 명명 패턴보다 애너테이션을 사용하라

애너테이션은 메타 정보를 표현할 수 있는 인터페이스의 종류 중 하나다. 애너테이션이 없을 때는 메소드 이름, 변수 이름 등에 규칙을 사용한 명명 패턴을 이용해서 메타 정보를 얻었는데 이 방식은 실수의 여지도 많고 타입 안전하지도 않다.

따라서 명명패턴 보다는 애너테이션을 사용하는 것이 훨씬 안전하다.

## Item 40. @Override 애너테이션을 일관되게 사용하라

@Override 애너테이션을 사용하면 상위 클래스의 메소드나 인터페이스의 메소드를 재정의할 때 메소드 시그니처 일부분을 실수로 다르게 만들어서 오류가 발생하는 것을 방지해준다. 따라서 메소드를 재정의할 때는 항상 일관되게 @Override 애너테이션을 사용하는 것이 좋다.

## Item 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

마커 인터페이스는 아무런 메소드가 없이 메타 정보만 제공하는 인터페이스이다. 애너테이션을 이용해서 메타 정보를 표현할 수 있지만 애너테이션은 타입으로 표현할 수 없다는 단점이 있다. 따라서 메타 정보를 타입으로 표현하고 싶을 때는 마커 인터페이스를 사용해야한다. 또한 마커 인터페이스는 적용대상을 클래스로 한정지을 수 있다.

애너테이션과 마커 인터페이스를 둘다 사용할 수 있을 때도 있는데 이 떄는 상황에 따라 적절히 선택한다. 만약 현재 애너테이션 기반의 프레임워크를 이용하고 있다면 일관성있게 애너테이션을 사용하는 것이 좋다.
