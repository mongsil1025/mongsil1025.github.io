---
layout: archive
title:  "[알고리즘] Union Find"
date: 2020-11-13
excerpt: ""
tags: [algorithm, basic]
categories: [algorithm/basic]
classes: wide
---

## Union-Find 알고리즘

`Disjoint Set`을 표현할 때 사용하는 알고리즘으로 트리 구조를 활용하는 알고리즘
간단하게, **노드들 중에 연결된 노드를 찾거나**, **노드들을 서로 연결할 때 (합칠 때) 사용**

<div class="alert alert-primary">
<b>Disjoint Set이란</b> <br/>

서로 중복되지 않는 부분 집합들로 나눠진 원소들에 대한 정보를 저장하고 조작하는 자료구조 <br/>
공통 원소가 없는 (서로소) 상호 배타적인 부분 집합들로 나눠진 원소들에 대한 자료구조를 의미함 <br/>
Disjoint Set = 서로소 집합 자료구조
</div>

#### 1. Union
두 개별 집합을 하나의 집합으로 합침, 두 트리를 하나의 트리로 만든다.
<div class="card mb-3">
    <img class="card-img-top" src="https://i.imgur.com/sncU4e6.png"/>
    <div class="card-body bg-light">
        <div class="card-text">
            자료출처:패스트캠퍼스
        </div>
    </div>
</div>


#### 2. Find
여러 노드가 존재할 때, 두 개의 노드를 선택해서, 현재 두 노드가 서로 같은 그래프에 속하는지 (부분집합인지) 판별하기 위해, 각 그룹의 최상단 원소(즉, 루트 노드)를 확인
<div class="card mb-3">
    <img class="card-img-top" src="https://i.imgur.com/soHinwt.png"/>
    <div class="card-body bg-light">
        <div class="card-text">
            자료출처:패스트캠퍼스
        </div>
    </div>
</div>

---

## Union-Find 알고리즘의 고려할 점

Union 순서에 따라서, 최악의 경우 트리가 아닌 링크드 리스트와 같은 형태가 될 수 있다.
이 때는 Find/Union 시 계산량이 O(N) 이 될 수 있으므로, 해당 문제를 해결하기 위해, `union-by-rank`, `path compression` 기법을 사용하는 것이 효과적이다.

#### 1. Union-by-Rank 기법
- 각 트리에 대해 높이(Rank)를 기억해 두고, Union 시 두 트리의 높이가 다르면, 높이가 큰 트리에 높이가 작은 트리를 붙인다.
![unionfind34](https://i.imgur.com/sIzllas.png)

- 높이가 `h-1`인 두 개의 트리(높이가 동일)를 합칠 때는, 한 쪽 트리 높이를 1 증가시켜주고, 다른 쪽 트리를 해당 트리에 붙인다. (어느쪽도 상관없다)
![unionfind3](https://i.imgur.com/gfnjDhO.png)

- 시간복잡도
  1. 높이가 `h`인 트리가 만들어지려면, 높이가 `h-1`인 두 개의 트리가 필요함 (그래야 Rank를 1개 증가시키니까)
  2. 높이가 `h-1`인 트리를 만들기 위해 최소 n개의 원소가 필요하다면, 높이가 h인 트리가 만들어지기 위해서는 최소 2n개의 원소가 필요하다. (이진 트리의 구조)
  3. 따라서, `Union-by-Rank` 기법을 사용하면, 루트 노드를 찾아가는 시간 복잡도는 O(n)이 아닌 𝑂(𝑙𝑜𝑔𝑁) 로 낮출 수 있음


#### 2. Path compression
Find를 실행한 노드에서 거쳐간 노드를 루트에 다이렉트로 연결하는 기법
-> 거쳐간 노드 모두를 루트에 다이렉트로 연결해서 트리의 높이가 2인 자료구조를 만든다.
그래서 Find를 실행한 노드는 이후부터는 루트 노드를 한번에 알 수 있다.
![unionfind5](https://i.imgur.com/IuKWQc8.png)


---


``` python
parent = dict() # 루트 노드를 계속 찾기 위해서
rank = dict()

# 근데 이렇게되면, 명확하게 어떻게 트리가 구성되어 있는지 모르겠다.
# parent Dictionory가 항상 루트 노드만 가지고 있는 구조가 될 것 같다.

def find(node):
    # path compression 기법
    if parent[node] != node: # 위에 루트노드가 있다는 것
        parent[node] = find(parent[node])
    return parent[node]
    # 1. 루트노드를 반환
    # 2. 반환한 루트노드를 자식노드의 바로 위 parent로 만든다.


def union(node_v, node_u):
    root1 = find(node_v)
    root2 = find(node_u)

    # union-by-rank 기법
    if rank[root1] > rank[root2]:
        parent[root2] = root1 # root1이 더 랭크가 크니까
    else:
        parent[root1] = root2 # 연결을 해줌 parent['D'] = 'F'
        if rank[root1] == rank[root2]: # 랭크가 동일할 경우
            rank[root2] += 1 # 어느 한쪽의 랭크를 올려준다.
```

Path compression을 수행하게 되면, `parent` Dictionary 자료형에서는 항상 루트노드만 가지고 있다. 결국 `h=2` 인 트리형태로 만들어지게 되어, 다이렉트로 노드를 찾을 수 있다.
