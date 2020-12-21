---
layout: archive
title:  "[백준] 프린터큐"
date: 2020-11-13
excerpt: ""
tags: [algorithm, backjun]
categories: [algorithm/backjun]
classes: wide
---

[#백준 1966](https://www.acmicpc.net/problem/1966)
{: .fs-5 }

<div class="code-example" markdown="1">
큐
{: .label }

그리디
{: .label .label-green }

백준
{: .label .label-yellow }

easy
{: .label .label-purple }
</div>

<!--more-->

## 문제
상근이는 새로운 프린터기 내부 소프트웨어를 개발하였는데, 이 프린터기는 다음과 같은 조건에 따라 인쇄를 하게 된다.

현재 Queue의 가장 앞에 있는 문서의 ‘중요도’를 확인한다.
나머지 문서들 중 현재 문서보다 중요도가 높은 문서가 하나라도 있다면, 이 문서를 인쇄하지 않고 Queue의 가장 뒤에 재배치 한다. 그렇지 않다면 바로 인쇄를 한다.

<div class="code-example" markdown="1">
입력
{: .fw-700 .fs-5 }

첫 줄에 test case의 수가 주어진다. 각 test case에 대해서 문서의 수 N(100이하)와 몇 번째로 인쇄되었는지 궁금한 문서가 현재 Queue의 어떤 위치에 있는지를 알려주는 M(0이상 N미만)이 주어진다. 다음줄에 N개 문서의 중요도가 주어지는데, 중요도는 1 이상 9 이하이다. 중요도가 같은 문서가 여러 개 있을 수도 있다. 위의 예는 N=4, M=0(A문서가 궁금하다면), 중요도는 2 1 4 3이 된다.
{: .fs-3 }
</div>
```
3
1 0
5
4 2
1 2 3 4
6 0
1 1 9 1 1 1
```

<div class="code-example" markdown="1">
출력
{: .fw-700 .fs-5 }

각 test case에 대해 문서가 몇 번째로 인쇄되는지 출력한다.
{: .fs-3 }
</div>
```
1
2
5
```

---
## 문제풀이

어려운 문제는 아니다. 입력받은 중요도와, 찾아야하는 문서의 인덱스를 튜플로 구성하고, 리스트에 넣은 뒤, 문제를 풀면 된다.

다만, 입력받을 경우 튜플을 구성하는 파이썬 방법과, max 값을 구할 수 있는 `람다(lambda)` 표현식은 익숙해져야겠다.

<div class="code-example" markdown="1">
#### 1. enumerate

- 반복문 사용 시 몇 번째 반복문인지 확인이 필요할 수 있습니다. 이때 사용합니다.
- 인덱스 번호와 컬렉션의 원소를 `tuple`형태로 반환합니다.


- `for문`으로 리스트에 넣는 방법


``` python
for i in range(len(print_weight)):
    print_list.append((print_weight[i], i == F))
```

- `enumerate` 활용

``` python
print_list = [(i, idx) for idx, i in enumerate(print_weight)]
```

<br/>

#### 2. lambda를 이용한 min max

``` python
if print_list[0][0] == max(print_list, key=lambda x:x[0])[0]:
```

첫번째 값 (가중치)가 MAX인 튜플의 값을 가져온다. [람다표현식]({{ site.baseurl }}/2020/11/07/lambda/)에 익숙해져야겠다.
{: .fs-3}
</div>

### 소스코드

``` python
test_case = int(input())

while test_case > 0:
    N, F = list(map(int, input().split(' ')))

    print_weight = list(map(int, input().split(' ')))
    print_list = [(i, idx) for idx, i in enumerate(print_weight)]

    count = 0

    while len(print_list) > 0:

        if print_list[0][0] == max(print_list, key=lambda x:x[0])[0]:
            count += 1
            if print_list[0][1] == F:
                print(count)
                break
            else:
                print_list.pop(0)
        else:
            print_list.append(print_list.pop(0))

    test_case -= 1
```
