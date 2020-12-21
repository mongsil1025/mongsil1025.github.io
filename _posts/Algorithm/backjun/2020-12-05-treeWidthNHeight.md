---
layout: archive
title:  "[백준] 트리의 높이와 너비"
date: 2020-12-05
excerpt: ""
tags: [algorithm, backjun]
categories: [algorithm/backjun]
classes: wide
---

[#백준 1991](https://www.acmicpc.net/problem/1991)
{: .fs-5 }

<div class="code-example" markdown="1">
이진탐색
{: .label }

그래프
{: .label }

백준
{: .label .label-yellow }

hard
{: .label .label-purple }
</div>

<!--more-->
## 문제
![이진트리](https://onlinejudgeimages.s3-ap-northeast-1.amazonaws.com/upload/201008/ttt.PNG)

임의의 이진트리가 입력으로 주어질 때 너비가 가장 넓은 레벨과 그 레벨의 너비를 출력하는 프로그램을 작성하시오

<div class="code-example" markdown="1">
입력
{: .fw-700 .fs-5 }

첫째 줄에 노드의 개수를 나타내는 정수 N(1 ≤ N ≤ 10,000)이 주어진다. 다음 N개의 줄에는 각 줄마다 노드 번호와 해당 노드의 왼쪽 자식 노드와 오른쪽 자식 노드의 번호가 순서대로 주어진다. 노드들의 번호는 1부터 N까지이며, 자식이 없는 경우에는 자식 노드의 번호에 -1이 주어진다.
</div>
```
19
1 2 3
2 4 5
3 6 7
4 8 -1
5 9 10
6 11 12
7 13 -1
8 -1 -1
9 14 15
10 -1 -1
11 16 -1
12 -1 -1
13 17 -1
14 -1 -1
15 18 -1
16 -1 -1
17 -1 19
18 -1 -1
19 -1 -1
```

<div class="code-example" markdown="1">
출력
{: .fw-700 .fs-5 }

첫째 줄에 너비가 가장 넓은 레벨과 그 레벨의 너비를 순서대로 출력한다. 너비가 가장 넓은 레벨이 두 개 이상 있을 때에는 번호가 작은 레벨을 출력한다.
{: .fs-3 }
</div>
```
3 18
```

---

## `중위순회` 하면서 각 레벨의 min x 값과 max x 값을 저장한다.

x 축 기준으로 왼쪽부터 오른쪽까지 순차적으로 `열번호`가 매겨지는 것을 확인할 수 있다. 따라서 `중위순회`를 하며 열번호를 늘려가면 된다.

또한, 각 레벨마다 맨 왼쪽에 있는 열번호 (min x값)과 맨 오른쪽에 있는 열번호 (max x값)을 배열에 저장한다.

그 뒤에 마지막으로, 저장한 배열을 순회하며 max level을 찾으면 된다.

1. 중위순회

    중위 순회를 하기 앞서, 어떤 노드가 루트 노드인지 찾아야 한다. 그러기 위해서, `Node` class에 parent를 -1로 모두 초기화한뒤, 입력받은 값으로 그래프를 초기화 할때 parent를 갱신시켜준다.

    결과적으로 `parent=-1`인 노드가 루트 노드가 된다.

    ``` python
    root = -1
    for i in range(1, n + 1):
        if mygraph[i].parent == -1:
            root = i
    ```

    그리고 루트 노드부터 중위순회를 시작한다.
    - 중위 순회를 하면서, 이 트리가 몇 `level` (height)인지 갱신을 같이 해준다.
    - `level_min[N.size]`와 `level_max[N.size]`에 현재 level의 최소 x 값과 최대 x 값을 저장한다.

    ``` python
    in_order(mygraph[root], 1)

    def in_order(node, level):
        global x, level_depth

        level_depth = max(level_depth, level)

        if node.left != -1:
            in_order(mygraph[node.left], level + 1)

        x += 1
        level_min[level] = min(level_min[level], x)
        level_max[level] = max(level_max[level], x)

        if node.right != -1:
            in_order(mygraph[node.right], level + 1)
    ```

    <div class="alert alert-primary">    
    N개의 노드가 이진트리를 구성한다면 <code>log2(N) + 1</code>의 높이를 가진다. 이를 착안해서 level_min과 level_max 배열의 사이즈를 해당 높이를 구해서 구성하려고 했다. <br/>
    <code>
    height = math.ceil(math.log2(n)) + 1
    </code> <br/>
    하지만 문제의 트리는 균등한 이진탐색트리가 아닌, 보통의 이진 트리이기 때문에, n+1의 높이를 가질 수 있다는 것을 놓쳤다. <br/>

    <b>만약 균등이진 트리라면 위 방식대로 높이를 구하면 될 것 같다.</b>
    </div>


2. max 값 구하기

    구한 level_max와 level_min을 조회하며 너비가 제일 큰 level과 그 너비를 출력하면 된다.
    ```python
    result_level = 1
    result_width = 1

    for i in range(2, level_depth + 1):
        width = level_max[i] - level_min[i] + 1
        if result_width < width:
            result_width = width
            result_level = i
    ```

<i class="icon icon-link" style="display: inline-block;"></i>[GIT Source](https://github.com/mongsilJeong/fastcampus/blob/main/repAthmPractice/TreeWidthNHeight.py)
