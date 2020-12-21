---
layout: archive
title:  "[알고리즘] 실전 코딩테스트 준비 시 Note"
date: 2020-12-14
excerpt: ""
tags: [algorithm, basic]
categories: [algorithm/basic]
classes: wide
---

## 코딩테스트 SUMMARY

- 정렬 알고리즘 기법
	- 정렬해야할 수는 굉장히 많은데, 그 숫자가 정해져 있는 경우 [백준:수정렬하기3](https://www.acmicpc.net/problem/10989)
		- **계수정렬 (Counting Algorithm)** 이 유리하다.
			- 숫자를 바로 배열의 index로 취급해서, 등장한 counting을 센다.
			- 다시 그 배열을 순회하면서, counting 된만큼 print 한다.


- 입력받을 숫자가 너무 크다면
	- input() 이 아닌 `sys.stdin.readline()`를 쓴다.


- 모든 경우의 수를 구해서 문제를 풀 수 있을 때
	- 완전 탐색
		- 이중 for문
		- **재귀로 모든 케이스를 만든 후 재귀 종료시점에 조건 체크**
			예) IP 만들기, 0 만들기


- 순서가 정해져 있는 작업을 차례로 수행해야할 때, 순서를 결정해주는 알고리즘
	- **위상 알고리즘 (Topology Sort 알고리즘)** [백준:문제집](https://www.acmicpc.net/problem/1766)
    - 시간 복잡도 : O(V+E)
    - 우선순위큐 (heap) 사용
    - 진입차수 저장 (깊이) : 배열로 `depth` 저장
  - [백준:문제집:소스코드](https://github.com/mongsilJeong/fastcampus/blob/main/repAthmPractice/Question.py)


- 동적프로그래밍 문제를 풀기 위해서는 `점화식`을 세워야 한다.
	- D[i] = D[i-1] + D[i-2]
	- 주로 **배열**을 이용해서, 각 케이스별 구해야 하는 값을 갱신시키는 형태로 풀 수 있다.
	- 보통 구해야하는 값 (예:무게K일때의 최대값)을 가능 케이스마다 DP로 계속 갱신한다.
  - N에 따른 값의 규칙을 찾아서 점화식을 만든다. (피보나치처럼)
  - [백준:2110](2020/12/16/backnack.html)

- [Codility] 지문에서 Not Empty Array 라고 적혀있지 않으면 Empty Array에 대한 처리를 해줘야 한다.
