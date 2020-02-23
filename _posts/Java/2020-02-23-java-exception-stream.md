---
layout: post
comments: true
title: 객체지향 프로그래밍(62강 - 72rkd)
category: Java
shortinfo: 예외 처리, 스트림 정리
tags:
- java
---



## 예외 처리

#### 오류

- 컴파일 오류: 프로그램 코드 작성 중 발생하는 문법적 오류
- 실행 오류: 실행 중인 프로그램이 의도하지 않은 동작을 하거나(bug) 중지되는 오류(runtime error)
- 자바는 예외처리를 통해 프로그램의 비정상 종료를 막고 log를 남길 수 있음

#### 오류와 예외 클래스

시스템 오류(error)

- 가상 머신에서 발생
- 프로그래머가 처리 불가
- 동적 메모리를 다 사용한 경우
-  stack over flow 등

예외(Exception)

- 프로그램에서 제어할 수 있는 오류
- 읽으려는 파일이 없는 경우
- 네트워크나 소켓 연결 오류
- 자바에서는 예외에 대한 처리 수행
- 모든 예외 클래스의 최상위 클래스는 `Exception` 클래스

#### try - catch

```java
try {
    // 예외가 발생할 수 있는 코드 부분
} catch (처리할 예외 타입 e) {
    // try 블록 안에서 예외가 발생할 때 수행되는 부분
}
```

#### try - catch - finally

```java
try {
    // 예외가 발생할 수 있는 코드 부분
} catch(처리할 예외 타입 e) {
    // try 블록 안에서 예외가 발생할 때 수행되는 부분
} finally {
    // 예외 발생 여부와 상관없이 항상 수행되는 부분
    // 주로 리소스를 정리하는 코드
}
```

#### try - with -resources

- 리소스 자동 해제
- 해당 리소스가 `AutoCloseable`을 구현한 경우 `close()`를 명시적으로 호출하지 않아도 try {} 블록에서 오픈된 리소스는 정상적인 경우나 예외가 발생한 경우 모두 자동으로 close() 호출
- Java 7부터 제공
- FileInputStream의 경우 AutoCleaseable 구현

#### AutoCloseable

```java
public class AutoCloseObj implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.prinln("리소스가 close() 되었습니다.");
    }
}
```

#### 향상된 try - with - resources

- Java 9에서 제공
- Java 9 이전: try 안에 다른 참조 변수로 다시 선언해서 사용

  ```java
  AutoCloseObj obj = new AutoCloseObj();
  try (AutoCloseObj obj2 = obj) {
      throw new Exception();
  } catch (Exception e) {
      System.out.println("예외");
  }
  ```

- Java 9: try에 외부에서 선언한 변수 그대로 사용 가능

  ```java
  AutoCloseObj obj = new AutoCloseObj();
  try (obj) {
      throw new Exception();
  } catch (Exception e) {
      System.out.println("예외");
  }
  ```

#### 예외 처리 미루기

- `throws` 사용
- try{} 블록으로 처리하지 않고 메서드 선언부에 throws 추가
- 예외가 발생한 메서드에서 예외 처리를 하지 않고 이 메서드를 호출한 곳에서 예외 처리 함
- main()에서 throws를 사용하면 가상머신에서 처리

#### 다중 예외 처리

- 하나의 try{} 블록에서 여러 예외가 발생하는 경우, catch{} 블록 한 곳에서 처리하거나 여러 catch{} 블록으로 나누어 처리
- 가장 최상위 클래스인 `Exception` 클래스는 가장 마지막 블록에 위치


## 자바 입출력 스트림

#### 입출력 스트림

- 다양한 입출력 장치에 독립적으로 일관성 있는 입출력 방식 제공
- 입출력이 구현되는 곳에서는 모두 I/O 스트림을 사용
  - 키보드, 파일 디스크, 메모리 등
- I/O 대상 기준: 입력 스트림, 출력 스트림
- 자료의 종류: 바이트 스트림, 문자 스트림
- 스트림 기능: 기반 스트림, 보조 스트림

#### 입출력 스트림과 출력 스트림

- 입력 스트림: 대상으로부터 자료를 읽어 들이는 스트림
- 출력 스트림: 대상으로 자료를 출력하는 스트림

|    종류     | 예시                                                         |
| :---------: | ------------------------------------------------------------ |
| 입력 스트림 | FileInputStream, FileReader, BufferedInputStream, BufferedReader 등 |
| 출력 스트림 | FileOutputStream, FileWriter, BufferedOutputStream, BufferedWriter 등 |

#### 바이트 단위 스트림과 문자 단위 스트림

- 바이트 단위 스트림: 바이트 단위로 자료를 읽고 씀(동영상, 음악 파일 등)
- 문자 단위 스트림: 문자는 2바이트씩 처리해야 함

|     종류      | 예시                                                         |
| :-----------: | ------------------------------------------------------------ |
| 바이트 스트림 | FileInputStream, FileOutputStream, BufferedInputStream, BufferedOutputStream 등 |
|  문자 스트림  | FileReader, FileWriter, BufferedReader, BufferedWriter 등    |

#### 기반 스트림과 보조 스트림

- 기반 스트림: 대상에 직접 자료를 읽고 쓰는 기능의 스트림
- 보조 스트림: 직접 읽고 쓰는 기능은 없고 추가적인 기능을 제공해주는 스트림. 기반 스트림이나 또 다른 보조 스트림을 생성자의 매개변수로 포함

|    종류     | 예시                                                         |
| :---------: | ------------------------------------------------------------ |
| 기반 스트림 | FileInputStream, FileOutputStream, FileReader, FileWriter 등 |
| 보조 스트림 | InputStreamReader, OutputStreamWriter, BufferedInputStream, BufferedOutputStream 등 |

#### 표준 입출력

- System 클래스의 표준 입출력 멤버

  ```java
  public class System {
      public static PrintStream out; 	// 표준 출력 스트림 
      public static InputStream in;	// 표준 입력 스트림
      public static PrintStream err;	// 표준 에러 스트림
  }
  ```

#### System.in 사용하기

- 한 바이트씩 읽어들임
- 한글과 같은 여러 바이트로 된 문자를 읽기 위해서는 `InputStreamReader`와 같은 보조 스트림 필요

#### Scanner 클래스

- `java.util` 패키지에 있는 입력 클래스
- 문자뿐 아니라 정수, 실수 등 다양한 자료형을 읽을 수 있음
- 생성자가 다양하여 여러 소스로부터 자료를 읽을 수 있음

| 생성자                      | 설명                                          |
| --------------------------- | --------------------------------------------- |
| Scanner(File source)        | 파일을 매개변수로 받아 Scanner 생성           |
| Scanner(InputStream source) | 바이트 스트림을 매개변수로 받아  Scanner 생성 |
| Scanner(String source)      | String을 매개변수로 받아 Scanner 생성         |

#### Console 클래스

- System.in을 사용하지 않고 콘솔에서 표준 입출력

| 메서드                | 설명                                   |
| --------------------- | -------------------------------------- |
| String readLine()     | 문자열 읽기                            |
| char[] readPassword() | 사용자에게 문자열을 보여주지 않고 읽기 |
| Reader reader()       | Reader 클래스 반환                     |
| PrintWriter writer()  | PrintWriter 클래스 반환                |

#### 바이트 단위 스트림

- InputStream: 바이트 단위 입력 스트림 최상위 클래스
- OutputStream: 바이트 단위 출력 스트림 최상위 클래스
- 추상 메서드를 포함한 추상 클래스로 하위 클래스가 구현

| 스트림 클래스        | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| FileInputStream      | 파일에서 바이트 단위로 자료 읽기<br>파일이 없는 경우 예외 발생 |
| ByteArrayInputStream | Byte 배열 메모리에서 바이트 단위로 자료 읽기                 |
| FilterInputStream    | 기반 스트림에서 자료를 읽을 때 추가 기능을 제공하는 보조 스트림의 상위클래스 |

| 스트림 클래스         | 설명                                                         |
| --------------------- | ------------------------------------------------------------ |
| FileOutputStream      | 파일에서 바이트 단위로 자료 쓰기<br>파일이 없는 경우 파일 생성하여 출력 |
| ByteArrayOutputStream | Byte 배열 메모리에서 바이트 단위로 자료 쓰기                 |
| FilterOutputStream    | 기반 스트림에서 자료를 쓸 때 추가 기능을 제공하는 보조 스트림의 상위클래스 |

#### 문자 단위 스트림

- Reader: 문자 단위로 읽는 최상위 스트림
- Writer: 문자 단위로 쓰는 최상위 스트림
- 추상 메서드를 포함한 추상 클래스로 하위 클래스가 구현

| 스트림 클래스     | 설명                                                         |
| ----------------- | ------------------------------------------------------------ |
| FileReader        | 파일에서 문자 단위로 읽는 스트림 클래스                      |
| InputStreamReader | 바이트 단위로 읽은 자료를 문자로 변환하는 보조 스트림 클래스 |
| BufferedReader    | 문자로 읽을 때 배열을 제공하여 한꺼번에 읽을 수 있는 기능을 제공하는 보조 스트림 |

| 스트림 클래스      | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| FileWriter         | 파일에 문자 단위로 출력하는 스트림 클래스                    |
| OutputStreamWriter | 파일에 바이트 단위로 출력한 자료를 문자로 변환해주는 보조 스트림 |
| BufferedWriter     | 문자로 쓸 때 배열을 제공하여 한꺼번에 쓸 수 있는 기능을 제공하는 보조 스트림 |

#### 보조 스트림

- 실제 읽고 쓰는 스트림이 아닌 보조 기능을 추가하는 스트림
- `FilterInputStream`과 `FilterOutputStream`이 보조스트림의 상위 클래스
- 생성자의 매개 변수로 또 다른 스트림을 가짐

  | 생성자                                      | 설명                                      |
  | ------------------------------------------- | ----------------------------------------- |
  | protected FilterInputStream(InputStream in) | 생성자의 매개변수로 InputStream을 받는다  |
  | public FilterOutputStream(OuputStream out)  | 생성자의 매개변수로 OutputStream을 받는다 |

- 데코레이터 패턴(Decorator Pattern)
- Buffered stream: 내부에 8192 바이트 배열을 가지며 읽거나 쓸 때 속도가 빠름
- DataInputStream/DataOutputStream : 자료가 저장된 상태 그대로 자료형을 유지하며 읽거나 쓰는 기능 제공

#### 직렬화(Serialization)

- 인스턴스의 상태를 그대로 저장하거나 네트워크로 전송하고 이를 다시 복원(Deserialization) 하는 방식
-  `ObjectInputStream`과 `ObjectOutputStream` 사용
- 보조 스트림
- 직렬화는 인스턴스의 내용이 외부(파일 ,네트워크)로 유출되는 것이므로 개발자가 객체의 직렬화 가능 여부를 명시
- Serializable 인터페이스: 구현 코드가 없는 mark interface

  ```java
  class Example implements Serializable {
      
  }
  ```

#### 그 외 입출력 클래스

- File 클래스
  - 파일 개념을 추상화
  - 입출력 기능은 없고 파일의 속성, 경로, 이름 등을 알 수 있음
- RandomAccessFile 클래스
  - 입출력 클래스 중 유일하게 파일 입출력을 동시에 할 수 있는 클래스
  - 파일 포인터가 있어서 읽고 쓰는 위치의 이동이 가능
  - 다양한 자료형에 대한 메서드 제공

#### Decorator Pattern

- Java의 입출력 스트림은 데코레이터 패턴 사용
- 실제 입출력 기능을 가진 객체(component)와 그 외 다양한 기능을 제공하는 데코레이터(보조 스트림)을 사용하여 다양한 입출력 기능 구현
- 상속보다 유연한 확장성 가짐
- 지속적인 서비스의 증가와 제거 용이


