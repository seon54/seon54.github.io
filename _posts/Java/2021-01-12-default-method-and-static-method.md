---
layout: post
comments: true
title: Java 8 - 디폴트 메서드와 정적 메서드
category: Java
shortinfo: 디폴트 메서드(default method)와 정적 메서드(static method)  알아보기
tags:
- java
---




> 더 자바  Java 8 강의, 모던 자바 인 액션 그리고 자바 공식문서를 보며 Java8 정리



## 디폴트 메서드(Default method)

인터페이스를 구현하는 클래스는 인터페이스에서 정의한 메서드를 모두 구현해야 한다. 디폴트 메서드는 라이브러리 설계자들의 유연한 설계와 구현을 위해 추가되었다.  디폴트 메서드는 인터페이스를 구현하는 클래스에서 구현하지 않아도 되며 인터페이스에서 기본으로 구현된다.

- 인터페이스에 메서드 선언이 아닌 구현체 제공
- 인터페이스를 구현한 클래스에 영향을 주지 않고 새 기능 추가 가능
- 디폴트 메서드의 리스크
  - 구현하는 클래스 모르게 추가된 기능으로, 구현하는 클래스에 따라 런타임 에러 발생 가능
  - `@implSpec` 자바독 태그 이용하여 문서화
- `java.lang.Object`가 제공하는 메소드(equals, hashCode)는 디폴트 메서드 불가
  - 인터페이스를 구현하는 클래스가 재정의 가능
- 인터페이스를 상속받은 인터페이스에서 추상 메서드로 변경 가능
- 인터페이스를 구현하는 클래스가 재정의 가능



## 정적 메서드(Static method)

이전까지는 정적 메서드를 정의하는 유틸리티 클래스를 사용했으나 Java 8에서는 인터페이스에서 정적 메서드를 선언할 수 있게 했다. 정적 메서드 또한 디폴트 메서드처럼 유연한 구현을 가능하게 한다.



## Java8의 API 변화

Java8에서 추가된 주요 디폴트 메서드와 정적 메서드를 살펴보도록 한다.

| interface            | default method                                            | static method                                                |
| -------------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| java.lang.Iterable   | forEach(), spliterator()                                  |                                                              |
| java.util.Collection | stream(), parallelStream()<br>removeIf()<br>spliterator() |                                                              |
| java.util.Comparator | reversed()<br>thenComparing()                             | reverseOrder(), naturalOrder()<br>nullsFirst(), nullsLast()<br>comparing() |

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Spliterator;
import java.util.stream.Collectors;

public class Example {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("harry");
        names.add("hermione");
        names.add("ron");
        names.add("black");

        System.out.println("=========== forEach ==========");
        names.forEach(System.out::println);

        Spliterator<String> spliterator = names.spliterator();
        Spliterator<String> spliterator1 = spliterator.trySplit();
        System.out.println("=========== first spliterator ==========");
        while (spliterator.tryAdvance(System.out::println)) ;
        System.out.println("=========== second spliterator ==========");
        while (spliterator1.tryAdvance(System.out::println)) ;

        List<String> h = names.stream().map(String::toUpperCase)
                .filter(s -> s.startsWith("H"))
                .collect(Collectors.toList());

        System.out.println("=========== stream, map, filter, collect ==========");
        h.forEach(System.out::println);

        names.removeIf(s -> s.startsWith("b"));
        System.out.println("=========== removeIf ==========");
        names.forEach(System.out::println);

        Comparator<String> compareToIgnoreCase = String::compareToIgnoreCase;
        names.sort(compareToIgnoreCase.reversed());
        System.out.println("=========== sort ==========");
        names.forEach(System.out::println);
    }
}
```
```
=========== forEach ==========
harry
hermione
ron
black
=========== first spliterator ==========
ron
black
=========== second spliterator ==========
harry
hermione
=========== stream, map, filter, collect ==========
HARRY
HERMIONE
=========== removeIf ==========
harry
hermione
ron
=========== sort ==========
ron
hermione
harry
```

