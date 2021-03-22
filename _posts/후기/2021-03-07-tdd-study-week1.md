---
layout: post
comments: true
title: TDD, Clean Code 스터디 1주차
category: 후기
shortinfo: TDD 스터디 1주차
tags:
- java
---





## 스터디를 시작하며

테스트 코드를 짜는 게 실력 향상에 도움이 많이 된다는 말을 보고 전 회사에 있을 때 혼자서라도 테스트 코드를 짜며 TDD를 해보려 했지만 테스트 코드를 짜는 것 자체가 쉽지 않았다. 어떤 걸 테스트 해야하는지, 어떻게 테스트 해야하는지에 대한 고민이 많았다. TDD 방식대로 테스트 코드를 먼저 짜고, 그 코드가 실패한 후 테스트 코드가 성공하도록 하고 또 리팩토링 하는 방식도 익숙치 않아서 쉽지 않았다. 

그러던 와중에 TDD 스터디가 있는 걸 알게 되서 신청이 열리자마자 참가하게 되었다. 현재가 11기이니 그간 많은 사람들이 이 스터디를 거쳐갔다는 말인데 이제라도 알게 된 게 다행이라는 생각이 든다. 스터디는 자바로 진행되지만 이 스터디를 마친 후에는 언어에 상관없이 테스트 코드를 짜고 TDD 방식대로 개발하는 것이 익숙해지는 것이 목표이다.

총 8주간 진행되는데 그간 기수들의 평균 완료율이 20 ~30% 정도 된다. 생각보다 낮은 숫자인데 갈수록 어려워지는 것도 이유 중 하나라고 하니 약간은 걱정이 된다. 하지만 혹시라도 8주간에 끝내지 못해도 리뷰어분께 부탁해서 꼭 코드 리뷰를 받고 끝까지 하겠다고 마음 먹었다.

스터디는 오픈소스에 기여하는 방식으로 진행됐다. 넥스트스텝의 레포지토리에서 포크한 후, 과제를 하고 PR을 하고 피드백을 받고 수정하는 과정이 계속 된다. 개발을 배우게 되면서 개발 문화라는 것을 알게 되고 코드 리뷰를 한다는 것도 알게 됐다. 꼭 코드 리뷰를 하는 회사에 들어가고 싶었는데 스터디에서라도 하게 돼서 정말 설레고 기뻤다. 별로 물어보거나 할 말이 없을 줄 알았는데 과제를 하면서 궁금했던 부분이 많았고 작은 거라도 리뷰어와 얘기하고 싶어서 많이 물어보고 대화하게 되었다. 처음으로 하는 코드리뷰라서 이 모든 과정이 즐거운데 리뷰어분께도 그런 시간이 되었으면 좋겠다.



## 1단계

1단계는 스터디가 시작되기 전 assertj와 git의 기본적인 사용 방식에 대해 익혔다. 자주 사용했던 `isEqualTo()` 외에도 실용적인 메서드를 알게 되었다. 예제에서 result는 1, 2, 3, 4를 포함하고 있으므로 `contains`에는 포함하는 값만 넣으면 되지만 `containsExactly` 에는 포함하는 모든 값을 넣어야만 한다. 만약 `containsExactly("1", "2", "3")` 식으로 쓴다면 테스트는 실패하게 된다.

```java
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class TestEx {
  
  @Test
  void arrayTest() {
    String[] result = "1,2,3,4".split(",");
    assertThat(result).contains("1", "2");
    assertThat(result).containsExactly("1", "2", "3", "4");
  }
}
```

테스트코드를 짤 때 예외 테스트를 항상 헷갈려 했는데 1단계에서 짚고 가서 좋았다. 메서드 이름만 봐도 이해하기 쉽다. `assertThatThrownBy()` 메서드 안에 예외를 일으키는 코드를 작성하고 `isInstanceOf()` 에서 어떤 예외인지 확인한다.

```java
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThatThrownBy;

public class TestEx {
  
  @Test
  void charTest() {
    String str = "abc";
    int index = 5;
    
    assertThatThrownBy(() -> s.charAt(index))
      .isInstanceOf(StringIndexOutOfBoundsException.class)
      .hasMessageContaining(String.format("String is out of range: %d", index));
  }
}
```

항상 @Test만 쓰다가 정말 유용한 걸 배우게 되었다. `@ParameterizedTest` 를 이용해서 테스트 할 값을 넘길 수 있는데 귀찮게 일일이 확인했던 걸 편하게 해준다. `@ValueSource` 를 이용하면 테스트 메서드에 매개변수로 넘어가서 그 값들을 확인한다. `@CsvSource`도 비슷한데 예제처럼 테스트 할 값과 결과를 비교할 때 사용하면 좋다. `,` 를 기준으로 값을 분리하며 `delimiter` 설정은 변경할 수 있다.

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
import org.junit.jupiter.params.provider.ValueSource;

public class TestEx {
  
  private Set<Integer> numbers;

    @BeforeEach
    void setUp() {
        numbers = new HashSet<>();
        numbers.add(1);
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
    }
  
  	@ParameterizedTest @ValueSource(ints = {1, 2, 3})
  	void contains(int number) {
        assertThat(numbers.contains(number)).isTrue();
    }

    @ParameterizedTest @CsvSource(value = {"1,true", "2,true", "3,true", "4,false", "5,false"})
    void containsFalse(int number, boolean expected) {
        assertThat(numbers.contains(number)).isEqualTo(expected);
    }  
}
```



## 2단계

2단계에서부터는 정말 과제를 하고 코드리뷰를 통해 피드백을 받고 수정하는 과정이 시작됐다. 문자열 계산기를 구현하고 테스트 코드를 짜는 과제였는데 처음 구현한 것과 머지됐을 때의 차이를 보면 새로운 걸 배우게 된 게 보여서 정말 기뻤다. 더 잘하지 못한 게 아쉽지만 앞으로의 과정에서 계속 발전해갈 수 있을거란 생각이 든다. 

가장 기억에 남는 건 `else` 를 사용하지 않도록 구현하는 것이었다. 처음에는 어떻게 해야하지 싶다가 그래도 요구사항을 지키려 생각해보니 떠오르긴 했었다. 리뷰어분의 피드백이 큰 도움이 되었다. 질문한 것들에 대해서 명확하게 답변해주셔서 정말 감사했다. 

변수에 대해서도 많이 고민하게 된 과제였다. static 필드를 잘 쓰지 못했는데 코드리뷰를 받으니 부족한 점을 확실히 알게 됐다. 괜히 코드 리뷰를 하는 게 아니구나라는 걸 깨달았다. 2단계를 하면서 받았던 피드백들은 3단계에서는 같은 실수를 반복하지 않고 알게 된 것들을 적용할 수 있는 상황이면 꼭 적용해봐야겠다.