---
layout: post
comments: true
title: 객체지향 프로그래밍(73강 - 77강)
category: Java
shortinfo: 쓰레드 정리
tags:
- java
---



## Thread

#### Process와 Thread

Process

- 실행중인 프로그램
- OS로부터 메모리를 할당 받음

Thread

- 실제 프로그램이 수행되는 작업의 최소 단위
- 하나의 프로세스는 하나 이상의 thread를 가짐

#### Thread 구현

- `Thread` 클래스로부터 상속 받아 구현

  ```java
  class Test {
      t = new Thread();
      t.start();
  }
  
  class AnyThread extends Thread {
      public void run() {
          
      }
          
  }
  ```

- `Runnable` 인터페이스 구현

  - 다중 상속이 되지 않기 때문에 다른 클래스를 상속한 경우에 인터페이스를 구현하여 thread를 만들도록 함

  ```java
  class Test {
      t = new Thread(new Anything());
      t.start();
  }
  
  class Anything implements Runnable {
      public void run() {
          
      }
  }
  ```

#### Multi Thread

- 동시에 여러 thread 수행
- Thread는 각각의 작업공간(context)를 가짐
- 공유 자원이 있는 경우, race condition 발생
- ciritical section에 대한 동기화 구현 필요  

#### Thread Status

1. `start`를 하게 되면 `runnable` 상태가 됨
2. 스케줄러가 CPU를 배분하여 thread를 `run`
3. thread가 끝나면 `dead` 상태가 됨

#### Thread 우선순위

- `Thread.MIN_PRIORITY`(=1) ~ `Thread.MAX_PRIORITY`(=10)
- 디폴트 우선 순위: Thread.NORM_PRIORITY(=5)
- `setPriority(int new Priority)`
- `int getPriority()`
- 우선 순위가 높은 thread는 CPU를 배분 받을 확률이 높음

#### join() 메서드

- 다른 thread의 결과를 보고 진행해야 하는 일이 있을 경우 활용
- join() 메서드를 호출한 thread가 non-runnable 상태가 됨

#### interrupt() 메서드

- 다른 thread에 예외를 발생시키는 interrupt를 보냄
- thread가 join(), sleep(), wait() 메서드에 의해 블럭킹되었다면 interrupt에 의해 다시 runnable 상태가 될 수 있음

#### Thread 종료

- daemon 등 무한 반복하는 thread가 종료될 수 있도록 run() 메서드 내의 while 문을 활용
- Thread.stop()은 사용하지 않음

#### 임계 영역(Critical section)

- 두 개 이상의 thread가 동시에 접근하는 리소스
- 임계 영역에 thread가 동시에 접근하게 되면 실행 결과를 보장할 수 없으므로 thread간에 순서를 맞추는 동기화 필요

#### 동기화(Synchronization)

- 임계 영역에 여러 thread가 접근할 경우 한 thread가 수행하는 동안 공유 자원을 잠궈 다른 thread의 접근을 막음
- 동기화를 잘못 구현하면 deadlock에 빠질 수 있음

#### Java에서 동기화 구현

- `synchronized` 수행문과 메서드 이용

- synchronized 수행문

  ```java
  synchronized(참조형 수식) {
      // 참조형 수식에 해당되는 객체에 lock을 건다
  }
  ```

- synchronized 메서드
  - 현재 이 메서드가 속해있는 객체에 lock을 건다
  - synchronized 메서드 내에서 다른 synchronized 메서드를 호출하지 않음(deadlock 방지)

#### wait() & notify()

- `wait()`
  - 리소스가 더 이상 유효하지 않은 경우 사용 가능할 때까지 thread를 non-runnable 상태로 전환
  - wait() 상태가 된 thread는 notify()가 호출될 때까지 기다림
- `notify()`
  - wait() 하고 있는 thread 중 한 thread를 runnable한 상태로 깨움
- `notifyAll()`
  - wait()하고 있는 모든 thread가 runnable한 상태가 되도록 함
  - notify()보다 notifyAll()을 권장
  - 특정 thread가 통지받도록 제어할 수 없으므로 모두 깨운 후 스캐쥴러에 CPU를 점유하는 것이 좀 더 공평함