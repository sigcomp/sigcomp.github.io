---
layout: page
title: Even Up Solitaire - Difficulty Level 2.7
---

### **Solved by Thomas McKanna on 10/05/2019.**

[**Link to the Kattis problem**](https://open.kattis.com/problems/evenup)

## Problem

Given a sequence of positive integers, keep taking away of adjacent pairs which sum to an even number until you can't anymore. Print out how many will be left over.

## Solution

This is a stack problem. We look at each of the numbers in order: 

1. If the stack is empty, push the number onto the stack.
2. If the number is even and the top of the stack is even or the number is odd and the top of the stack is odd, pop the stack.
3. If the number is even and the top of the stack is odd or the number is odd and the top of the stack is even, push the number onto the stack.

Then print out the size of the stack.

## Source Code

```c++
#include <iostream>
#include <stack>

using namespace std;

int main() {
    stack<int> nums;
    int n, t;

    cin >> n;
    
    for (int i = 0; i < n; i++) {
        cin >> t;
        t %= 2;

        if (nums.empty() || nums.top() != t) {
            nums.push(t);
        } else {
            nums.pop();
        }
    }

    cout << nums.size() << endl;
}
```