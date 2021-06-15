---
title: "Effective Java 객체의 생성과 파괴"
search: true
categories: 
  - Java
last_modified_at: 2021-06-07T21:00:00
---

**이 글은 프로그래밍인사이트의 Effective Java 3/E를 읽고 요약 정리한 글입니다.**

**책 내용이 훨씬 정확하니 책을 사서 보세요**

**문제 시에 글을 내리겠습니다**

# 이펙티브 자바 - 객체의 생성과 파괴

자바에서 객체는 프로그램을 작성할 때 중요한 요소 중 하나입니다. 객체를 사용하기 위해서는 생성을 해야하고 사용한 객체는 메모리를 절약하기 위해 정리해줘야 합니다. jvm에서 제공해주는 가비지 컬렉터 덕분에 쉽게 객체를 생성해서 사용하고 정리할 수 있지만 올바르게 코드를 작성하지 않으면 예상치 못한 버그나 메모리 문제가 발생할 수 있습니다.

자바 코드를 더 **명료하고 단순하게** 만들 수 있도록 상황별로 적합한 객체의 생성과 파괴 방법을 알아보겠습니다.

> 이 책의 규칙 대부분은 아주 핵심적인 기본 원칙 몇 개에서 파생된다. 바로 명료성(clarity)과 단순성(simplicity)이다. - 1장 들어가기

## 생성자 대신 정적 팩토리 메소드를 고려하라

생성자 대신 정적 팩토리 메소드를 사용하면 좀 더 가독성이 좋고 의미가 분명한 코드를 작성할 수 있습니다.

정적 팩토리 메소드를 사용할 때 장점과 단점을 정리하면 다음과 같습니다.

**정적 팩토리 메소드 장점**

1. 좀 더 의미있는 이름을 사용할 수 있습니다. \
-> 생성되는 객체의 의미를 좀 더 명확히 표현하여 읽기 좋은 코드를 만들 수 있습니다.

2. 호출될 때 마다 인스턴스를 새로 생성하지 않아도 됩니다. \
-> 인스턴스 생성을 통제하여 싱글톤을 반환하거나 캐싱을 해서 성능을 향상시킬 수 있습니다.

3. 하위 타입의 객체를 반환할 수 있습니다. \
-> 객체의 생성이 추상화 돼서 반환되는 객체가 다른 하위 타입 객체로 변경돼도 클라이언트의 코드에 영향을 주지 않습니다. \
-> OCP(확장에 열려있고 변경에 닫힌 코드)를 유지할 수 있습니다. 

4. 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있습니다. \
-> 3번 장점과 유사합니다. 입력 매개변수에 따라 다른 하위 타입의 객체를 반환해서 유연하게 기능을 제공할 수 있습니다. \
-> 클라이언트는 구현체를 신경쓰지 않고 편리하게 사용할 수 있습니다.

5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 됩니다. \
-> JDBC 등의 서비스 제공 프레임워크에서 사용하는 방식입니다. 작성 시점에는 반환할 객체가 없어도 추후에 구현체를 등록해서 반환할 수 있도록 작성할 수 있습니다. JDBC에서 등록된 Driver에 따라 생성되는 Connection 객체가 달라지는 것이 이 방식을 이용해서 구현됐습니다.

**정적 팩토리 메소드 단점**

1. public이나 protected 생성자 없이 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다. \
-> 단점이기는 하지만 상속보다는 합성(composition)을 사용하라는 원칙을 따르면 무시할 수 있는 단점입니다.

2. 정적 팩토리 메소드는 문서만 보고 찾기 힘들다.
-> 다른 정적 메소드와 섞여있어서 클라이언트가 찾기 힘들 수 있습니다.

위에서 살펴봤듯이 정적 팩토리 메소드를 사용하면 좀 더 명확하고 유지보수에 좋은 코드를 작성할 수 있습니다.

개인적으로 대부분의 상황에서 정적 팩토리 메소드를 쓰는 것이 의미가 명확하고 추상화를 더 잘 지원하므로 유지보수에 더 좋다고 생각합니다.

## 생성자에 매개변수가 많다면 빌더를 고려하라

생성자에 선택적(optional) 매개변수가 많다면 빌더 패턴을 사용하는 것이 더 좋습니다.

다음 예시를 보면 선택적 매개변수가 추가될 때 마다 정적 팩토리 메소드 또는 생성자를 추가해줘야 하는 것을 볼 수 있습니다.

```
public class Pizza {
  private String name;
  private String dough;
  private String sauce;

  public static Pizza of(String name) { ... };
  public static Pizza of(String name, String dough) { ... };
  public static Pizza of(String name, String dough, String sauce) { ... };
}
```

선택적 매개변수가 두개만 돼도 정적 팩토리 메소드가 많이 생기고 유지보수 하기가 어렵습니다.

빌더 패턴을 사용하면 유연하게 선택적 파라미터를 관리할 수 있습니다.

```
public class Pizza {
  private String name;
  private String dough;
  private String sauce;

  public static Builder builder(String name) {
    return new Builder(name);
  }

  public static class Builder {
    private Pizza pizza;

    private Builder(String name) {
      pizza = new Pizza(name);
    }

    public Builder dough(String dough) {
      pizza.dough = doguh;
    }

    public Builder sauce(String sauce) {
      pizza.sauce = sauce;
    }

    public Pizza build() {
      return pizza;
    }
  }
}

public class Main {
  public static void main(String[] args) {
    Pizza pizza = Pizza.builder()
      .dough("thin")
      .build();
  }
}
```

클래스에 선택적인 매개변수가 많으면 많을수록 생성자나 정적 팩토리 메소드로 관리하기 힘드므로 빌더 패턴을 사용하는 것이 더 유연하고 관리하기에 좋습니다.

## private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글톤 객체는 다음과 같이 작성합니다. 정적 팩토리 메소드로 인스턴스를 반환하게 되면 API 변경 없이 동작을 쉽게 변경할 수 있습니다.
또한 제너릭 싱글턴 팩토리 메소드로 변경하거나 Singleton::getInstance처럼 Supplier 형태로 사용할 수 있습니다.

```
public class Singleton {
  private static final INSTANCE = new Singleton();
  public static Singleton getInstance() { return INSTANCE; }
  private Singleton() {}
}
```

반드시 생성자를 private으로 만들어서 객체가 하나만 생성되는 것을 보장해야 합니다.

다만 reflection API를 사용하면 생성자를 private으로 만들어도 객체를 생성할 수 있습니다.
또한 Serializable을 구현한 싱글톤은 역직렬화 될 때 새로운 인스턴스가 생길 수 있습니다. (readResolve를 재정의하면 방지할 수 있음)

따라서 다음과 같이 열거 타입을 싱글톤으로 사용하는 것이 가장 안전합니다.
```
public enum Singleton {
  INSTANCE;
}
```

## 인스턴스화를 막으려거든 private 생성자를 사용하라

유틸 클래스처럼 정적 메소드 기능만 있는 클래스는 인스턴스화가 될 필요가 없습니다.

따라서 생성자를 private 접근자로 만드는 것이 안전합니다.
또한 reflection api로 인스턴스가 생성되는 것을 방지하기 위해 생성자에서 exception을 발생시킬 수 있습니다.

```
public class Util {
  private Util() {
    throw new AssertionError();
  }
}
```

## 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

객체의 책임을 분리해서 코드를 작성하다보면 객체가 다른 객체에게 기능을 위임하는 경우가 많습니다. 이 때 다른 객체를 직접 클래스 내에서 생성하지 않고 주입 받는 것이 더 좋습니다.

다음과 같이 무기를 플레이어 클래스에서 직접 생성하면 무기의 코드가 변경되거나 다른 무기로 변경할 때 플레이어의 코드를 수정해야합니다.

```
public class Player {
  private Sword sword;
  
  public Player() {
    sword = new Sword();
  }

  public void attack() {
    sword.attack();
  }
}
```

무기를 인터페이스로 분리하고 의존 객체를 주입해주면 무기가 변경되거나 혹은 플레이어의 테스트 코드를 작성할 때 테스트 무기 의존 객체를 주입하여 좀 더 쉽게 작성할 수 있습니다.

```
public class Player {
  private Weapon weapon;

  public Player(Weapon weapon) {
    this.weapon = weapon;
  }

  public void attack() {
    weapon.attack();
  }
}
```

의존 객체 주입을 사용하면 클래스 간의 결합도를 줄이고 책임에 집중할 수 있는 유연한 코드를 작성하도록 도와줍니다.

## 불필요한 객체 생성을 피하라

같은 기능을 하는 객체를 매번 생성하는 것 보다는 재사용을 하는 것이 더 좋을 때가 많습니다. 특히 불변 객체는 언제나 재사용이 가능합니다.

당연한 얘기지만 String 객체를 생성자 형태(new String("this is string")로 생성하는 것 보다는 표현식 형태("this is string"로 생성하는 것이 더 좋습니다.

표현식 형태로 사용하면 불변 객체로 상수 풀에서 관리 되기 때문에 메모리가 절약됩니다.

마찬가지로 Boolean 생성자를 사용하는 것 보다는 Boolean.valueOf 정적 팩토리 메소드를 사용하면 불필요한 중복 객체 생성을 피할 수 있습니다.

가장 주의해야할 점은 무거운 객체의 중복 생성을 피하는 것입니다.

예를들어 "string".matches는 매번 무거운 Pattern 객체를 생성하기 때문에 지양하고 Pattern 객체를 한번만 생성해서 사용하는 것이 더 좋습니다.

마지막으로 오토박싱을 주의해야합니다. 다음 코드는 반복문에서 Long 객체가 매번 생성되므로 매우 비효율적인 코드가 됩니다. sum의 자료형을 Long에서 long으로만 바꿔줘도 효율적으로 사용할 수 있습니다.

```
Long sum = 0L;
for (long i = 0;i <= 10000;i++) {
  sum += i;
}
```

## 다 쓴 객체 참조를 해제해라

자바는 메모리를 자동으로 관리해주기 때문에 메모리 참조에 대해 신경쓰지 않는 경우가 많습니다.

하지만 개발자가 참조를 제대로 관리해주지 않으면 메모리 누수나 예상치 못한 버그가 발생할 수 있습니다.

다 쓴 객체는 null로 초기화 해주거나 참조될 수 없도록 제거해주는 것이 좋습니다.

특히 캐싱처리를 할 때 이런 문제가 발생할 수 있는데 참조가 있어도 gc에서 처리해주는 WeakMap을 사용하거나 캐시의 개수를 제한할 수 있는 LRU 캐시를 사용하면 메모리 누수를 방지할 수 있습니다.

## finalizer와 cleaner 사용을 피하라

finalizer와 cleaner는 jvm에 의해 호출이 되지 않을 수도 있고 언제 호출이 될지 예측할 수 없습니다.

다음과 같은 상황을 제외하면 가급적 사용을 피하는 것이 좋습니다.

사용자가 호출해줘야하는 close같은 해제 메소드를 실수로 호출을 안하는 경우에는 finalizer나 cleaner를 사용해서 해제해주는 것이 안해주는 것 보다는 좋습니다.

## try-finally 보다는 try-with-resources를 사용하라

finally 구문에서 리소스를 정리하는 것은 실수로 누락하는 경우가 발생하거나 finally 구문에서 에러가 발생해서 원본 에러를 감춰버리는 문제가 발생할 수 있습니다.

따라서 항상 try-with-resources 사용하는 것이 좋습니다.