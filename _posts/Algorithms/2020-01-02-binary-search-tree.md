---
layout: post
comments: true
title: 알고리즘 - 이진 탐색 트리(Binary Sesarch Tree)
category: Algorithms
shortinfo: 이진 탐색 트리에 대한 설명과 간단한 예제
tags:
- algorithms
---





## 이진 검색 트리(Binary Search Tree)

이진 검색 트리는 노드를 정렬된 순서로 유지하는 자료구조로서 이진 트리로 이루어져 있으며 아래의 특징을 가지고 있다.

- 노드의 왼쪽 하위 트리에는 노드의 값보다 작은 값의 노드만 존재
- 노드의 오른쪽 하위 트리에는 노드의 값보다 큰 값의 노드만 존재
- 왼쪽과 오른쪽 하위 트리 모두 이진 탐색 트리
- 중복 노드 없음

왼쪽 하위 트리와 오른쪽 하위 트리의 높이 차가 같은 균형 트리일 경우에는 노드 검색, 삽입, 삭제에 대한 시간 복잡도는 O(log n)이 된다.



### 구현

```python
# Binary Tree


class NodeBT:
    def __init__(self, value=None, level=1):
        self.value = value
        self.level = level
        self.left = None
        self.right = None

    def _add_next_node(self, value, level_here=2):
        new_node = NodeBT(value, level_here)
        if not self.value:
            self.value = new_node
        elif not self.left:
            self.left = new_node
        elif not self.right:
            self.right = new_node
        else:
            self.left = self.left._add_next_node(value, level_here + 1)
        return self

    def _search_for_node(self, value):
        if self.value == value:
            return self
        else:
            found = None
            if self.left:
                found = self.left._search_for_node(value)
            if self.right:
                found = found or self.right._search_for_node(value)
            return found


class BinaryTree:

    def __init__(self):
        self.root = None

    def add_node(self, value):
        if not self.root:
            self.root = NodeBT(value)
        else:
            self.root._add_next_node(value)
```

```python
# Binary Search Tree


class NodeBST(NodeBT):

    def __init__(self, value=None, level=1):
        self.value = value
        self.level = level
        self.left = None
        self.right = None

    def _add_next_node(self, value, level_here=2):
        new_node = NodeBST(value, level_here)
        if value > self.value:
            self.right = self.right and self.right._add_next_node(
                value, level_here + 1) or new_node
        elif value < self.value:
            self.left = self.left and self.left._add_next_node(
                value, level_here + 1) or new_node
        else:
            print("중복 노드 허용하지 않음")
        return self

    def _search_for_node(self, value):
        if self.value == value:
            return self
        elif self.left and value < self.value:
            return self.left._search_for_node(value)
        elif self.right and value > self.value:
            return self.right._search_for_node(value)
        else:
            return False


class BinarySearchTree(BinaryTree):
    def __init__(self):
        self.root = None

    def add_node(self, value):
        if not self.root:
            self.root = NodeBST(value)
        else:
            self.root._add_next_node(value)
```





###### 참조
파이썬 자료구조와 알고리즘
