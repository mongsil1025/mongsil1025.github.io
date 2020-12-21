---
layout: archive
title:  "[백준] 중량제한"
date: 2020-11-13
excerpt: ""
tags: [algorithm, backjun]
categories: [algorithm/backjun]
classes: wide
---

[#백준 1939](https://www.acmicpc.net/problem/1939)
{: .fs-5 }

<div class="code-example" markdown="1">
이진탐색
{: .label }

그래프
{: .label }

백준
{: .label .label-yellow }

medium
{: .label .label-purple }
</div>

<!--more-->
## 문제
N(2≤N≤10,000)개의 섬으로 이루어진 나라가 있다. 이들 중 몇 개의 섬 사이에는 다리가 설치되어 있어서 차들이 다닐 수 있다.

영식 중공업에서는 두 개의 섬에 공장을 세워 두고 물품을 생산하는 일을 하고 있다. 물품을 생산하다 보면 공장에서 다른 공장으로 생산 중이던 물품을 수송해야 할 일이 생기곤 한다. 그런데 각각의 다리마다 중량제한이 있기 때문에 무턱대고 물품을 옮길 순 없다. 만약 중량제한을 초과하는 양의 물품이 다리를 지나게 되면 다리가 무너지게 된다.

한 번의 이동에서 옮길 수 있는 물품들의 중량의 최댓값을 구하는 프로그램을 작성하시오.

<div class="code-example" markdown="1">
입력
{: .fw-700 .fs-5 }

첫째 줄에 N, M(1≤M≤100,000)이 주어진다. 다음 M개의 줄에는 다리에 대한 정보를 나타내는 세 정수 A, B(1≤A, B≤N), C(1≤C≤1,000,000,000)가 주어진다. 이는 A번 섬과 B번 섬 사이에 중량제한이 C인 다리가 존재한다는 의미이다. 서로 같은 두 도시 사이에 여러 개의 다리가 있을 수도 있으며, 모든 다리는 양방향이다. 마지막 줄에는 공장이 위치해 있는 섬의 번호를 나타내는 서로 다른 두 정수가 주어진다. 공장이 있는 두 섬을 연결하는 경로는 항상 존재하는 데이터만 입력으로 주어진다.
</div>
```
3 3
1 2 2
3 1 3
2 3 2
1 3
```

<div class="code-example" markdown="1">
출력
{: .fw-700 .fs-5 }

첫째 줄에 답을 출력한다.
{: .fs-3 }
</div>
```
3
```

---

## 이진탐색으로 최대 중량을 구한다.

- C의 크기가 10억이므로 이를 O(logn)으로 만들어야한다는 힌트를 얻을 수 있다.
- c의 크기를 이진 탐색하면서, 해당 중량으로 공장이 있는 섬을 방문할 수 있는지 확인한다.

```python
while start <= end:
    mid = (start + end) // 2
    # 이 가중치로 공장을 방문할 수 있는지 확인
    if bfs(mid):
        result = mid
        start = mid + 1
    else:
        end = mid - 1
```


## BFS로 공장이 있는 섬을 방문하는지 확인한다.

- 입력받은 간선을 `vice-versa`의 쌍으로 구성을 한다.
- 그래야만 연결된 모든 간선을 반복적으로 `visit`할 수 있다.

<div class="bd-callout bd-callout-warning">
<h5 id="jquery-incompatibility">BFS</h5>
<p>
1. 인접한 노드를 [Key-Value] vice-versa 쌍으로 가지고 있다.<br/>
2. 방문해야할 노드를 저장할 queue와, 방문을 했다는것을 표현할 list를 준비한다. <br/>
3. need_visit 큐에서 선입선출로 노드를 빼와서 순회한다. <br/>
4. need_visit 큐에 원소가 없어질때 까지 반복한다.
</p>
</div>

1. dict() 구조가 아니라, 배열의 인덱스를 노드로 활용한다.

    ``` python
    adj = [[] for _ in range(n+1)]  # 첫번째 노드를 인덱스로 사용할 것이기 때문에 n+1

    for _ in range(m):
        x, y, weight = map(int, input().split(' '))

        # 모든 연결된 간선을 탐색하기 위해서
        # vice-versa로 간선 추가

        adj[x].append((y, weight))
        adj[y].append((x, weight))
    ```

2. 파이썬의 deque를 활용해서 queue를 만든다.

    - start_node부터 시작한다.
    - BFS는 너비우선탐색 (형제 노드들을 먼저 탐색)이므로 큐로 구현

    ``` python
    def bfs(c):
        # 첫번째 노드부터 시작한다.
        queue = deque([start_node])
        visited = [False] * (n + 1)
        visited[start_node] = True
        while queue:
            x = queue.popleft()  # BFS는 큐로 구현하므로, 선입선출로 구현
            for y, weight in adj[x]:
                if not visited[y] and weight >= c:
                    visited[y] = True
                    queue.append(y)
        return visited[end_node]
    ```

<i class="icon icon-link" style="display: inline-block;"></i>[GIT Source](https://github.com/mongsilJeong/fastcampus/blob/main/repAthmPractice/LimitWeight.py)
