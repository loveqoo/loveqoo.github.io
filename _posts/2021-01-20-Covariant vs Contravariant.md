
# Variance

`변성` 이라는 뜻.

다음과 같은 상황일 때, 

- `A`타입, `B`타입이 있다.
- `A ≤ B`의 의미는 
	- `A`가 `B`의 서브타입이라는 표현이다. 
	- `A`보다 `B`가 더 많은 타입을 포괄한다는 뜻이다. (`B` 가 부모의 개념)
- `F`가 어떤 타입을 품고 새로운 타입으로 변환된다. (리스트와 같은 컨테이너)
- `A ≤ B` 이고, `F(A) ≤ F(B)` 이면 공변성이다.
- `A ≤ B` 이고, `F(B) ≤ F(A)` 이면 반공변성이다.


자바는 `Use-site variance` 라고 해서, 메소드 선언 시점에만 변성을 정의 할 수 있고 클래스 선언 시점에서는 정의 할 수 없다.

하지만 코틀린과 스칼라는 클래스 선언 시점에도 정의할 수 있다. (`Declaration-Site Variance`)

# 이거 왜 누가 만들었나

유명한 마틴 오더스키 외 그의 동료들이 팀([Team GJ](http://lampwww.epfl.ch/gj/))을 꾸려서 만들기 시작했다. 

하지만 그 당시에는 지금처럼 복잡하지 않았다. 
- [GJ의 튜토리얼](http://lampwww.epfl.ch/gj/Documents/gj-tutorial.pdf)의 Bound, Subtyping 섹션을 보면 알 수 있다.

그리고 JDK 1.5에 반영되었다.

# 변성을 정의하면 읽기만 할 수도 있고 쓰기만 할 수도 있다.

## 읽기 전에 알아두어야 할 사항

1. `Double`은 `Number`의 서브 클래스이다.
> public final class Double extends Number implements Comparable<Double>
2. `Integer`는 `Number`의 서브 클래스이다.
> public final class Integer extends Number implements Comparable<Integer>
3. `Double`은 `Integer`의 상위 클래스가 아니다. 그러므로 서로 변환 할 수 없다.


## Extends 를 파헤쳐보자

``` java
List<? extends Number> foo3 = new ArrayList<Number>();  // (1)
List<? extends Number> foo3 = new ArrayList<Integer>(); // (2)
List<? extends Number> foo3 = new ArrayList<Double>();  // (3)
```


### 읽을 때

위의 코드를 보고, 어떤 타입의 엘리먼트를 `List foo3`에서 읽을 때 보장할 수 있을지 생각해보자.

- `Number` 타입을 기대하고 엘리먼트를 읽을 수 있다.
	- 3개의 리스트의 엘리먼트는 `Number` 인스턴스 이고, `Number` 하위 클래스의 인스턴스이기 때문
- `Integer` 타입을 기대하고 엘리먼트를 읽을 수 없다.
	- (3)에서 `Integer`라고 기대하고 엘리먼트를 하나 읽을 때, `Double`타입의 엘리먼트가 나오므로 `Integer` 타입이 아니다.
- `Double` 타입을 기대하고 엘리먼트를 읽을 수 없다.
	- (2)에서 `Double`이라고 기대하고 엘리먼트를 하나 읽을 때, `Integer`타입의 엘리먼트가 나오므로  `Double` 타입이 다르다.

### 쓸 때

위의 코드를 보고, 어떤 타입을 추가할 때 문법적으로 문제가 없는지 생각해보자.

- `Number` 타입의 엘리먼트를 넣을 수 없다.
	- `Double` 타입의 엘리먼트를 `Number` 타입으로 보고, 그걸 (2)에 넣으면 잘못된 타입의 엘리먼트를 넣게 된다.
- `Double` 타입의 엘리먼트를 넣을 수 없다.
	- `Interge` 타입의 엘리먼트를 `Number` 타입으로 보고, 그걸 (3)에 넣으면 잘못된 타입의 엘리먼트를 넣게 된다.
- `Integer` 타입의 엘리먼트를 넣을 수 없다.
	- `Double` 타입의 엘리먼트를 `Number` 타입으로 보고, 그걸 (2)에 넣으면 잘못된 타입의 엘리먼트를 넣게 된다.

### 결론

`List<? extends T>`는 

- `T` 타입이나 `T`의 서브 클래스의 인스턴스만 읽을 수 있다.
- 어떤 타입의 오브젝트도 넣을 수 없다.
	- 어떤 `List` 구현체가 할당 되었는지 모르기 때문이다.

그렇기 때문에 `extends` 제한자는 

- Read Only 속성을 줄 수 있다.
- `Upper Bound` (상한 제한) 속성이 있다고 말한다.
	- 위의 예제에서 가장 높은 타입인 `Number` 타입으로 변성을 정의하면, 그 하위 타입의 리스트를 할당 할 수 있다.
- Immutable 클래스 설계에 쓰일 수 있다. 
	- [코틀린의 AbstractCollection](https://github.com/JetBrains/kotlin/blob/eed0f50c5d08a34cf657df84826890e9a417b3d0/libraries/stdlib/src/kotlin/collections/AbstractCollection.kt#L15)
	- 참고로 코틀린에서는 `out` 선언으로 표현한다.
- 리스트의 엘리먼트의 상속 관계가 그대로 리스트까지 전이되는 결과를 가져오므로 `공변적`이라고 말한다.
	- `Interger ≤ Number` 일 때, `List<Interger> ≤ List<Number>` 이다.
	- 왜 그런지는 직접 타입이 누가 더 많은 타입을 포괄하는지 확인하면 된다.

``` java

// Integer 의 타입은 1개의 타입을 수용할 수 있다. 자기 자신.
Integer foo2 = new Number(0); // Error
Integer foo2 = new Integer(0);
Integer foo2 = new Double(0); // Error

// Number 의 타입은 3개의 타입을 수용할 수 있다.
Number foo2 = new Number(0);
Number foo2 = new Integer(0);
Number foo2 = new Double(0);

// List<Integer> 타입은 1개의 타입을 수용할 수 있다. 자기 자신
List<? extends Integer> foo3 = new ArrayList<Number>(); // Error
List<? extends Integer> foo3 = new ArrayList<Integer>();
List<? extends Integer> foo3 = new ArrayList<Double>(); // Error

// List<Number>의 타입도 마찬가지로 3개의 타입을 포괄한다.
List<? extends Number> foo3 = new ArrayList<Number>();
List<? extends Number> foo3 = new ArrayList<Integer>();
List<? extends Number> foo3 = new ArrayList<Double>();

```

## Super 를 파헤쳐보자

``` java
List<? super Integer> foo3 = new ArrayList<Integer>(); // (1)
List<? super Integer> foo3 = new ArrayList<Number>();  // (2)
List<? super Integer> foo3 = new ArrayList<Object>();  // (3)
```

### 읽을 때

위의 코드를 보고, 어떤 타입의 엘리먼트를 `List foo3`에서 읽을 때 보장할 수 있을지 생각해보자.

- `Integer` 타입을 기대하고 엘리먼트를 읽을 수 없다.
	- (2), (3)에 `Integer` 타입과 연관이 없는 인스턴스가 들어가지 않았다고 보장할 수 없다. (Ex: `Double` 타입)
- `Number` 타입을 기대하고 엘리먼트를 읽을 수 없다.
	- (3)에 `Number` 타입과 연관이 없는 인스턴스가 들어가지 않았다고 보장할 수 없다.
- `Object` 타입을 기대하고 읽을 수만 있다. 
	- 하지만 읽은 엘리먼트가 `Object`의 어떤 서브 클래스의 인스턴스인지 모른다.

### 쓸 때

위의 코드를 보고, 어떤 타입을 추가할 때 문법적으로 문제가 없는지 생각해보자.

- `Integer` 타입의 엘리먼트를 넣을 수 있다.
- `Integer`의 서브 클래스 인스턴스를 엘리먼트로 넣을 수 있다.
- `Double` 타입의 엘리먼트를 넣을 수 없다.
	- (1) 때문
- `Number` 타입의 엘리먼트를 넣을 수 없다.
	- (1) 때문
- 'Object' 타입의 엘리먼트를 넣을 수 없다.
	- (1) 때문

### 결론

`List <? super T>` 는

- 어떤 타입으로도 읽을 수 없다.
- `T` 타입 또는 `T` 타입의 서브 클래스의 인스턴스만 넣을 수 있다.

그렇기 때문에 `super` 제한자는

- `Lower Bound` (하한 제한) 속성이 있다고 말한다.
	- 위의 예제에서 `Integer` 타입으로 변성을 설정하면, 그 상위 타입의 리스트만 할당할 수 있었다.
	- 리스트의 엘리먼트의 상속 관계가 반대로 리스트에 전이되므로 `반공변적`이라고 말한다.
	- `Interger ≤ Number` 일 때, `List<Number> ≤ List<Interger>` 이다.

``` java

// Integer 의 타입은 1개의 타입을 수용할 수 있다. 자기 자신.
Integer foo2 = new Number(0); // Error
Integer foo2 = new Integer(0);
Integer foo2 = new Double(0); // Error

// Number 의 타입은 3개의 타입을 수용할 수 있다.
Number foo2 = new Number(0);
Number foo2 = new Integer(0);
Number foo2 = new Double(0);

// List<Integer>의 타입은 3개의 타입을 포괄한다.
List<? super Integer> foo3 = new ArrayList<Integer>(); 
List<? super Integer> foo3 = new ArrayList<Number>();
List<? super Integer> foo3 = new ArrayList<Object>();

// List<Number>의 타입은 2개의 타입을 포괄한다.
List<? super Number> foo3 = new ArrayList<Integer>(); // Error
List<? super Number> foo3 = new ArrayList<Number>();
List<? super Number> foo3 = new ArrayList<Object>();

```

### PECS ?

> PECS: Producer Extends, Consumer Super

#### Producer Extends

만약 `Integer` 타입의 엘리먼트를 읽기만을 위해 `List`가 필요하다면, 반드시 `List<? extends Integer>`로 선언해야 한다. 

이 리스트는 어떤 엘리먼트도 추가할 수 없다.

#### Consumer Super

만약 `Integer` 타입의 엘리먼트를 넣기만을 위한 `List`가 필요하다면, 반드시 `List<? super Integer>`로 선언해야 한다.

`Integer` 타입 또는 `Integer` 타입의 서브 클래스의 인스턴스만 넣을 수 있다.

리스트에서 엘리먼트를 하나 읽을 수 있다. 하지만 그 엘리먼트가 어떤 타입인지는 보장할 수 없다.

#### 둘 다 아닌 경우

와일드카드를 쓸 수 없다. `List<T>` 로 정의한다.


#### 실제 코드를 보자

``` java
public interface Function<T, R> {

  R apply(T t);
  
  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
}

```


``` kotlin
public interface Function1<in P1, out R> : Function<R> {
    public operator fun invoke(p1: P1): R
}

```


출처 및 참고: 

1. [https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java](https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java)
2. [https://schneide.blog/2015/05/11/declaration-site-and-use-site-variance-explained/](https://schneide.blog/2015/05/11/declaration-site-and-use-site-variance-explained/)
3. [https://iosroid.tistory.com/70](https://iosroid.tistory.com/70)

