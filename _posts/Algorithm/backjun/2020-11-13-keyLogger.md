---
layout: archive
title:  "[백준] 키로거"
date: 2020-11-13
excerpt: ""
tags: [algorithm, stack, backjun]
categories: [algorithm/backjun]
classes: wide
---

[#백준 5397](https://www.acmicpc.net/problem/5397)
{: .fs-5 }

<div class="code-example" markdown="1">
스택
{: .label }

링크드 리스트
{: .label }

자료구조
{: .label .label-green }

백준
{: .label .label-yellow }

medium
{: .label .label-purple }
</div>

<!--more-->


## 문제
키로거는 사용자가 키보드를 누른 명령을 모두 기록한다. 따라서, 강산이가 비밀번호를 입력할 때, 화살표나 백스페이스를 입력해도 정확한 비밀번호를 알아낼 수 있다.

강산이가 비밀번호 창에서 입력한 키가 주어졌을 때, 강산이의 비밀번호를 알아내는 프로그램을 작성하시오.

<div class="code-example" markdown="1">
입력
{: .fw-700 .fs-5 }

첫째 줄에 테스트 케이스의 개수가 주어진다. 각 테스트 케이스는 한줄로 이루어져 있고, 강산이가 입력한 순서대로 길이가 L인 문자열이 주어진다. **(1 ≤ L의 길이 ≤ 1,000,000)**
'-'가 주어진다. 이때 커서의 바로 앞에 글자가 존재한다면, 그 글자를 지운다. 화살표의 입력은 '<'와 '>'로 주어진다.
{: .fs-3 }
</div>
```
2
<<BP<A>>Cd-
ThIsIsS3Cr3t
```

<div class="code-example" markdown="1">
출력
{: .fw-700 .fs-5 }

각 테스트 케이스에 대해서, 강산이의 비밀번호를 출력한다. 비밀번호의 길이는 항상 0보다 크다.
{: .fs-3 }
</div>
```
BAPC
ThIsIsS3Cr3t
```

---
## Result

| Used Algorithm        | Time Solved          | Result |
|:-------------|:------------------|:------|
| Linked List (mine)           | 1h | <span class="text-red-200">Fail</span>|
| Stack (mine) | 1h   | <span class="text-red-200">Fail</span>  |
| 2 Stack | -   | <span class="text-green-200">Success</span>  |


입력 문자열 크기가 최대 1,000,000이므로 시뮬레이션 방식으로는 문제를 해결할 수 없었다. (시간제한 오류)
따라서, `O(n)` 시간복잡도 안에 해결해야 하는 문제였다.

1. **링크드 리스트 사용**
    - '<', '>' 커서로 문자열의 위치를 접근해야 했다.
    - '-' 데이터의 삭제가 필요했고, 데이터의 삽입이 필요했다.
    - 따라서, 나는 링크드 리스트를 채택해서, 문제를 해결했다.

**시간제한 오류** <br/>커서가 움직이면서, 원하는 특정 위치에 데이터를 삽입해야하는데, 링크드 리스트의 경우 특정 위치까지 가려면, list를 `순차탐색`해서 해당 인덱스 까지 가야한다. 따라서 이 부분에서 최악의 경우 `O(n2)` 시간 복잡도가 된다.

2. **런타임 에러**
    - 어김없이 런타임 에러가 났다.

**배열 사용 시 index에 주의하자** <br/> 배열 사용 시 `런타임에러`의 주요 원인은 index 초과이다.

---
## 문제 풀이

#### My Solution (Fail)

<div class="code-example" markdown="1">
- answer을 링크드리스트로 구현했다.
- 원소가 추가될 때마다 `pointer ++`
- '<' 입력 시 `pointer --`
- '>' 입력 시 `pointer ++`
- '-' 입력 시 `pop` (값이 있을 경우에만)
- pointer (인덱스) 위치까지 링크드리스트에 접근해서 데이터 삽입
    <details markdown="1">
    <summary>소스보기</summary>


    ``` python

    # 링크드 리스트 사용

    class Node:
        def __init__(self, data, next=None):
            self.data = data
            self.next = next


    class LinkedList:
        def __init__(self):
            self.head = None

            self.size = 0

        def add(self, data, index):
            if self.head is None:
                self.head = Node(data)
            else:
                node = self.head

                i = 1
                while i < index:
                    node = node.next
                    i += 1

                temp = node.next
                node.next = Node(data)
                node.next.next = temp

            self.size += 1

        def delete(self, index):
            node = self.head
            if node:  # 값이 있을 경우
                if index == 1:
                    temp = self.head
                    self.head = temp.next
                    del temp
                else:

                    i = 1
                    while i < index - 1:
                        node = node.next
                        i += 1

                    # node.next 가 삭제대상 노드
                    if node.next:
                        temp = node.next
                        node.next = temp.next
                        del temp
                    else:
                        node.next = None

            self.size -= 1

        def getSize(self):
            return self.size

        def print(self):
            node = self.head
            answer_list = list()
            while node:
                answer_list.append(node.data)
                node = node.next
            print(''.join(answer_list))


    def my_solution(pwd):
        pointer = 0

        answer = LinkedList()
        for i in range(len(pwd)):
            data = pwd[i]
            if data == '<':
                if pointer > 0:
                    pointer -= 1
            elif data == '>':
                if pointer < answer.getSize():
                    pointer += 1
            elif data == '-':
                if answer.head is not None:
                    answer.delete(pointer)
                    pointer -= 1
            else:
                answer.add(pwd[i], pointer)
                pointer += 1

        answer.print()

    ```
</div>

#### Lecture Solution

<div class="code-example" markdown="1">
- 두 개의 스택을 사용한다.
- 원소가 추가될 때마다 `left_stack`에 쌓는다.
- '<' 입력 시 `right_stack`에 값을 옮긴다.
- '>' 입력 시 `left_stack`에 값을 옮긴다.
- '-' 입력 시 `pop` (값이 있을 경우에만)
- 모든 문자열을 순환했을 경우, `extend()` 함수로 두 스택을 합친다.
    <details markdown="1">
    <summary>소스보기</summary>

    ``` python

    left_stack = []
    right_stack = []

    for i in pwd:
        if i == '-':
            if left_stack: # 런타임 에러 가능성 (배열)
                left_stack.pop()
        elif i == '<':
            if left_stack:
                right_stack.append(left_stack.pop())
        elif i == '>':
            if right_stack:
                left_stack.append(right_stack.pop())
        else:
            left_stack.append(i)

    left_stack.extend(reversed(right_stack))
    print(''.join(left_stack))

    ```
</div>
