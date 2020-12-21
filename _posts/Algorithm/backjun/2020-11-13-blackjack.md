---
layout: archive
title:  "[백준] 블랙잭"
date: 2020-11-13
excerpt: ""
tags: [algorithm, iterations, backjun]
categories: [algorithm/backjun]
classes: wide
---

[#백준 2798](https://www.acmicpc.net/problem/2798)
{: .fs-5 }

<div class="code-example" markdown="1">
배열
{: .label }

완전탐색
{: .label .label-green }

백준
{: .label .label-yellow }

easy
{: .label .label-purple }
</div>

<!--more-->

## 문제
제한된 시간 안에 N장의 카드 중에서 3장의 카드를 골라야 한다. 블랙잭 변형 게임이기 때문에, 플레이어가 고른 카드의 합은 M을 넘지 않으면서 M과 최대한 가깝게 만들어야 한다.

N장의 카드에 써져 있는 숫자가 주어졌을 때, M을 넘지 않으면서 M에 최대한 가까운 카드 3장의 합을 구해 출력하시오.

<div class="code-example" markdown="1">
입력
{: .fw-700 .fs-5 }

첫째 줄에 카드의 개수 N(3 ≤ N ≤ 100)과 M(10 ≤ M ≤ 300,000)이 주어진다. 둘째 줄에는 카드에 쓰여 있는 수가 주어지며, 이 값은 100,000을 넘지 않는다.
합이 M을 넘지 않는 카드 3장을 찾을 수 있는 경우만 입력으로 주어진다.
{: .fs-3 }
</div>
```
5 21
5 6 7 8 9
```

<div class="code-example" markdown="1">
출력
{: .fw-700 .fs-5 }

첫째 줄에 M을 넘지 않으면서 M에 최대한 가까운 카드 3장의 합을 출력한다.
{: .fs-3 }
</div>
```
21
```

---
## 문제풀이 핵심 아이디어

### 완전탐색
- 카드 중 3개씩 뽑는 모든 경우의 수는 C(n,3)이며, n은 최대 100
- 경우의 수가 많지 않기 때문에 `3중 반복문` 으로 문제를 푼다.

{% highlight python %}
n,m = list(map(int, input().split(' ')))
data = list(map(int, input().split(' ')))

sum = 0
for i in range(len(data)):
    for j in range(i+1,len(data)):
        for t in range(j+1,len(data)):
            modSum = data[i] + data[j] + data[t]
            if sum < modSum and modSum <= m:
                sum = modSum

print(sum)
{% endhighlight %}

경우의 수가 무수히 많을 경우에는 어떻게 할까?
**백트래킹** 을 사용해보자"

---

### 백트래킹
- `재귀`를 사용해서, 3번의 depth를 반복적으로 체크한다.
- 조건에 맞지 않으면 `backTrack` 해서 다른 카드를 선택한다.
- 공간복잡도는 완전탐색보다 크지만, N의 숫자가 커질때, 시간복잡도는 위의 알고리즘보다 개선 될 것으로 생각된다.

백트래킹 알고리즘 문제를 더 풀어보자.


{% highlight python %}
n,m = list(map(int, input().split(' ')))
data = list(map(int, input().split(' ')))

path = [0 for i in range(3)]  # 선택된 숫자
used = [False for i in range(n)]  # 선택 여부

answer = {
    'diff': 9999999,
    'sum' : 0,
    'selected': [0, 0, 0]
}

def DFS(depth):
    if depth == 3:
        tmpSum = path[0] + path[1] + path[2]
        if tmpSum > m: # max보다 크다면
            return
        else:
            diff = m - tmpSum
            if(diff < answer['diff']): # 더 근접하다는 뜻
                answer['diff'] = diff
                answer['sum'] = tmpSum
                answer['selected'][0] = path[0]
                answer['selected'][1] = path[1]
                answer['selected'][2] = path[2]
    else:
        for i in range(n):
            if(used[i] == False):
                path[depth] = data[i]
                used[i] = True
                # 재귀 호출 시작
                DFS(depth + 1)
                # 재귀 호출 종료
                used[i] = False
                path[depth] = 0

def solve():
    DFS(0)
    print(answer['sum'])

solve()
{% endhighlight %}
