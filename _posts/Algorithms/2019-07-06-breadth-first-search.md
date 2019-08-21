---
layout: post
comments: true
title: 알고리즘 - 너비 우선 탐색(Breadth First Search)
category: Algorithms
shortinfo: 너비 우선 탐색에 대한 설명과 간단한 예제
tags:
- algorithms
---

## 알고리즘 - 너비 우선 탐색(Breadth First Search)

너비 우선 탐색은 최단 거리를 찾기 위해 사용된다. 예를 들어 아래와 같은 상황에 사용된다.

- 체커에서 가장 적은 수로 이길 수 있는 방법
- 단어에서 가장 적은 개수의 글자를 고쳐서 올바를 단어를 만드는 방법
- 가장 가까운 병원 찾기

너비 우선 탐색에서는 그래프를 이용하는데  node와 edge로 이루어져 연결의 집합을 모형화한 것이다. node A와 node B가 있을 때 먼저 node A에서 node B로 가는 경로가 있는지를 확인한 후에 최단 경로를 확인한다.



## 예제

친구 관계를 찾는 예제에서는 나의 친구와 친구의 친구를 모두 찾는데 너비 우선 탐색을 사용할 수 있다. 먼저 나의 친구들을 확인한다. 그 후 친구의 친구들을 확인하면 된다. 그래프를 먼저 확인하면 아래와 같다.

```PYTHON
friends = {
    '지은': ['영준', '태호', '유리', '세준'],
    '영준': ['지은', '세호', '유리'],
    '유리': ['지은', '영준'],
    '세준': ['지은', '한일', '민균'],
    '세호': ['영준'],
    '한일': ['세준'],
    '민균': ['세준']
}
```

지은이는 '영준', '태호', '유리', '세준' 4명의 친구가 있고 각 친구들의 친구 목록에도 '지은'과 각자의 친구들을 배열로 추가하고 friends 자체는 딕셔너리로 작성한다. 여기서  각각의 이름들이 node가 되고 그 node의 연결인 edge가 키, 값의 관계로 표현된 것이다. 친구의 친구도 친구 관계에 포함시킬 때, 유리의 친구는 '지은', '영준', '세호' 총 3명이 된다. 

너비 우선 탐색은 순서대로 탐색을 하게 되는데 이를 위해 queue가 사용된다. 위의 friends 그래프를 이용해 친구를 찾는 알고리즘을 구현하면 아래와 같이 된다.

```python
def find_friends(graph, me):
    qu = []
    done = set()
    qu.append(me)
    done.add(me)

    while qu:
        person = qu.pop(0)
        print(person)
        for friend in graph[person]:
            if friend not in done:
                qu.append(friend)
                done.add(friend)

friends = {
    '지은': ['영준', '유리'],
    '영준': ['지은', '세호', '유리'],
    '유리': ['지은', '영준'],
    '세준': ['한일', '민균'],
    '세호': ['영준'],
    '한일': ['세준'],
    '민균': ['세준']
}

find_friends(friends, '한일')		# 한일, 세준, 민균        
```

먼저 queue로 사용할 배열을 만들고 qu에 추가한 사람을 기록할 집합 done을 만든다. qu와 done에는 me에 해당하는 이름을 넣어준다. qu에 값이 있는 동안 친구의 목록을 확인하는데 print()를 통해 친구 이름을 출력한다. 먼저 for문을 돌면서 나의 친구가 qu에 추가되고 그 후 나의 친구의 친구가 또 for문에서 추가된다. for문 안에서 친구가 done에 있는지 확인하는데 set는 중복 허용이 안 되므로 qu와 done에 추가됐는지 확인할 수 있다. 



## deque 이용하기

파이썬의 collections에서는 양방향 queue인 deque를 제공한다.  deque에서 왼쪽부터 꺼내기 위해 popleft()를 사용하여 친구를 확인하며 나머지는 위와 동일하다.

```python
from collections import deque


def find_friends(graph, me):
    search_queue = deque()
    done = set()

    search_queue.append(me)
    done.add(me)

    while search_queue:
        person = search_queue.popleft()
        print(person)
        for friend in graph[person]:
            if friend not in done:
                search_queue.append(friend)
                done.add(friend)


friends = {
    '지은': ['영준', '유리'],
    '영준': ['지은', '세호', '유리'],
    '유리': ['지은', '영준'],
    '세준': ['한일', '민균'],
    '세호': ['영준'],
    '한일': ['세준'],
    '민균': ['세준']
}

find_friends(friends, '세호')		# 세호, 영준, 유리, 지은

```





###### 참조
그림으로 이해하는 알고리즘
모두의 알고리즘 with 파이썬