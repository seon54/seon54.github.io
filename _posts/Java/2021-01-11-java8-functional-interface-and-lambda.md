---
layout: post
comments: true
title: Java 8 - 함수형 인터페이스와 람다 표현식
category: Java
shortinfo: 함수형 인터페이스(Functional Interface)와 람다 표현식(Lambda Expressions)  알아보기
tags:
- java
---




> 더 자바  Java 8 강의, 모던 자바 인 액션 그리고 자바 공식문서를 보며 Java8 정리



## 함수형 인터페이스(Funtional Interface)

- 람다 표현식과 메서드 참조를 위해 대상 타입 제공
- `추상메서드`가 1개만 존재
  - `함수형 메서드(functional method)`라 불림
  - 람다 표현식의 매개 변수와 반환 타입이 일치
  - `abstract` 키워드 생략 가능
  - SAM(Single Abstract Method) 인터페이스라고도 불림
  - 구현된 메서드가 여러개 있어도 추상메서드가 1개만 있으면 됨
- `static` 메서드에 `public` 제한자도 생략 가능
- `@FunctionalInterface` 애노테이션을 가지고 있는 인터페이스

```java
// Functional Interface example

@FunctionalInterface
public interface NewFunctionalInterface {

    void doIt();

    static void printName(String name) {
        System.out.println(name);
    }

    default void printAge(int age) {
        System.out.println(age);
    }
}
```



### 기본 함수형 인터페이스

`java.util.function` 패키지에서 제공하며 자주 사용할만한 함수 인터페이스를 미리 정의해두었다. 

| 인터페이스     | 설명                        | 메소드       | 함수 조합용 메소드   |
| -------------- | --------------------------- | ------------ | -------------------- |
| Function<T, R> | T 타입을 받아서 R 타입 반환 | R apply(T t) | andThen(), compose() |
|BiFunction<T, U, R>|두 개의 값(T, U)를 받아 R 타입 반환|R apply(T t, U u)|andThen()|
|Consumer\<T>|T 타입을 받아 아무것도 반환하지 않음|void accept(T t)|andThen()|
|Supplier\<T>|T 타입 값 제공|T get()||
|Predicate\<T>|T 타입을 받아 boolean 반환|boolean test(T t)|and(), or(), negate()|
|UnaryOperator\<T>|입력값을 하나 받아 동일한 타입 반환|T apply(T t)|andThen(), compose()|
|BinaryOperator\<T>|동일한 타입의 입력값 두 개를 받아 반환|T apply(T t1, T t2)|andThen()|



## 람다 표현식(Lambda Expressions)

#### 특징

- 이름이 없는 익명 함수를 단순화
- 메서드의 매개변수, 반환 타입, 변수로 사용 가능
- 함수형 인터페이스의 인스턴스를 만들 때 사용 가능
- 코드를 간결하게 함

#### 구성

- (파라미터) -> 바디
- 파라미터
  - 파라미터가 없을 경우, ()
  - 파라미터가 하나일 경우, (param) 또는 param
  - 파라미터가 여러 개일 경우, (p1, p2)
  - 컴파일러가 추론할 수 있어 파라미터 타입은 생략 가능
- 바디
  - 화살표 오른쪽에 함수 정의
  - 여러 줄일 경우,  `{ }` 사용
  - 한 줄인 경우, `{ }`와 `return`  생략 가능

#### 변수 캡쳐(Variable Capture)

- effective final
  - 사실상 final인 변수
  - final 키워드가 없는 변수를 익명 클래스 구현체나 람다에서 참조 가능
- 지역 변수 캡쳐
  - `final` 이거나 `effective final`인 경우에만 참조 가능
  - 그렇지 않을 경우 동시성 문제 발생 가능
- scope
  - 이너 클래스나 익명 클래스 구현체는 새로운 scope를 가지지만, 람다는 람다를 감싸는 scope와 같음

#### 메소드 레퍼런스(Method Reference)

| 종류                                         | 방법                           | 예제                        |
| -------------------------------------------- | ------------------------------ | --------------------------- |
| 스태틱 메소드 참조                           | 타입::스태틱 메소드            | Integer::parseInt           |
| 특정 객체의 인스턴스 메소드 참조             | 객체 레퍼런스::인스턴스 메소드 | member::findById            |
| 특정 유형의 임의 객체의 인스턴스 메소드 참조 | 타입::인스턴스 메소드          | String::compareToIgnoreCase |
| 생성자 참조                                  | 타입::new                      | Member::new                 |

**위 예제에서 Member라는 객체가 있다고 가정*



```java
// Lambda Expressions example
public class Test {
    
    public static void main (String[] args) {
        NewFunctionalInterface newInterface = () -> System.out.println("Hello");
        newInterface.doIt(); // Hello
    }
}
```



## 자바에서 함수형 프로그래밍

- 함수형 프로그래밍에서의 함수는 수학적인 함수 의미
  - 0개 이상의 인수를 가지며 한 개 이상의 결과 반환
  - 부작용이 없음(no side effect)
  - 인수가 동일할 경우, 결과가 항상 같음
  - 시스템의 다른 부분에 영향을 미치지 않음
- 지역 변수만을 변경해야 함
  - 함수가 참조하는 객체는 불변 객체
  - 객체의 모든 필드가 `final` 이어야 함
  - 함수 내에서 생성한 객체의 필드는 갱신할 수 있음
- 예외를 일으키지 않아야 함
- 함수형 인터페이스를구현한 객체
- 변수에 할당하고 메서드에 파라미터를 전달하거나 그 자체를 리턴하는 것도 가능
- 함수가 함수를 리턴하는 것도 가능

  