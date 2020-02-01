---
layout: post
comments: true
title: 객체지향 프로그래밍(17강 - 23강)
category: Java
shortinfo: 배열, 다차원 배열, ArrayList 정리
tags:
- java
---





## 배열

- 동일한 자료형의 순차적 자료 구조

- 선언

  ```java
  int[] arr = new int[10];
  int arr[] = new int[10];
  ```

- 메모리 구조
  
  - 위의 예제에서 배열 길이는 10이 되고 배열 전체는 int가 4 byte이므로 총 40 byte가 됨

- 단점: 길이가 정해져 있음
- 장점: index 사용 편리



## 참고 자료형 배열(객체 배열)

- 참고 자료형 배열은 선언 후에는 null 값이 들어있음



## 객체 배열 복사

- 얕은 복사: 주소값 복사(System.arraycopy)
- 깊은 복사: 새로운 배열 생성



## 다차원 배열

- 2차원 이상의 배열

- 지도, 게임, 평면이나 공간을 구현할 때 사용

  ```java
  // 2행 3열의 다차원 배열
  
  int[][] arr = new int[2][3];
  ```



## ArrayList

- 자바에서 제공되는 객체 배열이 구현된 클래스

- 객체 배열을 사용하는데 필요한 여러 메서드 구현

  | 메서드              | 설명                                                |
  | ------------------- | --------------------------------------------------- |
  | boolean add(E e)    | 요소 하나를 배열에 추가. E는 요소의 자료형 의미     |
  | int size()          | 배열에 추가된 요소 전체 개수 반환                   |
  | E get(int index)    | 배열의 index 위치에 있는 요소 값 반환               |
  | E remove(int index) | 배열의 index 위치의 요소 값을 제거하고 그 값을 반환 |
  | boolean isEmpty()   | 배열이 비어있는지 확인                              |

  