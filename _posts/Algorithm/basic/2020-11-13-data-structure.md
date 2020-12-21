---
layout: archive
title:  "Data Structure (자료구조)"
date: 2020-11-13
excerpt: ""
tags: [algorithm, basic]
categories: [algorithm/basic]
classes: wide
---

1. TOC
{:toc}

---

알고리즘에서 꼭 필요한 자료구조

<!--more-->

## 배열 (Array List)

-	첫 데이터의 위치에서 `index`로 데이터에 접근할 수 있기 때문에 빠른 접근가능
-	`미리 최대 길이`를 설정해야 하기 때문에 데이터 추가/삭제가 어렵다.

---

## 큐 (Queue)

-	FIFO (First-In, First-Out)
{: .text-red-100 }
-	파이썬에서의 queue
    - Queue()

    ``` python
    import queue

    data_queue = queue.Queue()
    data_queue.put(1)
    data_queue.qsize()
    data_queue.get()
    ```

    - LifoQueue() : 나중에 입력된 데이터가 먼저 출력되는 구조 (Stack)

    ``` python
    import queue

    data_queue = queue.LifoQueue()
    ```

    - PriorityQueue() : 데이터마다 `우선순위`를 넣어서 `우선순위가 높은 순`으로 출력

    ``` python
    import queue

    data_queue = queue.PriorityQueue()
    data_queue.put((10, "korea"))
    data_queue.put((5, 1))
    ```
<div class="code-example" markdown="1">
어디에 큐가 많이 쓰일까?

1. 멀티 태스킹을 위한 프로세스 스케쥴링 방식을 구현하기 위해 많이 사용됨 (운영체제)
2. 프로세스 스케쥴링 방식 참고
    -	스케쥴링 : 프로세스가 생성되어 실행될 때 필요한 여러자원을 해당 프로세스에게 할당하는 작업, 대기시간 최소화와 최대한 공평하게 자원을 배분하는 것을 목적으로 둔다.
    -	여러 프로세스들은 Ready Queue에서 준비상태로 대기한다.
    -	운영체제는 CPU Scheduler를 통해 Ready Queue에 있는 프로세스를 실행한다.
    ![](https://media.vlpt.us/images/hax0r/post/2d7ab339-d0c4-4950-abd7-8fb77270d40a/Untitled-Diagram-150.png)
3. Apache Kafka의 메시지 Broadcast/ Consume 방식도 큐의 일환이다.
</div>

---

## 스택 (Stack)

-	Last-In, First-Out : 가장 나중에 쌓은 데이터를 가장 먼저 빼낼 수 있는 구조
{: .text-red-100 }
-	대표적인 스택의 활용
    1.	프르그래밍에서의 함수 동작 방식

        프로그램의 메모리는 다음 세가지 영역으로 나누어진다.
        -	`정적영역` : 프로그램이 실행을 시작할 때 일정 크기가 할당되어, 끝날 때까지 유지되는 메모리 영역으로 주로 전역변수나 정적변수가 저장되는 영역이다.
        -	`스택영역` : 함수의 매개변수와 지역변수들이 저장되는 영역이다. 스택은 함수가 호출될 때 할당되고 리턴될 때 Pop 한다.
        -	`힙 영역` : 동적 할당 메모리가 할당되는 메모리 영역이다.

            ex) 동영상 플레이를 힙영역에서 수행하고 메모리 반납


    2.	C/C++, Java에서 객체를 힙에 생성하고 나면 그 주소를 스택에 쌓는다. C++는 delete를 통해 개발자가 직접 메모리를 반환하게 하지만, Java에서는 GC가 그 역할을 한다.
        -	`Garbage Collector`가 스택에 있는 객체주소를 찾아서 힙에 접근한 뒤, 메모리를 해제한다.  

        <div class="alert alert-info">
            GC의 종류를 찾아보자.
        </div>

        -	스택은 캐싱되어 있어서 접근이 빠르지만, 힙 영역은 위와 같이 두 번 접근해야하므로 상대적으로 느리다.
        -	C/C++는 객체를 스택에 할당하는데, Java는 힙에 할당한다.
    3.	스택의 장단점
        -	장점 : 데이터의 저장/읽기 속도가 빠르다.
        -	단점 : 데이터의 최대 개수를 미리 정해야 한다. -> 저장 공간의 낭비

-   파이썬으로 구현한 Stack

``` python
stack_list = list()

def push(data):
    stack_list.append(data)

def pop():
    data = stack_list[-1]
    del stack_list[-1]
    return data
```

---

## 링크드 리스트 (Linked List)

-	필요할 때마다 요소를 추가/삭제할 수 있다.
-	각 노드가 `데이터`와 `포인터`를 가지고 한 줄로 연결되어 있는 자료구조이다.
-	ArrayList에 비해서 데이터의 추가나 삭제가 용이하지만, 인덱스가 없기에 특정 요소에 접근하기 위해서는 `순차탐색`이 필요하며 `탐색속도가 떨어진다`는 단점이 있다.


탐색 or 정렬을 자주한다면 **배열**, 데이터의 추가/삭제가 많은 경우에는 **연결리스트**

- **링크드 리스트 유형**
    1. 단방향 링크드 리스트
        - next 포인터를 가진다.
        <details markdown="1">
        <summary>소스보기</summary>

        ``` python
        class Node:
            def __init__(self, data, next=None):
                self.data = data
                self.next = next

        class LinkedList:
            def __init__(self, data):
                self.head = Node(data)

            def add(self, data):
                if self.head == '':
                    self.head = Node(data)
                else:
                    node = self.head
                    while node.next:
                        node = node.next
                    node.next = Node(data)

        def delete(self, data):
            if self.head == '':
                print ("해당 값을 가진 노드가 없습니다.")
                return

            if self.head.data == data:
                temp = self.head
                self.head = self.head.next
                del temp
            else:
                node = self.head
                while node.next:
                    if node.next.data == data:
                        temp = node.next
                        node.next = node.next.next
                        del temp
                        return
                    else:
                        node = node.next
        ```
        </details>
    2. 양방향 링크드리스트 (더블 링크드리스트)
        -  next, prev 포인터를 가진다.
        <details markdown="1">
        <summary>소스보기</summary>

        ``` python
        class Node:
            def __init__(self, data, prev=None, next=None):
                self.prev = prev
                self.data = data
                self.next = next

        class NodeMgmt:
            def __init__(self, data):
                self.head = Node(data)
                self.tail = self.head

            def insert_before(self, data, before_data):
                if self.head == None:
                    self.head = Node(data)
                    return True            
                else:
                    node = self.tail
                    while node.data != before_data:
                        node = node.prev
                        if node == None:
                            return False
                    new = Node(data)
                    before_new = node.prev
                    before_new.next = new
                    new.next = node
                    return True

            def insert_after(self, data, after_data):
                if self.head == None:
                    self.head = Node(data)
                    return True            
                else:
                    node = self.head
                    while node.data != after_data:
                        node = node.next
                        if node == None:
                            return False
                    new = Node(data)
                    after_new = node.next
                    new.next = after_new
                    new.prev = node
                    node.next = new
                    if new.next == None:
                        self.tail = new
                    return True

            def insert(self, data):
                if self.head == None:
                    self.head = Node(data)
                else:
                    node = self.head
                    while node.next:
                        node = node.next
                    new = Node(data)
                    node.next = new
                    new.prev = node
                    self.tail = new

            def desc(self):
                node = self.head
                while node:
                    print (node.data)
                    node = node.next
        ```
        </details>


---

## 해쉬 테이블 (Hash Table)

### 기본개념

-	Hash Table : Key에 Data를 저장하는 데이터 구조
Key를 통해 바로 데이터를 받아올 수 있으므로, 속도가 획기적으로 빠르다.
-	Hash : 임의 값을 고정 길이로 변환한 값 (정의된 고정 길이로 변환)
-	Hash Function : Key에 대해 산술 연산을 이용해 데이터 위치를 찾을 수 있는 함수
-	Hash Value / Hash Address : Key를 해싱함수로 연산해서, 해쉬 값을 알아내고 이 해쉬 값으로 해쉬 테이블에서 데이터 위치를 찾음

    ![](https://media.vlpt.us/images/inyong_pang/post/570764b2-cea4-4727-938f-ace8684b31a7/image.png)


- 파이썬으로 구현해본 Hash

```python
def storage_data(data, value):
    key = ord(data[0])
    hash_address = hash_func(key)
    hash_table[hash_address] = value
```

보통 <span class="text-blue-100">배열</span>로 미리 `Hash Table` 사이즈만큼 생성 후에 사용한다. 즉, <span class="red-200">공간을 더 늘려서 Hash를 만들고, 시간을 그만큼 단축</span>시킬 수 있다.

- Hash 테이블의 장단점과 주요 용도
    - 장점
        1. 데이터 저장/읽기 속도가 빠르다
        2. 해쉬는 키에 대한 데이터가 있는지 확인이 쉽다. (캐싱 용도로)
    - 단점
        1. 여러 키에 해당하는 주소가 동일할 경우 충돌을 해결하기 위한 별도 자료구조가 필요하다.
    - 주요 용도
        1. 검색이 많이 필요한 경우
        2. 저장, 삭제, 읽기가 빈번한 경우
        3. 캐쉬 구현 시 ( 중복 확인이 쉬움, 있냐 없냐에 대한 판단이 쉽다.)

---

### 충돌(Collision) 해결 알고리즘

<div class="code-example" markdown="1">
Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page](another-page).
</div>
```markdown
Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page](another-page).
```

---

## 트리 (Tree)

There are a number of specific typographic CSS classes that allow you to override default styling for font size, font weight, line height, and capitalization.


---

## 힙 (Heap)
