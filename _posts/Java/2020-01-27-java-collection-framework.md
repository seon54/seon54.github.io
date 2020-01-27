---
layout: post
comments: true
title: 객체지향 프로그래밍(48강 - 54강)
category: Java
shortinfo: 
tags:
- java
- OOP
---



## Collection Framework

- 프로그램 구현에 필요한 자료구조와 알고리즘을 구현한 라이브러리
- `java.util` 패키지
- 개발에 소요되는 시간을 절약하고 최적화된 라이브러리 사용 가능
- Collection 인터페이스와 Map 인터페이스로 구성

#### Collection Interface

- 하나의 객체의 관리를 위해 필요한 기본 메서드가 선언되어 있음
- 하위에 `List`, `Set` 인터페이스가 있음
- List
  - 순서가 있는 자료 관리
  - 중복 허용
  - List로 ArrayList, Vector, LinkedList, Stack, Queue 구현
- Set
  - 순서가 정해져 있지 않음
  - 중복 허용하지 않음
  - 구현한 클래스를 이용할 때는 `equals()` 와 `hashCode()` 메서드를 재정의해야 중복된 객체를 구분할 수 있음
  - Set으로 HashSet, TreeSet 구현

#### Map Interface

- 쌍으로 이루어진 객체를 관리하는데 필요한 메서드가 선언되어 있음
- Map을 사용하는 객체는 key-value 쌍으로 되어 있고 key는 중복될 수 없음
- HashTable, HashMap, TreeMap



## Generic Programming

- 변수의 선언이나 메서드의 매개변수를 하나의 참조 자료형이 아닌 여러 자료형으로 변환될 수 있도록 하는 방식
- 실제 사용되는 참조 자료형으로서의 변환은 컴파일러가 검증하므로 안정적

#### 자료형 매개변수 T

- 여러 참조 자료형으로 대체될 수 있는 부분을 문자로 표현

- type의 의미

- 두 개 이상도 사용 가능

  ```java
  public class GenericPrinter<T> {
      private T material;
      
      public void setMaterial(T material) {
          this.material = material;
      }
      
      public T getMaterial() {
          return mateiral;
      }
  }
  ```

#### \<T extends Class>

- T 대신에 사용될 자료형을 제한하기 위해 사용
- 클래스에 정의된 메서드 공유 가능

#### Generic Method

- 메서드의 매개 변수를 자료형 매개 변수로 사용하는 메서드
- 메서드 내에서의 자료형 매개 변수는 메서드 내에서만 유효(지역 변수와 같은 개념)



## List Interface

- Collection 하위 인터페이스
- 객체를 순서에 따라 저장하고 관리하는데 필요한 메서드가 선언된 인터페이스
- 배열의 기능을 구현하기 위한 메서드 선언
- ArrayList, Vector, LinkedList

#### ArrayList와 Vector

- 객체 배열 클래스
- Vector
  -  Java2부터 제공하며 멀티 쓰레드 프로그램에서 동기화 지원
  - 동기화: 두 개의 쓰레드가 동시에 하나의 리소스에 접근할 때 순서를 맞춰 데이터 처리 안정적
- ArrayList
  - 일반적으로 더 많이 사용
- capacity: 배열의 용량
- size: 요소의 개수

#### ArrayList와 LinkedList

- 자료의 순차적 구조 구현

- ArrayList
  - 배열을 구현한 클래스로 논리적 순서와 물리적 순서 동일
- LinkedList
  - 논리적으로 순차적인 구조지만 물리적으로는 순차적이지 않을 수 있음

#### Stack 구현하기

- Last In First Out(LIFO): 마지막에 추가된 요소가 먼저 꺼내지는 자료구조
- ArrayList나 LinkedList로 구현 가능

#### Queue 구현하기

- First In First Out(FIFO): 먼저 저장된 자료가 먼저 꺼내지는 자료구조
- 선착순, 대기열 등을 구현할 때 많이 사용되는 자료구조
- ArrayList나 LinkedList로 구현 가능



## Set Interface

- Collection 하위의 인터페이스로 중복을 허용하지 않으며 순서가 없음
- get(i) 메서드가 제고오디지 않고 Iterator로 순회함
- 저장된 순서와 출력 순서는 다를 수 있음
- ID, 주민번호, 사번 등 유일한 값이나 객체를 관리할 때 사용
- HashSet, TreeSet 클래스

#### Iterator로 순환하기

- Collection의 개체를 순환하는 인터페이스

- Iterator에 선언된 메서드

  | 메서드             | 설명                                              |
  | ------------------ | ------------------------------------------------- |
  | boolean hashNext() | 이후에 요소가 더 있는지 체크하며 있으면 true 반환 |
  | E next()           | 다음에 있는 요소 반환                             |

#### HashSet

- Set 인터페이스를 구현한 클래스
- 중복을 허용하지 않으므로 저장되는 객체의 동일 여부를 알기 위해 equals()와 hashCode() 메서드를 재정의 해야 함

#### TreeSet

- 객체의 정렬에 사용하는 클래스
- 중복을 허용하지 않으면서 오름차순이나 내림차순으로 객체 정렬
- 내부적으로 이진 검색 트리(binary search tree)로 구현되어 있음
- 이진 검색 트리에 자료가 저장될 때 비교하여 저장될 위치 정함
- 객체 비교를 위해 Comparable이나 Comparator 인터페이스 구현

#### Comparable 인터페이스와 Comparator 인터페이스

- 정렬 대상이 되는 클래스가 구현해야 하는 인터페이스

- Comparable은 `compareTo()` 메서드 구현

  - 매개 변수와 객체 자신(this) 비교

- Comparator는 `compare()` 메서드 구현

  - 두 개의 매개변수 비교

  - TreeSet 생성자에 Comparator가 구현된 객체를 매개변수로 전달

    ```java
    TreeSet<MyClass> treeset = new TreeSet<MyClass>(new MyClass());
    ```

- 일반적으로 Comparable 더 많이 사용
- 이미 Comparable이 구현된 경우, Comparator를 이용하여 다른 정렬 방식 정의 가능



## Map Interface

- key-value 쌍의 객체를 관리하는데 필요한 메서드 정의

- key는 중복될 수 없음

- 검색을 위한 자료 구조

- key를 이용하여 값을 저장하거나 검색, 삭제할 때 사용. 내부적으로 hash 방식으로 구현

  ```java
  index = hahs(key)
  ```

- key가 되는 객체는 객체의 유일함 여부를 알기 위해 `equals()`와 `hashCode()` 메서드 재정의

#### HashMap

- Map 인터페이스를 구현한 클래스 중 가장 일반적으로 사용하는 클래스
- HashTable 클래스는 Java2부터 제공되어 Vector처럼 동기화 제공
- pari 자료를 쉽고 빠르게 관리 가능

#### TreeMap

- key 객체에 정력하여 key-value를 쌍으로 관리하는 클래스

- key에 사용되는 클래스에 Comparable, Comparator 인터페이스 구현

- 많은 클래스가 Comparable이 구현되어 있어 구현된 클래스를 key로 사용하는 경우는 구현할 필요 없음

  ```java
  public final class Integer extends Number implements Comparable<integer> {
          
      public int compareTo(Integer anotherInteger) {
          return compare(this.value, anotherInteger.value);
      }
  }
  ```

  