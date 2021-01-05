---
layout: post
comments: true
title: 디자인 패턴 - 싱글톤 패턴
category: Java
shortinfo: 싱글톤 패턴 알아보기
tags:
- java
- design pattern
---



## 싱글톤 패턴(Singleton Pattern)

>Head First Design Patterns & GoF의 디자인 패턴 정리



### 방법

```
특정 클래스에 대해서 객체 인스턴스가 하나만 만들어지게 하고 전역적인 접근을 제공한다.
```

스프링 프레임워크의 주요 개념인 DI, IoC 등을 공부하면서 싱글톤 패턴에 처음 알게 되었다. 다른 디자인 패턴에 비해 이름만 봐도 어떤 것을 의도하는지 알 수 있다. 쓰레드 풀이나 캐시 등 하나만 있으면 되는 객체들은 인스턴스가 두 개 이상 만들어지게 되면 프로그램이 원래 의도와는 다르게 돌아갈 수 있다. 스프링 같은 경우는 엔터프라이즈 시스템을 위해 만들어진 프레임워크이므로 많은 요청이 들어올 때마다 인스턴스가 생성되어 부하가 걸리는 것을 방지하기 위해 스프링 빈을 싱글톤으로 생성한다.

싱글톤 패턴에 대해 알아보다가 백기선님의 영상 [[이팩티브 자바] #3 싱글톤을 만드는 여러가지 방법 그중에 최선은?](https://youtu.be/xBVPChbtUhM) 을 보게 되었다. 50분 안 되는 영상인데 정말 재밌게 보았고 큰 도움이 되었다. 이 영상에서 설명하는 Effective Java에서는 싱글톤을 구현하는 일반적인 두 가지 방법을 설명하는데 모두 생성자의 접근 제한자를 `private`으로 두는 것과 오직 하나의 인스턴스만 접근할 수 있도록 `public static` 멤버를 제공한다. 클래스를 만들 때 생성자의 접근 제한자를 public으로 두어야 다른 객체에서도 생성이 가능한데 싱글톤 패턴에서는 객체 생성을 막기 위해서 생성자 접근 제한자를 `private`으로 둔다. 

첫번째 방법은 `public static final` 멤버를 사용하는 것이다. private 생성자는 Elvis.INSTANCE를 생성할 때 오직 한 번만 불리게 된다. 이 방법의 장점은 명확하고 간단하다는 것이다.

```java
// Effective Java example code
// public final 필드 이용
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}    
}

```

두번째 방법에서는 static final 멤버는 `private`으로 바꾸고 `static` 메소드를 통해 인스턴스를 제공한다. 이 방법의 장점은 싱글톤 패턴으로 쓰고 싶지 않을 때 쉽게 바꿀 수 있다는 점이다.

```java
// Effective Java example code
// static factory 이용
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

그리고  `enum`을 이용하는 세번째 방법이 하나 더 있다. enum을 이용하는 것은 처음 봤는데 이 책에서는 이 방법을 가장 권장하고 있는 것 같다. public 필드 방법과 유사하지만, 더 간결하다. 

```java
// Effective Java example code
// Enum singleton - 선호되는 방법
public enum Elvis {
    INSTANCE;
}
```



>**참조**
>토비의 스프링 3.1
>Effective Java 3/e