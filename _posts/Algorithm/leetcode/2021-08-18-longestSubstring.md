---
layout: archive
title:  "[LeetCode] Longest Substring Without Repeating Characters"
date: 2021-08-18
excerpt: ""
tags: [algorithm, leetcode]
categories: [algorithm/leetcode]
---

## Problem

Given a string s, find the length of the longest substring without repeating characters.

``` console
Input: s = "abcabcbb"
Output: 3
Explanation: The answer is "abc", with the length of 3.

Input: s = "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
```

## Solution

### First Approach : Use array to cash Characters

First, to find the duplicated character, I used 128 size of `boolean[]` array to store true/false data


Second, if found duplicated data which means if `char_set[c]` return true, break loop and return the size (of substring)

``` java
public int getSubstringSize(String s) {

    boolean[] char_set = new boolean[128];
    int size = 0;

    for(char c : s.toCharArray()) {
        if(char_set[c]) {
            break;
        } else {
            char_set[c] = true;
            size ++;
        }
    }

    return size;
}
```

Finally, iterate this `getSubstringSize` function until end of the string

``` java
public int lengthOfLongestSubstring(String s) {
    int answer = 0;
    int length = s.length();
    for(int i=0; i<length; i++) {
        int size = getSubstringSize(s.substring(i));
        if (size == length) {
            answer = size;
            break;
        }
        answer = answer < size ? size : answer;
    }
    return answer;
}
```

This Solution accepted, but it's not efficient because iterate N size of string at `getSubstringSize` and loop this function N times O(n2).

So leetcode shows below efficieny result

Runtime: 584 ms
Memory Usage: 39.7 MB
{: .notice}


### Second Approach : Use Two Pointers

As I used above, using boolean array `boolean[128]` was good approach, but for reducing complexity, we have to get rid of `for` statement.

the basic idea is, keep this array which stores occurance of the characters in string, but change `boolean to int` to store their **positions**


And use **two pointers** which define the max substring.
  - move the right pointer to scan through the string (basic loop O(N))
  - and meanwhile update the array.


If the character is already in the array (duplicated), then move the left pointer(j) to the right of the same character last found.

``` java
public int lengthOfLongestSubstring(String s) {
    int result = 0;
    int[] cache = new int[128];
    for (int i = 0, j = 0; i < s.length(); i++) {
        j = (cache[s.charAt(i)] > 0) ? Math.max(j, cache[s.charAt(i)]) : j;
        cache[s.charAt(i)] = i + 1;
        result = Math.max(result, i - j + 1);
    }
    return result;
}
```

:white_check_mark: The point was "character behind j index is already proved to be not duplicated"
So that we can just get length of substring by `i - j + 1`

[here shows detail process by images](https://docs.google.com/presentation/d/1VEJthAqNO7sUe1g0T110UkQDI9xiJGAfzxucoqHsXFY/edit?usp=sharing)

Its effitiency is dramatically improved :blush:

Runtime: 2 ms
Memory Usage: 39 MB
{: .notice}
