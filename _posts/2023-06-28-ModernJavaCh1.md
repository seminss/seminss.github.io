---
title: 자바 8,9,10,11 : 무슨 일이 일어나고 있는가?
date: 2023-06-28 12:28:00 +0900
categories: [Java, Modern Java in Action]
tags: [Java]
render_with_liquid: false
---


# 1.1 역사의 흐름은 무엇인가?

> 자바 1.1(1997) → 자바 7(2011) → 자바 8(2014) → 자바 10(2018년 3월) → 자바 11(2018년 9월) → 자바 17(2021년 9월)

자바 역사를 통틀어 가장 큰 변화가 자바 8에서 일어났다.

<br>

### ☑️자연어에 가까운 코드 구현
자바 8을 이용함으로써 자연어에 더 가깝게 간단한 방식으로 코드를 구현할 수 있게 되었다.

```java
/**이전**/
Collections.sort(inventory, new Comparator<Apple>()){
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2,getWeight());
  }
});
```
```java
/**java8**/
inventory.sort(comparing(Apple::getWeight));
```

### ☑️간결한 코드, 멀티코어 프로세서의 쉬운 활용

멀티코어 CPU가 대중화 되었지만, 자바 8이 등장하기 이전 자바 프로그램은 여러 코어 중 하나만을 사용했다.
이전까지도 병렬 실행을 위해 여러 도구 및 모델을 도입했지만, 자바 8부터 **병렬 실행을 새롭고 단순한 방식으로 접근할 수 있는 방법이 등장하게 되었다.**

이 새로운 기법을 이용하려면 지켜야 하는 몇 가지 규칙이 있기는 하다. (뒷 부분에서 다룰 예정)

그래서 ? 자바 8에서 제공하는 새로운 기술이라는 것은 아래와 같다.

> - 스트림 API
> - 메서드에 코드를 전달하는 기법 (메서드 참조와 람다)
> - 인터페이스의 디폴트 메서드


[//]: # (![img.png]&#40;morden_java/Unix commands related to the stream.png&#41;)


<br>

---

# 1.2 왜 아직도 자바는 변화하는가?

[//]: # (## 1.2.1 프로그래밍 언어 생태계에서 자바의 위치)




## 1.2.2 스트림 처리

*스트림*이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다.

이론적으로 프로그램은 입력 스트림에서 데이터를 한 개씩 읽어들이며 마찬가지로 출력 스트림으로 데이터를 한 개씩 기록한다.
즉, 어떤 프로그램의 출력 스트림은 다른 프로그램의 입력 스트림이 될 수 있다.

일례로 유닉스나 리눅스의 많은 프로그램은 표준 입력에서 데이터를 읽은 다음에, 데이터를 처리하고, 결과를 표준 출력으로 기록한다.

유닉스의 `cat` 명령은 두 파일을 연결해서 스트림을 생성하며, `tr`은 스트림의 문자를 번역하고, `sort`는 스트림의 행을 정력하며, `tail-3`은 스트림의 마지막 3개 행을 제공한다.

다음 예제처럼 유닉스 명령행에서는 파이프 (`|`) 를 이용해서 명령을 연결할 수 있다.

```shell
cat faile1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
```

`sort`는 여러 행의 스트림을 입력으로 받아 여러 행의 스트림을 출력으로 만들어낸다.

유닉스에서는 여러 명령(cat, tr, sort, tail)을 병렬로 실행한다. 따라서 `cat`이나 `tr`이 완료되지 않은 시점에서 `sort`가 행을 처리할 수 있다.

자바 8에서는` java.util.stream` 패키지에 스트림 API가 추가되었다. 스트림 패키지에 정의된 `Stream<T>` 는 `T` 형식으로 구성된 일련의 항목을 의미한다.

우선은 **스트림 API가 조립 라인처럼 어떤 항목을 연속으로 제공하는 어떤 기능이라고 단순하게 생각**하고 넘어가면 된다.
스트림 API를 사용함으로서 스레드라는 복잡한 작업을 사용하지 않고도 공짜로 **`병렬성`**을 얻을 수 있다.


<br>


## 1.2.3 동작 파라미터화로 메서드에 코드 전달하기

자바 8에서는 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공한다.
이러한 기능을 이론적으로 `동작 파라미터화`라고 부른다.
스트림 API는 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기초하기 때문이다.

[//]: # (![img.png]&#40;morden_java/method as parameter of sort.png&#41;)



<br>



## 1.2.4 병렬성과 공유 가변 데이터

`병렬성`을 얻기 위해 스트림 메서드로 전달하는 코드의 동작 방식을 조금 바뀌어야 한다.

스트림 메서드로 전달하는 코드는 **다른 코드와 동시에 실행하더라도 안전하게 실행될 수 있어야** 하고,
안전하게 실행 할 수 있는 코드를 만들기 위해서는 **가변 데이터에 접근하지 않아야** 한다.

이러한 함수를  `순수(pure) 함수`,  `부작용 없는(side-effect-free) 함수`,  `상태 없는(stateless) 함수` 라 부른다.
(synchronized를 이용해서 공유된 가변 데이터를 보호할 수도 있지만, 그러면 생각보다 비싼 대가를 치뤄야 할 수도 있기 때문에..)

공유되지 않은 가변 데이터(no shared mutable data), 메서드, 함수 코드를 다른 메서드로 전달하는 두가지 기능은 `함수형 프로그래밍`의 핵심적인 사항이며
이후 자세히 다룰 예정이다.

반면 `명령형 프로그래밍`(함수형 프로그래밍의 반대) 패러다임에서는 일련의 가변 상태로 프로그램을 정의한다.

공유되지 않은 가변 데이터 요구사항이란 인수를 결과로 변환하는 기능과 관련된다. 즉, 이 요구사항은 수학적인 함수처럼 함수가 정해진 기능만 수행하며
다른 부작용을 일으키지 않음을 의미한다.


<br>


## 1.2.5 자바가 진화해야 하는 이유

지금까지 자바는 계속해서 진화해왔다. 자바 5에서 `제네릭`이 등장했고, `List`가 `List<String>`으로 바뀌게 되었다.
`Iterator` 대신 `for-each` 루프를 사용하게 되는 변화도 있었다.

처음엔 낯설었지만, 결과론적으로 이 변화들이 프로그래머에게 여러 편리함을 가져다 주었다는 것은 부정할 수 없다.

전통 객체지향 프로그래밍과 함수형 프로그래밍은 완전히 상극이지만,
자바 8에서 함수형 프로그래밍을 도입함으로써, 두 가지 프로그래밍 패러다임의 장점을 모두 활용할 수 있게 되었다.

언어는 하드웨어나 프로그래머 기대의 변화에 부응하는 방향으로 진화해야 한다.

자 그럼 이제 본격적으로 자바 8에 추가된 새로운 개념을 하나씩 살펴보자.


<br>

---
# 1.3 자바 함수

프로그래밍 언어에서 함수라는 용어는 메서드, 특히 정적 메서드와 같은 의미로 사용된다.
**자바의 함수는 이에 더해 수학적인 함수처럼 사용되며 부작용을 일으키지 않는 함수를 의미**한다.

자바 8에서는 함수를 새로운 값의 형식으로 추가했다.
이는 1.4 멀티 코어에서 병렬 프로그래밍을 활용할 수 있는 *스트림*과 연계될 수 있도록 함수를 만들었기 때문이다.

먼저 함수를 값처럼 취급한다고 했는데, 이 특징이 어떤 장점을 제공하는지 살펴보자.


### 프로그래밍 언어의 핵심은 값을 바꾸는 것이다.

전통적으로 프로그래밍 언어에서는 값을 바꿀 수 있는 값을 **일급(first-class)값**, 또는 **시민(citizen)** 이라 부른다.

자바 프로그래밍에서 일급 값은 42(int형), 3.14(double형) 등의 `기본 값`과 `객체`가 일급 값으로 칭해질 수 있다.

> new 또는 팩토리 메서드 또는 라이브러리 함수를 이용해서 객체의 값을 얻을 수 있고, 객체 참조는 클래스의 인스턴스를 가리킨다.
>
> 예를 들어 "abc"(String형), new Integer(1111) (Integer형), new HashMap<Integer, String>(100) (HashMap의 생성자를 호출) 등으로 객체 참조를 얻을 수 있다.

자유롭게 전달할 수 없는 구조체는 **이급 시민**이다. `메서드`, `클래스` 등이 이급 자바 시민에 해당한다.

인스턴스화한 결과가 값으로 귀결되는 클래스를 정의할 때 메서드를 아주 유용하게 활용할 수 있지만 여전히 메서드와 클래스는 그 자체로 값이 될 수 없다.

**자바 8 설계자들은 이급 시민을 일급 시민으로 바꿀 수 있는 기능을 추가했다.**

<br>

## 1.3.1 메서드와 람다를 일급 시민으로

자바 8 설계자들은 메서드를 값으로 취급할 수 있게, 그리하여 프로그래머들이 더 쉽게 프로그램을 구현할 수 있는 환경이 제공되도록 자바 8을 설계하기로 결정했다.

더불어 자바 8에서 메서드를 값으로 취급할 수 있는 기능은 스트림 같은 다른 자바 8 기능의 토대를 제공했다.

```java
/**자바 8 이전**/
File[] hiddenFiles = new File(".").listFiles(new FileFilter(){
    public boolean accept(File file){
        return file.isHidden();
  }
});
```

```java
/**이후 코드**/
File[] hiddenFiles = new File(".").listFilles(File::isHidden);
```

자바 8의 메서드 참조 `::`('이 메서드를 사용하라'란 의미)를 이용해서 `isHidden` 함수를 `listFiles`에 직접 전달한다.

**기존에 비해 문제 자체를 더 직접적으로 설명한다**는 점이 자바 8 코드의 장점이다.

**자바 8에서는 더 이상 메서드가 이급값이 아닌 일급값이다.**
기존 객체 참조를 이용해서 객체를 이리저리 주고받았던 것처럼 자바 8에서는 `File::isHidden`을 이용해서 메서드 참조를 만들어 전달할 수 있게 되었다.

### 람다 : 익명함수

자바 8에서는 메서드를 일급값으로 취급할 뿐 아니라 람다를 포함하여 함수도 값으로 취급할 수 있다.

람다 문법 형식으로 구현된 프로그램을 함수형 프로그래밍, 즉 '함수를 일급값으로 넘겨주는 프로그램을 구현한다.'라고 한다.

[//]: # (![img.png]&#40;morden_java/compare old and new.png&#41;)

<br>

## 1.3.2 코드 넘겨주기 : 예제

특정 항목을 선택해서 반환하는 동작을 필터라고 한다. 자바 8 이전에는 `필터링`을 하기 위해 메서드를 하나하나 구현해야 했다.

여러가지 조건을 주고 싶다면 동일한 틀의 메서드를 복사 붙여넣기 해서,
`GREEN.equals(apple.getColor())` 또는 `apple.getWeight()>150` 처럼 조건식만 변경해야 했을 것이다.

그러나 자바 8에서는 코드를 인수로 넘겨줄 수 있으므로 `filter` 메서드를 중복으로 구현할 필요가 없어졌다.

```java
public static boolean isGreeanApple(Apple apple){
    return GREEN.equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple){
    return apple.getWeight()>150;
}

public interface Predicate<T>{ //java.util.function에서 임포트 가능하나 의미를 명확히 하기 위해 선언
    boolean test(T t);
}

//메서드가 p라는 이름의 프레디 케이트 파라미터로 전달됨
static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p){
    List<Apple> result=new ArrayList<>();
    for(Apple apple: inventory){
        if(p.test(apple)){ //사과는 p가 제시하는 조건에 맞는가?
            result.add(apple);
        }
    }
    return result;
}
```

```java
/**호출**/
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

코드의 작동 방식은 이후 챕터에서 자세히 다루기로 하고, 여기서 핵심은 자바 8에서는 메서드를 인자로 전달할 수 있다는 사실이다.

> #### 프레디케이트(predicate)란 무엇인가?
> 앞에서 다룬 예제에서는 Apple::isGreenApple 메서드를 filterApples로 넘겨주었다.(filterApples는 (Predicate<Apple>를 파라미터로 받음))
> 수학에서는 인수로 값을 받아 true나 false를 반환하는 함수를 프레디케이트라고 한다.

<br>

## 1.3.3 메서드 전달에서 람다로

메서드를 값으로 전달하는 것은 유용한 기능이지만, `isHeavyApple`이나 `isGreenApple`처럼 한두 번만 사용할 메서드를 매번 정의하는 것은 귀찮은 일이다.

자바 8에서는 람다라는 새로운 개념을 이용한다.

```java
filterApples(inventory, (Apple a)->GREEN.equals(a.getColor()));
filterApples(inventory, (Apple a)->a.getWeight()>150);
filterApples(inventory, (Apple a)->GREEN.equals(a.getColor())||RED.equals(a.getColor()));
```

한 번만 사용할 메서드는 따로 정의 및 구현할 필요 없이 사용 가능하게 된 것이다.

사실 라이브러리 메서드 `filter`를 사용하면 `filtertApples` 메서드의 구현 없이 `filter(inventory, (Apple a)->a.getWeight()>150);`
와 같은 작성이 가능하지만,
자바 8에서는 **병렬성**이라는 중요성 때문에 이와 같은 설계를 포기했다.

대신, **`filter`와 비슷한 동작을 수행하는 연산 집합을 포함하는 새로운 `스트림 API`를 제공한다.**

<br>
---

# 1.4 스트림

```java
/**컬렉션 API**/
  Map<Currency, List<Transaction>> transcationByCurrencies = new HashMap<>(); //그룹화된 트랜잭션을 더할 Map 생성
  for(Transaction transaction : transactions){ //트랜잭션의 리스트를 반복
    if(transaction.getPrice()>1000){ //고가의 트랜잭션을 필터링
        //트랜잭션 통화 추출 코드 (get)
        //현재 통화의 그룹화된 맵에 항목이 없으면 새로 만든다. (null이면 new, put)
    }
    transactionForCurrency.add(transaction); //현재 탐색된 트랜잭션을 같은 통화의 트랜잭션 리스트에 추가한다.
  }
```

```java
/**스트림 API**/
  import static java.util.stream.Collectors.groupingBy;
  Map<Currency, List<Transaction>> transcationByCurrencies = 
    transactions.stream()
            .filter((Transaction t)-> t.getPrice()>1000) //고가의 트랜잭션 필터링
            .collect(grouping By(Transaction::getCurrency)); //통화로 그룹화 합

```

`스트림 API`를 이용하면 `컬렉션 API`와는 상당히 다른 방식으로 데이터를 처리할 수 있다.

컬렉션에서는 **반복 과정을 직접 처리**(for-each) 하는데, 이런 방식의 반복을 **외부 반복**이라 한다.

반면 스트림의 경우에는 **라이브러리 내부에서 모든 데이터가 처리**된다. 이런 방식을 **내부 반복**이라 한다.

컬렉션은 단일 CPU에서 반복문을 돌려 데이터를 처리하지만, 스트림은 데이터를 병렬적으로 처리할 수 있기 때문에 사용하는 코어의 갯수배만큼 작업 속도가 빨라진다.

<br>

## 1.4.1 멀티스레딩은 어렵다

이전 자바 버전에서 제공하는 스레드 API로 멀티스레딩 코드를 구현해서 병렬성을 갖도록 하는 것은 쉽지 않았다. (동기화 문제)

자바 8은 스트림 API로 컬렉션을 처리하면서 발생하는 _모호함과 반복적인 코드 문제_, 그리고 _멀티 코어 활용 어려움_ 이라는 두가지 문제를 모두 해결했다.

`컬렉션`은 어떻게 데이터를 저장하고 접근할지에 중점을 두는 반면, **`스트림`은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중점을 둔다.**
스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것이 핵심이다.

심지어 컬렉션을 필터링할 수 있는 가장 빠른 방법은 컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에,
리스트를 다시 복원하는 것이기도 하다.

```java
/**순차 처리**/
import static java.util.stream.Collectors.toList; 
List<Apple> heavyApples = 
    inventory.stream().filter((Apple a) -> a.getWeight() > 150) 
                      .collect(toList()); 
```
```java
/**병렬 처리**/
import static java.util.stream.Collectors.toList; 
List<Apple> heavyApples =
  inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
                            .collect(toList());
```

<br>

---

[//]: # ()
[//]: # (# 1.5 디폴트 메서드와 자바 모듈)

[//]: # ()
[//]: # (# 함수형 프로그래밍에서 가져온 다른 유용한 아이디어)

[//]: # ()
[//]: # (> JVM을 구성하는 자바 및 기타 언어에서 함수형 프로그래밍이라는 존재가 어떤 영향을 미치는지 제시한다.)

[//]: # ()


