---
layout: archive
title:  "[LeetCode] Add Two Numbers"
date: 2021-08-01
excerpt: ""
tags: [algorithm, leetcode]
categories: [algorithm/leetcode]
---

## Problem

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

``` console
Input: l1 = [2,4,3], l2 = [5,6,4]
Output: [7,0,8]
Explanation: 342 + 465 = 807.
```

## Solution

> Iterate linked list with carry

- if node is null, set value as 0
- add new Node to linked list (answer)
  - This was the point. if I set **dummy head of the node**, I dont need to split the cases with if statement

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

        ListNode answer = null;
        ListNode next   = null;
        int carry       = 0;

        while(l1 != null || l2 != null) {

            int i1 = l1 != null ? l1.val : 0;
            int i2 = l2 != null ? l2.val : 0;

            if(answer == null) {
                answer = new ListNode((carry + i1 + i2) % 10, next); // init
            } else {
                if(next == null) {
                    next = new ListNode((carry + i1 + i2) % 10);    
                    answer.next = next;
                } else {
                    next.next = new ListNode((carry + i1 + i2) % 10);    
                    next = next.next;
                }
            }

            if((carry + i1 + i2) > 9) {
                carry = 1;
            } else {
                carry = 0;
            }   

            l1 = l1 != null ? l1.next : null;
            l2 = l2 != null ? l2.next : null;
        }

        if (carry > 0) {
            if(next == null) {
                next = new ListNode(1);
                answer.next = next;
            } else {
                next.next = new ListNode(1);    
            }
        }

        return answer;
    }
}
```

> Use dummy head of the list

As you can see below, we can shorten code comapre to previous version.
Note that we use a dummy head to simplify the code. Without a dummy head, you would have to write extra conditional statements to initialize the head's value.

``` java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null){
        int x = p != null ? p.val : 0;
        int y = q != null ? q.val : 0;
        int sum = carry + x + y;

        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;

        p = p != null ? p.next : null;
        q = q != null ? q.next : null;
    }

    if (carry > 0) {
        curr.next = new ListNode(carry);
    }

    return dummyHead.next; // just return linked list without dummy data
```
