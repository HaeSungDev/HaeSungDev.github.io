---
title: "Effective Java 모든 객체의 공통 메소드"
search: true
categories: 
  - Java
last_modified_at: 2021-06-15T21:00:00
---

**이 글은 프로그래밍인사이트의 Effective Java 3/E를 읽고 요약 정리한 글입니다.**

**책 내용이 훨씬 정확하니 책을 사서 보세요**

**문제 시에 글을 내리겠습니다**

# 이펙티브 자바 - 모든 객체의 공통 메소드

자바에서 모든 클래스는 Object를 상속하고 있습니다. Object에는 하위클래스가 상속해서 재정의를 염두해두고 설계된 메소드(equals, hashCode, toString, clone, finalize)들이 있습니다.

이번 장에서는 해당 메소드를 재정의 할 때 주의해야 할 점과 지켜야할 일반 규약들에 대해 알아보겠습니다.

## Item 10. equals는 일반 규약을 지켜 재정의 하라

equals는 재정의하기 쉬워보이지만 생각보다 많은 함정을 가지고 있는 메소드입니다. 따라서 다음 경우를 제외하면 재정의하지 않는게 오히려 좋습니다.

* 각 인스턴스가 본질적으로 고유할 때
  * 값을 표현하는 객체가 아닌 동작하는 개체를 표현하는 클래스인 경우. ex) Thread, Service 클래스, 무상태 클래스 등
* 논리적 동치성을 검사할 일이 없을 때
  * 애초에 동치성을 검사할 일이 없는 경우에는 재정의 하지 않는 것이 더 낫습니다.
* 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어 맞을 때
  * 상위 클래스의 equals를 그대로 쓰는 것이 더 부작용이 적고 좋습니다.
* 클래스가 private이거나 package-private이고 equals 메소드를 호출할 일이 없다.
  * 호출할 일이 없는데 구현하는 것은 낭비입니다.

그렇다면 언제 재정의 해야할까요?

* 객체의 주소가 아닌 객체가 지니고 있는 값의 논리적인 동치성을 확인할 때
  * 예를 들어, 전화번호라는 객체가 다음과 같이 두번 생성 됐다고 가정했을때 두 객체는 서로 다른 메모리 주소에 저장되지만 논리적으로 봤을 때는 동일한 값을 가지고 있습니다. 따라서 다음과 같은 경우에는 equals 메소드를 재정의 해주는게 좋습니다.
    ```
    PhoneNumber phoneNumber1 = new PhoneNumber("01000000000");
    PhoneNumber phoneNumber2 = new PhoneNumber("01000000000");
    ```
  * 하지만 논리적 동치성 비교가 필요하더라도 동일한 값을 가진 객체가 여러개 만들어지지 않고 단 하나만 만들어지는 것이 보장된다면 굳이 equals 메소드를 재정의할 필요는 없습니다. (항상 같은 메모리의 주소를 가진 객체이므로)

이제 equals 메소드를 언제 재정의해야하고 하지 말아야하는지 알았으니 올바르게 재정의하는 법을 알아보도록 하겠습니다.
> [equals 일반 규약](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object))

* reflexivity(반사성): null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true이다.
* symmetric(대칭성): null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true이다
* transitivity(추이성): null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y)가 true, y.equals(z)가 true이면 x.equals(z)도 true이다
* consistency(일관성): null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환해야한다.
* null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false이다.

많은 라이브러리들이 Object가 위 일반 규약들을 지킨다고 가정하고 개발되었기 때문에 찾기 힘든 버그를 만들지 않으려면 equals 재정의 시 반드시 일반 규약을 지켜야 합니다.

위 규약을 지키면서 주의해야할 점들이 몇가지 알아보겠습니다.

* 하위 클래스에 값을 추가하고 equals 메소드를 재정의하면 일반 규약을 지킬 수 없다. (리스코프 치환 원칙을 무시하지 않는다면)
  * 따라서 값을 추가하고 equals 메소드를 재정의하고 싶으면 Composition을 활용하는 것이 더 좋습니다.
    아래 예제처럼 객체를 생성하고 equals 비교를 하면 대칭성 규약을 지킬 수 없습니다.
    ```
    public class Car {
      private final Color color;

      public Point(Color color) {
        this.color = color;
      }

      @Override
      public boolean equals(Object o) {
        if (!(o instanceof Car)) return false;
        Car c = (Car) o;
        return c.color == color;
      }
    }

    public class Truck extends Car {
      private final int loadWeight;

      public Truck(Color color, int loadWeight) {
        super(color);
        this.loadWeight = loadWeight;
      }

      @Override
      public boolean equals(Object o) {
        if (!(o instanceof Truck)) return false;
        return super.equals(o) && ((Truck) o).loadWeight == loadWeight;
      }
    }
    ```
    ```
    Car car = new Car(Color.RED);
    Truck truck = new Truck(Color.RED, 100);
    
    car.equals(truck); // true;
    truck.equals(car); // false;
    ```
  * 다음과 같이 수정하면 대칭성은 지켜지지만 추이성은 지킬 수 없습니다.
    ```
    // in truck class
    @Override
    public boolean equals(Object o) {
      if (!(o instanceof Truck)) return false;

      if (!(o instance of Truck)) return o.equals(this);

      return super.equals(o) && ((Truck) o).loadWeight == loadWeight;
    }
    ```
    Car car = new Car(Color.RED);
    Truck truck1 = new Truck(Color.RED, 100);
    Truck truck2 = new Truck(Color.RED, 200);
    
    truck1.equals(car); // true
    car.equals(truck2); // true
    truck1.equals(truck2); // false
    ```
  * 결과적으로 위와 같은 경우에는 다음과 같이 Composition을 사용하는 것이 더 좋습니다.
    ```
    public class Truck {
      private final Car car;
      private final int loadWeight;

      public Truck(Car car, int loadWeight) {
        this.car = car;
        this.loadWeight = loadWeight;
      }

      @Override
      public boolean equals(Object o) {
        if (!(o instanceof Truck)) return false;
        Truck t = (Truck) o;
        return t.car.equals(car) && t.loadWeight == loadWeight;
      }
    }
    ```

지금까지 정리한 내용을 바탕으로 equals를 안전하게 구현하는 방법을 알아보겠습니다.

1. == 연산자를 이용해 입력이 자기 자신의 참조인지 확인한다. (성능 최적화용)
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
5. 다 구현했다면 대칭성, 추이성, 일관성이 잘 지켜지는지 단위테스트를 한다.

추가적으로 equals를 재정의했다면 hashCode를 반드시 재정의해야합니다.

equals 메소드 재정의가 생각보다 간단하지 않다고 느껴지실텐데 대부분의 경우에는 복잡한 동치성 비교가 없으므로 AutoValue 같은 라이브러리에 의존하는 것이 좋습니다.

> [AutoValue 참고](https://www.baeldung.com/introduction-to-autovalue)
> [lombok @EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode)

## Item 11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스에서는 모두 hashCode를 재정의해야합니다. 그렇지 않으면 hashCode의 일반 규약에 의존하는 클래스들을 사용할 때 문제가 생깁니다.

> [hashCode 일반 규약](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#hashCode())

* equals 비교에 사용되는 정보가 변경되지 않았다면 애플리케이션이 실행되는 동안 hashCode 메소드는 항상 일관된 값을 반환해야한다.
* equals가 두 객체를 같다고 판단했다면 두 객체는 모두 hashCode도 똑같은 값을 반환해야한다.
* equals가 두 객체를 다르다고 판단했을 떄 hashCode는 다른 값이 아니어도 상관 없다. 단, hashCode가 다른 값을 반환하는 것이 해시테이블 성능에 더 좋다.

올바른 해시코드를 구현하는 방법은 생략하도록 하겠습니다. 대부분의 경우 라이브러리에 의존하는 것이 안전하고 성능에 유리합니다.
아래 라이브러리를 사용하면 hashCode를 쉽게 재정의할 수 있습니다.

> [guava Hashing](https://guava.dev/releases/21.0/api/docs/com/google/common/hash/Hashing.html)
> [lombok @EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode)

마지막으로 라이브러리를 만들고 API를 사용자에게 공개한다면 hashCode가 반환하는 값의 생성 규칙을 알려주지 않는 것이 좋습니다. 사용자가 이 값에 의지하게되면 추후에 계산 방식을 바꾸기 힘듭니다.

## Item 12. toString을 항상 재정의하라

toString을 재정의하지 않으면 객체를 출력하거나 로깅할 때 무의미한 값만 보이므로 항상 재정의해주는 것이 개발 편의성에 큰 도움이 됩니다.

toString을 구현할 때는 그 객체가 가진 주요 정보를 모두 반환하는 것이 좋습니다. 또한 포맷을 정해서 구현하는 것이 더 좋습니다. API 문서에는 해당 포맷을 공개해도 좋고 안해도 좋지만 사용자에게 해당 포맷에 의존하지 말라는 문구를 표시하는 것이 좋습니다. 그래야 추후 변경이 용이합니다.

일반적인 로깅용으로는 자동 생성되는 toString 메소드를 사용해도 좋지만 해당 객체 값의 의미를 유의미하게 파악하지는 못하므로 직접 재정의 해주는 것이 좋습니다.

다음과 같이 사용하는 것을 추천합니다.

1. 기본적으로 자동으로 생성해주는 라이브러리를 사용한다.
2. 좀 더 유의미한 표현을 해주고 싶으면 toString 메소드를 직접 재정의한다.

## Item 13. clone 재정의는 주의해서 진행하라.

clone 재정의는 일반적으로 사용하지 않고 위험하므로 간단히 정리 하겠습니다.

* clone 메소드는 protected 메소드이므로 재정의하지 않으면 사용 불가하다.
* Clonable을 구현한 클래스는 clone 메소드를 public으로 재정의하고 복사한 객체를 반환한다.
* clone 메소드를 구현한 클래스의 하위 클래스를 만들게 되면 문제가 발생할 수 있다. 따라서 final 클래스에서만 사용하자.

결론적으로 clone 메소드를 사용하는 것 보다는 복사 생성자(동일한 타입의 객체를 받아서 생성)나 복사 팩토리를 사용하는 것이 좋습니다.

## Item 14. Comparable을 구현할지 고려하라

자바 컬렉션 라이브러리는 모두 Comparable 인터페이스를 활용해서 구현되어있습니다. 따라서 컬렉션에 객체를 담을 때 Comparable을 구현한 객체를 담는 것이 유용합니다.

Comparable의 compareTo 구현 일반 규약은 equals와 유사하므로 문서로 대체하겠습니다 ㅎ.

>[compareTo 일반 규약](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Comparable.html#compareTo(T))

위 규약을 잘 지켜야 객체들을 올바르게 비교할 수 있습니다.

compareTo 규약을 올바르게 지키지 않으면 비교를 활용하는 TreeSet, TreeMap, Arrays.sort 등을 이용할 수 없습니다. equals와 동일하게 하위 클래스에서 compareTo를 재정의하면 추상화의 이점을 포기하지 않고는 일반 규약을 지킬 수 없으므로 equals와 동일하게 Composition을 활용하는 것이 좋습니다.

compareTo를 구현할 때 몇가지 주의점을 알아보겠습니다.

* 정수 기본 타입 필드를 비교할때 Integer.compare, Short.compare, Long.compare를 사용하는 것이 좋습니다.
* 실수 기본 타입 필드를 비교할때는 반드시 Float.compare, Double.compare를 사용해야합니다.
* hashCode 값의 차이로 비교하는 것은 절대 하면 안됩니다.