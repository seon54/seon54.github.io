---
layout: post
comments: true
title: 디자인 패턴 - 전략 패턴
category: Java
shortinfo: 전략 패턴 알아보기
tags:
- java
- design pattern
---



## 전략 패턴(Strategy Pattern)

>Head First Design Patterns & GoF의 디자인 패턴 정리



### 방법

```
알고리즘을 정의하고 각각을 캡슐화하여 교환해서 사용하도록 만든다. 알고리즘을 사용하는 클라이언트와 상관없이 독립적으로 변경할 수 있다.
```



알고리즘이라는 말이 나오니 어렵게 느껴졌다. 헤드 퍼스트에서 Duck 클래스로 든 예를 찬찬히 따라가면 좀 더 이해하기 쉽다. Duck은 추상클래스로서 모든 오리가 가지는 꽥꽥 우는 것과 헤엄치는 것에 대한 메서드는 구현하고 모양은 각각 다르기 때문에 서브클래스가 구현하도록 추상메서드 display()를 구현했다. 

```java
public abstract class Duck {
    public void quack() {
        System.out.println("꽥꽥")
    }
    
    public void swim() {
        System.out.println("헤엄치기")
    }
    
    public abstract void display();
}
```

문제는 Duck에 fly() 메서드를 추가하면서 발생한다. 서브 클래스 중에는 날지 못하는 오리도 있어 일일이 찾아서 변경해줘야 하는 번거로움이 생긴 것이다. 고무 오리 서브클래스는 꽥꽥 소리도 내지 못해 기존의 quack() 메서드도 오버라이드를 해야했다.

fly()나 quack() 등의 메서드는 서브클래스마다 달라질 수 있다. 이런 함수들을 위의 정의에서 말한 알고리즘군으로 볼 수 있다. fly() 메서드를 가지는 FlyBehavior 인터페이스를 만들고 이를 각각 구현한 클래스를 만들면 아래와 같이 된다.

```java
public interface FlyBehavior {
    void fly();
}

public class FlyWithWings implements FlyBehavior {
    public void fly() {
        System.out.println("날아올라")
    }
}

public class NoFly implements FlyBehavior {
    public void fly() {
        System.out.println("못 날아요")
    }
}
```

FlyBehavior 인터페이스를 사용하도록 Duck 클래스도 바꾸도록 한다. 여기서 `위임`이라는 개념이 나오게 된다. Duck 클래스 혹은 서브클래스는 난다는 행위를 직접 처리하지 않고 FlyBehavior에게 위임한다. 

```java
public abstract Duck {
    FlyBehavior flyBehavior;
    
    public void performFly() {
        flyBehavior.fly();
    }
    
   ...
}
```

Duck 클래스를 상속한 고무 오리 클래스를 만들어보자. RubberDuck은 Duck에서 flyBehavior 인스턴스 변수를 상속받고 performFly() 메서드가 실행되면 NoFly 객체에게 위임한다.

```java
public class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = NoFly();
    }
    
    public void display() {
        System.out.println("고무오리에요")
    }
}
```

RubberDuck은 FlyBehavior 인스턴스를 생성자에서 만들었다. 이 행위를 동적으로 지정하기 위해 Duck 클래스에 setter() 메서드를 만들 수 있다.

```java
public abstract Duck {
    FlyBehavior flyBehavior;
    
    public void setFlyBehavior(FlyBehavior flyBehavior) {
        this.flyBehavior = flyBehavior;
    }
    
   ...
}
```

메인 메서드에서 RubberDuck을 실행하면 아래의 결과가 나오게 된다. 위에서 추가한 setter() 메서드 덕분에 실행 중에 고무 오리의 행동을 바꿀 수 있게 되었다.

```java
public class DuckTest {
    
    public static void main(Stirng[] args) {
        Duck rubberDuck = new RubberDuck();
        rubberDuck.performFly();	// 못 날아요
        rubberDuck.setFlyBehavior(new FlyWithWings());
        rubberDuck.performFly();	// 날아올라
    }
}
```



### 정리

전략 패턴을 통해 알 수 있는 객체지향 원칙

- 바뀌는 부분은 캡슐화
  - 데이터와 데이터를 다루는 메서드를 클래스로 묶어 객체 외부에서는 getter(), setter() 메서드 등으로만 접근 가능하게 하기
- 상속보다는 구성 활용
  - 바뀌는 행위를 인터페이스로 만들어 인스턴스 변수로 만든 후, 행위에 대한 것을 위임하기
- 구현이 아닌 인터페이스에 맞춰 프로그래밍 