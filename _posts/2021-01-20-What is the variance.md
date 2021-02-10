
# Variance

`변성` 이라는 뜻.

다음과 같은 상황일 때, 

- `A`타입, `B`타입이 있다.
- `A ≤ B`의 의미는 
	- `A`가 `B`의 서브타입이라는 표현이다. 
	- `A`보다 `B`가 더 많은 타입을 포괄한다는 뜻이다.
	- `B` 가 반드시 부모 타입이라는 것은 아니다.
- `F`가 어떤 타입을 품고 새로운 타입으로 변환된다. (리스트와 같은 컨테이너)
- `A ≤ B` 이고, `F(A) ≤ F(B)` 이면 공변성이다.
- `A ≤ B` 이고, `F(B) ≤ F(A)` 이면 반공변성이다.

검색해보면 이런 정의는 차고 넘치는데, 왜 이렇게 되는지 설명하는 글을 거의 없는 것 같다.

그래서 나름대로 검색 짜깁기 + 추측으로 소설을 써 본다.

## 왜 필요한가?

어떤 [블로그](https://schneide.blog/2015/05/11/declaration-site-and-use-site-variance-explained/)를 보면 왜 변성을 사용해야 하는지 예제를 들어 설명하고 있다.

``` java
// 사과를 담는 리스트를 만든다.
List<Apple> apples = new ArrayList<>();

// 과일을 담는 리스트로 캐스팅한다.
List<Fruit> fruits = apples;

// 과일을 담는 리스트에 오렌지를 넣는다.
fruits.add(new Orange());
 
// 여기서 오렌지가 나오면 어떻게 될까?
Apple apple = apples.get(0);

```

변성을 사용하면 이러한 문제를 컴파일 시점에 차단할 수 있다고 한다. 

이런 문제는 자바 뿐 아니라, C# 에서도 유효하다고 하니 잘 알아두면 좋을 것 같다.

## 이거 누가 만들었나

자바의 경우 유명한 마틴 오더스키 외 그의 동료들이 1996년 쯤에 팀([Team GJ](http://lampwww.epfl.ch/gj/))을 꾸려서 만들기 시작했다.

그 당시 만들었던 [GJ의 튜토리얼](http://lampwww.epfl.ch/gj/Documents/gj-tutorial.pdf)을 보면 지금의 변성 설정과 매우 비슷하다는 것을 엿볼 수 있다. 와일드카드 구문을 비롯한 타입 추론 까지 가능한 컴파일러를 만들어놨다.

> 이건 개인적인 생각인데, 타입 추론이 필요한 이유는 컴파일 과정에 반드시 필요하기 때문이라고 본다. 컴파일러 입장에서 생각해보면, 타입이 공변적 또는 반공변적 성격을 가지게 될 때는 가능한 모든 타입이 필요한데, 그것을 타입 추론이 제공해준다.

> 여담이지만, 마틴 오더스키가 타입추론 기능을 확장해서 스칼라의 `implicit` 기능을 만든게 아닐까.

서문에 얼마 지나지 않아서 아래와 같은 문구가 있는데,

> In order to keep the language simple, you are forced to do some of the work yourself: you must keep track of the fact that you have a collection of bytes, and when you extract an element from the collection you must cast it to class Byte before further processing.

그도 역시 귀차니즘으로 시작한 게 아닐까 추측해본다.

이후 변성 기능은 자바에서 [2004년 JDK 1.5에 Generics의 스펙에 포함](https://en.wikipedia.org/wiki/Generics_in_Java)되어 적용되었다.

> C#의 경우 [2005년에 버전 2.0에 포함](https://docs.microsoft.com/ko-kr/dotnet/csharp/whats-new/csharp-version-history)되었다고 한다.

## Use-Site Variance VS Declaration-Site Variance

자바는 `Use-Site Variance` 라고 해서, 메소드 선언 시점에만 변성을 정의 할 수 있고 클래스 선언 시점에서는 정의 할 수 없다.

하지만 코틀린과 스칼라는 클래스 선언 시점에도 정의할 수 있다. (`Declaration-Site Variance`)

더 알아보지 않았지만 그정도 차이랄까?

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
	- Immutable 클래스 설계에 쓰일 수 있다. 
		- [코틀린의 AbstractCollection](https://github.com/JetBrains/kotlin/blob/eed0f50c5d08a34cf657df84826890e9a417b3d0/libraries/stdlib/src/kotlin/collections/AbstractCollection.kt#L15)
	- 참고로 코틀린에서는 `out` 선언이 이와 같은 표현이다.
- `Upper Bounds` (상한 제한) 속성이 있다고 말한다.
	- 위의 예제에서 가장 높은 타입인 `Number` 타입으로 변성을 정의하면, `Number` 타입 리스트와 그 하위 타입의 리스트를 할당 할 수 있다.
- 엘리먼트의 타입 관계가 그대로 리스트까지 전이되는 결과를 가져오므로 `공변적`이라고 말한다.
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

> `List` 클래스는 한 개 인데, `List<Integer>` 보다 `List<Number>`가 타입이 더 크다니 이게 무슨 소린가 할 수도 있다. `Integer`는 명시적으로 `Number`를 상속하여 구현하고 있으니까 `Integer`보다 `Number` 타입이 더 크다고 하면 끄덕일 수 있지만, `List`는 클래스가 동일하고 타입만 다른 것이니 헷갈릴만 하다. 타입이 크다는 것은 꼭 상속을 이야기 하는 게 아니라 더 많은 타입을 표현(포괄, 포함)할 수 있다는 의미이다.

> 공변이라는 것이, 실제 두 타입의 크고 작음이, 어떤 컨테이너 타입(리스트)에 적용되었을 때도 그대로 전이되느냐 역전되느냐를 보는 것이기 때문에 어려울 수 있다.

> 하지만 컴파일러 입장에서 생각해보면, 상속은 타입의 힌트를 주기 위한 하나의 예시에 불과하다. 컴파일러는 가능한 타입 관계를 나열하고 어느 타입까지를 허용할지에 대해 관심이 있을 것이기 때문이다. (라고 추측해본다)


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
- `Object` 타입의 엘리먼트를 넣을 수 없다.
	- (1) 때문

### 결론

`List <? super T>` 는

- 어떤 타입으로도 읽을 수 없다.
	- 정확히 Object 타입으로 읽을 수 있는데, 그러면 타입을 쓰는 이유가 없다.
- `T` 타입 또는 `T` 타입의 서브 클래스의 인스턴스만 넣을 수 있다.

그렇기 때문에 `super` 제한자는

- `Lower Bounds` (하한 제한) 속성이 있다고 말한다.
	- 위의 예제에서 `Integer` 타입으로 변성을 설정하면, `Interger` 타입 리스트와 그 상위 타입의 리스트만 할당할 수 있었다.
	- 엘리먼트의 타입 관계가 반대로 전이되므로 `반공변적`이라고 말한다.
	- `Interger ≤ Number` 일 때, `List<Number> ≤ List<Interger>` 이다.

``` java

// Integer 의 타입은 1개의 타입을 수용할 수 있다. 자기 자신.
Integer foo2 = new Integer(0);
Integer foo2 = new Number(0); // Error
Integer foo2 = new Object(0); // Error

// Number 의 타입은 3개의 타입을 수용할 수 있다.
Number foo2 = new Integer(0);
Number foo2 = new Number(0);
Number foo2 = new Object(0); // Error

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

어떤 곳에서는 무공변이라고 표현한다.

자바는 클래스 선언에 공변 설정을 할 수 없으므로, 무공변이다.


#### 실제 코드를 보자

아래 코드는 JDK 1.8 에서 가져온 `Function` 클래스이다.

`compose` 메소드 선언 시점에 공변 설정이 되어 있다.

``` java
public interface Function<T, R> {

  R apply(T t);
  
  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
}

```


아래는 파라미터 1개짜리 코틀린 함수이다.


``` kotlin
public interface Function1<in P1, out R> : Function<R> {
    public operator fun invoke(p1: P1): R
}

```

`P1`은 `Lower bounds` 이므로 
- P1과 P1보다 더 큰 타입이 대상이지만
- P1 타입이나 P1의 하위타입의 인스턴스로만 **수정** 할 수 있다. 
- 반공변성

`R`은 `Upper bounds` 이므로 
- R과 R 하위의 작은 타입이 대상이지만
- R 타입으로만 읽을 수 있다. 
- 공변성


출처 및 참고: 

1. [https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java](https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java)
2. [https://schneide.blog/2015/05/11/declaration-site-and-use-site-variance-explained/](https://schneide.blog/2015/05/11/declaration-site-and-use-site-variance-explained/)
3. [https://iosroid.tistory.com/70](https://iosroid.tistory.com/70)

