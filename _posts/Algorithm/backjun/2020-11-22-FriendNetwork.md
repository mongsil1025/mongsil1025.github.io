---
layout: archive
title:  "[백준] 친구 네트워크"
date: 2020-11-13
excerpt: ""
tags: [algorithm, backjun]
categories: [algorithm/backjun]
classes: wide
---

[#백준 4195](https://www.acmicpc.net/problem/4195)
{: .fs-5 }

<div class="code-example" markdown="1">
Hash
{: .label }

Union-Find
{: .label }

Graph
{: .label .label-green }

백준
{: .label .label-yellow }

medium
{: .label .label-purple }
</div>

<!--more-->


## 문제
어떤 사이트의 친구 관계가 생긴 순서대로 주어졌을 때, 두 사람의 친구 네트워크에 몇 명이 있는지 구하는 프로그램을 작성하시오.

친구 네트워크란 친구 관계만으로 이동할 수 있는 사이를 말한다.

<div class="code-example" markdown="1">
입력
{: .fw-700 .fs-5 }

첫째 줄에 테스트 케이스의 개수가 주어진다. 각 테스트 케이스의 첫째 줄에는 친구 관계의 수 F가 주어지며, 이 값은 100,000을 넘지 않는다. 다음 F개의 줄에는 친구 관계가 생긴 순서대로 주어진다. 친구 관계는 두 사용자의 아이디로 이루어져 있으며, 알파벳 대문자 또는 소문자로만 이루어진 길이 20 이하의 문자열이다.
{: .fs-3 }
</div>
```
2
3
Fred Barney
Barney Betty
Betty Wilma
3
Fred Barney
Betty Wilma
Barney Betty
```

<div class="code-example" markdown="1">
출력
{: .fw-700 .fs-5 }

**친구 관계가 생길 때마다**, 두 사람의 친구 네트워크에 몇 명이 있는지 구하는 프로그램을 작성하시오.
{: .fs-3 }
</div>
```
2
3
4
2
2
4
```

---
## Result

[Union-Find](/2020/11/18/union-find.html) 기법을 사용해서 푸는 문제이다.

<div class="alert alert-primary">
<b>Union-Find 특징</b> <br/>
find : dict() 형인 `parent`로 루트 노드를 갱신 <br/>
--> 여기서 find를 했을때만, 루트노드가 갱신되는 것이 특징이다.
union : 두 집합의 루트노드가 다를 경우, 합한다 (한쪽의 루트 노드를 다른 한쪽의 루트노드로 갱신한다.)
</div>

---
## 문제 풀이


``` python

test_case = int(input())

while test_case > 0:

    F = int(input())

    parent = dict()
    number = dict()

    def find(node):
        if node == parent[node]:
            return node
        else:
            p = find(parent[node])  # 거슬러 올라가면서 루트노드로 만들어줌
            parent[node] = p
            return parent[node]


    def union(x, y):
        x = find(x)
        y = find(y)

        if x != y:  # 둘이 같은 루트 노드가 아니라면 (부분집합이 아니라면)
            parent[y] = x
            number[x] += number[y]


    for i in range(F):
        u, v = input().split(' ')

        if u not in parent:
            parent[u] = u  # 초기화
            number[u] = 1
        if v not in parent:
            parent[v] = v
            number[v] = 1

        union(u, v)

        print(number[find(u)])
        print(parent)

    test_case -= 1

```
---
## More

이 문제는, 합집합의 `size`를 출력하면 되는 항목이다. 만약에 친구 네트워크가 어떻게 이루어졌는지 모두 출력하라는 문제였다면 어땠을까?

``` python
for i in range(F):
    u, v = input().split(' ')

    if u not in parent:
        parent[u] = u  # 초기화
        number[u] = 1
    if v not in parent:
        parent[v] = v
        number[v] = 1

    v1 = find(u)
    v2 = find(v)
    if v1 != v2:
        union(v1, v2)
        mst.append((u, v))

    print(number[find(u)])
    print(parent)
    print(mst)
```
