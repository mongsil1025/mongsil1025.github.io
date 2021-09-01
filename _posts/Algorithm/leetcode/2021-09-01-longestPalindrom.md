---
layout: archive
title:  "[LeetCode] Longest Palindromic Substring"
date: 2021-08-18
excerpt: ""
tags: [algorithm, leetcode]
categories: [algorithm/leetcode]
---

## Problem

Given a string s, return the longest palindromic substring in s.

```
babad // return 3 (bab / aba)
cbbd // return 2 (bb)
```

## Solution : Expand around Center

[Algorithm Details](https://docs.google.com/presentation/d/1Hz1hKC_8QtKSIsnBTFN0nBWOLiHUziGaxI0l-qOFUB0/edit#slide=id.p)

Use 2 pointers that start from me and the indices between me. Then **Expand Around the center (start index)**

O(n2) 의 시간 복잡도를 가지며 O(1) 의 공간 복잡도를 가진다.

여기서 포인트는, 기준점이 되는 index와 palindrome 의 길이를 이용해서 palindrome 인 string 을 다시 계산하는 것이다.

``` java
int max = Math.max(len1, len2);
if(max > end - start) {
    start = i - ((max  - 1) / 2);
    end = i + (max / 2);
}
```

길이가 3인 palindrom 이 max인 상태에서, 어떤 문자열이 그에 해당하는지 까지 구하고 싶다면

내 index가 중간인 상태에서 길이가 3인 string 이 palindrome 이라는 뜻이니까
[index - (len - 1)/2 , index + (len/2)] 까지의 길이가 그에 해당할 것이다.

len - 1 을 하는 이유는, 내 자신을 포함해서 왼쪽 인덱스를 계산해야 하기 때문에 하나를 줄여준다.

---

[소스코드](https://github.com/mongsil1025/algorithm/blob/master/src/algorithm/leetCode/LongestPalindromicSubstring.java)
[참고 유튜브](https://www.youtube.com/watch?v=y2BD4MJqV20)
