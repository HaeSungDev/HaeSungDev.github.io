---
title: "Effective Java 제네릭"
search: true
categories: 
  - Java
last_modified_at: 2021-06-20T15:00:00
---

**이 글은 프로그래밍인사이트의 Effective Java 3/E를 읽고 요약 정리한 글입니다.**

**책 내용이 훨씬 정확하니 책을 사서 보세요**

**문제 시에 글을 내리겠습니다**

# 이펙티브 자바 - 제네릭

제네릭을 사용하면 타입을 파라미터화 해서 컴파일타임에 컬렉션에 어떤 타입의 객체가 들어가는지 알 수 있어서 안전하고 명확한 프로그램을 작성할 수 있다. 이번 장에서는 이펙티브 자바의 제네릭 챕터를 읽고 정리했다.

## Item 26. Raw 타입은 사용하지 말라

타입 매개변수를 사용하는 클래스, 인터페이스를 제네릭 클래스, 제네릭 인터페이스라고 한다. 예를들어 List<E>는 E라는 타입 매개변수를 받는 제네릭 클래스이다. 제네릭 클래스는 Raw type을 함께 정의하는데 List<E>의 Raw type은 List이다. 타입 매개변수 정보가 없어진 채로 동작한다.

Raw type의 제네릭 클래스는 타입 정보를 무시해서 다른 타입의 클래스와 함께 동작할 수 있다. 예를 들어 Raw type 제네릭 클래스 List에는 아무 객체나 넣어도 동작한다. 하지만 아이템을 List로 부터 가져올 때는 어떤 타입인지 알 수 없다.

**따라서 Raw type의 제네릭 클래스 사용은 하면 안된다**

제네릭 타입 정보를 사용하면 List에 넣을 때나 List에서 가져올 때 모두 컴파일 타임에 확인이 가능하다. 따라서 훨씬 안전하고 명확하게 프로그램을 만들 수 있다. 그렇다면 Raw type은 왜 있을까? 자바 5 이전에는 Generic 없이 Raw type만 사용했기 때문에 하위 호환성을 지키기 위해 Raw type을 여전히 제공한다.

Raw type을 예외적으로 사용해야 하는 경우도 있다.

1. Class literal에는 제네릭 타입을 사용할 수 없으므로 Raw type을 사용해야한다.
2. 런타임에는 제네릭 정보가 지워지므로 instanceof 연산자를 사용할 때는 Raw type을 사용해야한다.

이 외에는 Raw type을 사용하지 말아야한다.

여러가지 종류의 원소를 담고 싶을 때는 List<Object>를 타입 매개변수를 신경쓰고 싶지 않을 떄는 List<?>를 사용하면 된다.

List<?>와 같이 ?는 어떤 타입이든 신경쓰지 않는다는 비한정적 와일드카드 타입이다. 어떤 타입인지 모르기 때문에 null을 제외한 새로운 원소를 넣을수는 없다.

정리하면 컴파일 타임에 안전한 타입 체크를 위해 Raw 타입 대신 제네릭 타입을 사용하자.

## item 27. 비검사 경고를 제거하라

unchecked 경고는 type이 올바르게 cast 되지 못할 가능성이 있을 때 발생한다. 따라서 코드 안정성을 위해 해당 검사는 무시하면 안된다. 하지만 경고를 없애는 것이 불가능하고 타입 안정성이 확실하게 보장된다고 생각하면 @SuppressWarnings("unchecked") 어노테이션을 사용하자.

@SuppressWarnings 어노테이션을 사용할 때는 반드시 좁은 범위를 지정해야한다. @SuppressWarnings 어노테이션은 선언문이라면 어디든 달 수 있다. 따라서 지역변수로 선언범위를 좁하는 것을 추천한다.

## Item 28. 배열보다는 리스트를 사용하라

배열과 제너릭 타입 리스트에는 중요한 두가지 차이가 있다.

1. 배열은 공변인 반면 리스트는 불공변이다. 배열은 Sub 클래스가 Super 클래스의 하위 타입이면 Sub[]가 Super[]의 하위 타입이 되지만 List<Sub>는 List<Super>의 하위 타입이 될 수 없다.

2. 배열은 실체화된다. 따라서 런타임에 자기가 담기로 한 원소의 타입을 인지하고 확인한다. 따라서 `Object[] arr = new Long[1];`인 배열이 있을 때 `arr[0] = "타입이 다르다";` 이 코드는 컴파일 타임은 통과하지만 런타임에서 ArrayStoreException 예외가 발생한다. 반면 List는 런타임에는 타입 정보가 소거돼서 어떤 타입인지 알수 없지만 컴파일 타임에 확인한다.

컴파일 타임에 타입을 확인하는 편이 좀 더 안전한 코드를 작성할 수 있으므로  리스트를 사용하는 것이 좋다.

또한 제너릭 타입은 배열로 만들 수 없다. 제너릭 타입의 배열을 만들 수 있게하면 런타임에 ClassCastException이 발생할 수 있으므로 제너릭 타입 시스템의 취지와 어긋난다. 마찬가지로 제너릭 타입은 가변인수로 사용할 수 없다. 가변인수는 메소드 호출마다 배열을 생성하기 때문이다.

정리하면 배열보다는 리스트 타입을 사용하는 것이 컴파일 타임에 타입 체크가 되므로 좀더 안전한 코드를 작성할 수 있다. 따라서 리스트를 사용하자

## Item 29. 이왕이면 제네릭 타입으로 만들라

제네릭 타입을 새로 만드는 일은 쉽지 않지만 한번 배워두면 매우 유용하다. 제너릭 타입을 이용하면 클라이언트가 형변환 할 필요없이 안전하게 사용할 수 있다.

Item 28과는 상반되는 내용이긴 하지만 제너릭 타입을 만들 때는 배열을 사용해야할 수 도 있다. 따라서 제너릭 타입 내부적으로 명시적으로 형변환을 사용해야할 수도 있다. 이 경우에는 ClassCastExceptino이 발생하지 않도록 프로그래머가 주의해서 작성해야한다.

정리하면 제너릭 타입을 잘 만들면 클라이언트가 형변환 할 필요없이 컴파일 타임에 안전하게 사용할 수 있는 클래스를 제공할 수 있다.

## Item 30. 이왕이면 제네릭 메소드로 만들라

메소드를 제너릭 타입으로 만들면 클라이언트가 형변환 없이 안전하게 메소드를 사용할 수 있다.

불변 객체를 여러 타입으로 활용할 수 있게 만들때도 제너릭 메소드는 유용하다. 예를들어 항등함수 불변객체가 있을 때 이를 여러 타입에 사용할 수 있게하려면 다음과 같이하면 된다.

```
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```

항등함수는 자기 자신을 반환하므로 타입변환은 항상 성공한다. 따라서 컴파일러의 경고를 무시하기 위해 SuppressWarnings 어노테이션을 사용해도 안전하다. 이 코드는 클라이언트가 사용할 때 형변환 없이 안정적으로 사용할 수 있다.

드물게 사용되는 편이긴 하지만 재귀적 타입 한정을 사용한 제너릭 메소드도 있다. 주로 Comparable 인터페이스와 함께 쓰인다.

```
public static <E extends Comparable<E>> E max(Collection<E> c) { ... }
```

자기 자신 타입이 포함된 Comparable<E> 타입을 상속하는 객체의 컬렉션만 메소드로 넘길 수 있다. 컴파일 오류나 경고 없이 사용할 수 있다.

정리하면 제너릭 타입 메소드를 사용하면 형변환 없이 메소드를 안정적으로 사용할 수 있다.

## Item 31. 한정적 와일드카드를 사용해 API 유연성을 높여라

자바에서 제너릭 타입은 불공변이다. 따라서 `List<String>`과 `List<Object>`는 서로 상위타입도 하위타입도 아니다. 하지만 때로는 공변또는 반공변이 필요할 때도 있다. 이때 사용할 수 있는 것이 한정적 와일드카드이다.

다음 Stack 클래스가 있다고 했을 때
```
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```

여러개의 원소 전체를 스택에 넣는 메소드를 추가한다고 해보자
```
public void pushAll(Iterable<E> src) {
  for (E e : src) {
    push(e);
  }
}
```

이 메소드는 컴파일은 되고 Iterable<E> 타입과 일치하면 잘 작동하지만 문제가 있다.

```
Stack<Number> stack = new Stack<>();
Iterable<Integer> integers = ...;
stack.pushAll(integers);
```
Integer는 number의 하위 클래스이지만 제너릭은 불공변이므로 위 코드는 오류 메세지가 뜬다.

이런 상황을 한정적 매개변수를 사용해서 해결할 수 있다.

```
public void pushAll(Iterable<? extends E> src) { ... };
```

위 코드처럼 한정적 메소드를 사용하면 E의 하위클래스인 Iterable 타입을 인수로 사용할 수 있다.

마찬가지로 모든 원소를 옮겨담는 메소드를 만들때도 한정적 매개변수를 사용할 수 있다.

```
public void popAll(Collection<? super E> dst) {
  while (!isEmpty()) {
    dst.add(pop());
  }
}
```

이번에는 반대로 반공변을 사용했다. dst 컬렉션은 E 타입 또는 E 타입의 상위 클래스가 와야 원소를 추가할 수 있기 떄문이다.

위 예제들처럼 한정적 와일드 카드를 사용하면 유연한 코드 작성이 가능하지만 한정적 와일드 카드를 사용하는 것은 약간 복잡하다. 따라서 다음 공식을 사용하자

PECS: producer-extends, consumer-super

생산자는 extends를 소비자는 super 한정적 와일드 카드를 사용하면 된다. 위 예제코드에서도 Iterable은 크게보면 값을 제공해주는 입장으로 생산자라서 extends를 사용했고 Collection은 크게보면 값을 소비하는 입장이라 super를 사용했다.

PECS 공식을 사용하면 쉽고 빠르게 어떤 한정적 와일드 카드를 사용할 지 판단할 수 있다.

## Item 32. 제네릭과 가변인수를 함께 쓸 떄는 신중하라

가변인수 메소드는 가변인수를 담기 위한 배열을 생성하고 노출한다는 단점이 있다. 따라서 가변인수가 제네릭 타입과 같이 쓰이면 컴파일 경고가 발생한다. 매개변수화 타입을 가변인수와 사용하면 다음과 같은 문제가 발생한다.

```
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // heap pollution
    String s = stringLists[0].get(0); // ClassCastException
}
```

위 코드를 보면 `List<String>` 타입의 배열에 `List<integer>` 타입의 원소가 들어갔다. 따라서 힙이 오염되고 첫번째 배열의 원소에서 값을 가져올 때 ClassCastException이 발생한다.

이처럼 배열과 제네릭은 서로 호환되지 않기 때문에 메소드 작성자가 주의해서 작성해야한다. 다음과 같은 규칙을 사용하면 제네릭 가변인자 메소드를 안전하게 작성할 수 있다.

1. varargs 매개변수 배열에 아무것도 저장하지 않는다. Heap 오염을 방지할 수 있다.
2. 가변인자 매개변수 배열을 신뢰할 수 없는 코드에 노출하면 안된다.

위 규칙을 지키고 @SafeVarargs 어노테이션을 달아주면 사용자는 컴파일 경고 없이 안전하게 제너릭 가변인자 메소드를 사용할 수 있다.

또 다른 대안은 가변인자 매개변수 대신 List를 사용하는 것이다. 자바 List 클래스가 제공해주는 List.of 메소드를 사용하면 가변인자를 직접 다루지 않고 안전하게 메소드를 사용할 수 있다.

## Item 33. 타입 안전 이종 컨테이너를 고려하라

제네릭을 사용할 때 매개변수화할 수 있는 타입의 개수가 제한된다. 하지만 좀 더 유연하게 매개변수화 된 타입의 개수를 사용해야할 때도 있다. 이럴때는 Class 객체를 타입 토큰으로 사용하면 된다.

예를들어 다음과 같이 Favorites 클래스를 만들 수 있다.
```
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}
```

위 클래스는 Class 객체를 토큰으로 사용해서 값을 저장하고 Class 객체 토큰으로 값을 가져올 수 있다.

```
Favorites f = new Favorites();

f.putFavorite(String.class, "Java");
f.putFavorite(Integer.class, 0xcafebebe);

String favoriteString = f.getFavorite(String.class);
int favoriteInteger = f.getFavorite(Integer.class);
```

위 Favorites 클래스처럼 매개변수화된 타입을 키로 제공하는 방식을 **타입 안전 이종 컨테이너 패턴**이라고 한다. Favorite 인스턴스를 타입 안전하게 사용할 수 있다.

Favoites 클래스는 생각보다 간단하게 만들 수 있다.

```
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<>();

  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
  }

  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```

위 Favorites 클래스는 두가지 제약이 있는데 첫번째는 클라이언트가 Raw Type 객체를 넘기면 타입 안정성이 깨진다

```
f.putFavorite((Class)Integer.class, "not integer");
```

이 코드는 에러가 발생하는데 다음과 putFavorite이 호출될때 오류가 발생하도록해서 막을수 있다.
```
public <T> void putFavorite(Class<T> type, T instance) {
  favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

두번째 제약은 제네릭 타입을 타입 토큰으로 사용할 수 없다는 점이다. 이 제약에 대한 완벽한 우회는 없다.

정리하면 타입 안정 이종 컨테이너를 사용하면 여러개의 매개변수화 된 타입을 사용해서 타입 안정성이 있게 값을 저장하고 제공할 수 있다.