---
layout: post
comments: true
title: 객체지향 프로그래밍(43강 - 강)
category: Java
shortinfo: Object, String, Wrapper 클래스 정리
tags:
- java
- OOP
---



## Object 클래스

- java.lang.Object: 모든 클래스의 최상위 클래스
- 모든 클래스는 Object클래스에서 상속받아 메서드를 사용할 수 있음
- 모든 클래스는 Object 클래스의 일부 메서드를 재정의하여 사용할 수 있음(final로 선언된 메서드 제외)



#### toString()

```java
getClass().getName() + "@" + Integer.toHexString(hashCode())
```

- 객체의 정보를 String으로 바꿀 때 유용
- 자바 클래스 중 이미 정의된 클래스가 많음
  - 예: String, Integer, Calendar
- 많은 클래스에서 재정의하여 사용



#### equals()

- 두 객체의 동일함을 논리적으로 재정의 가능
- 물리적 동일함: 같은 주소를 가지는 객체
- 논리적 동일함: 같은 학번의 학생, 같은 주문 번호의 주문
- 물리적으로 다른 메모리에 위치한 객체라도 논리적으로 동일함을 구현하기 위해 사용하는 메서드



#### hashCode()

- 반환값: 인스턴스가 저장된 가상머신의 주소를 10진수로 변환

> 두 개의 서로 다른 메모리에 위치한 인스턴스가 동일하다는 의미?
>
> - 논리적으로 동일: equals()의 반환값이 true
> - 동일한 hashCode 값을 가짐: hashCode() 반환값이 동일



#### clone()

- 객체의 복사본을 만들며 기본 틀(prototype)으로부터 같은 속성 값을 가진 객체의 복사본 생성

- 객체지향 프로그래밍의 정보은닉에 위배되는 가능성이 있으므로 복제할 객체는 cloneable 인터페이스를 명시해야 함

  ```java
  class Example implements Cloneable {
      ...
  }
  ```



## String 클래스

#### 선언하기

```java
String str1 = new String("abc");    // 인스턴스로 생성
String str2 = "abc";				// 상수 풀에 있는 문자열을 가리킴
```



#### String은 immutable

- 한 번 선언되거나 생성된 문자열은 변경할 수 없음
- String 클래스의  `concat()` 메서드 또는 `"+"` 를 이용하여 String을 연결하는 경우 문자열은 새로 생성 됨



#### StringBuilder & StringBuffer

- 가변적인 char[] 배열을 멤버 변수로 가지고 있는 클래스
- 문자열을 변경하거나 연결하는 경우 사용하면 편리한 클래스
- StringBuffer는 멀티 스레드 프로그래밍에서 동기화 보장
- 단일 스레드 프로그래밍에서는 StringBuilder를 사용하는 것이 더 좋음
- `toString()` 메서드로 String 변환



## Wrapper 클래스

| 기본형  | Wrapper 클래스 |
| :-----: | :------------: |
| boolean |    Boolean     |
|  byte   |      Byte      |
|  char   |   Character    |
|  short  |     Short      |
|   int   |    Integer     |
|  long   |      Long      |
|  float  |     Float      |
| double  |     Double     |

