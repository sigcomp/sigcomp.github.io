---
layout: page
title: Magical 3 - Difficulty Level 5.9
---

### **Solved by Nicholas Jacobson on 05/30/2019.**

## Problem

For up to 1000 values of $$n$$ such that $$0 \le n \le 2^{31}$$

Print the smallest base $$b$$ where $$n$$ in base $$b$$ ends in 3 or "no such base" if $$b$$ does not exist.

Input ends with $$n = 0$$

CPU time limit of 5 seconds

##  Explanation

For $$n$$ in a base $$b$$ to have a 3, $$b\ge4$$

$$n$$ in base $$b$$ is the same as:

$$n=c_0b^0+c_1b^1+c_2b^2+\ldots+c_{\log_bn}b^{\log_bn}$$

Where $$c_0 = 3$$ thus:

$$n=3b^0+c_1b^1+c_2b^2+\ldots+c_{\log_bn}b^{\log_bn}$$

$$n=3+b(c_1b^0+c_2b^1+\ldots+c_{\log_bn}b^{\log_bn-1})$$

If $$k=c_1b^0+c_2b^2+\ldots+c_{\log_bn}b^{\log_bn-1}$$ then:

$$n = 3+bk$$

$$n-3=bk$$

The edge case of no base being found occurs when  $$n-3=bk$$ is false.

Taking the minimum values for $$b$$ and $$k$$ give us $$n-3\ge4$$ or $$n\ge7$$

The edge case occurs when $$n<7$$ excluding $$3$$.

$$b$$ and $$k$$ are factors of $$n-3$$ and integers where $$b\ge4$$, $$k\ge1$$

All possible factors for $$n$$ are in  $$[1, n]$$, but all pairs of factors can be found by using factors in $$[1, \sqrt n]$$ to get a factor in $$[\sqrt n, n]$$. We only need $$[1, \sqrt n]$$ to find the smallest base $$b$$. This is done by checking $$k\in[1, 3]$$ and checking $$b \in [4, \sqrt n]$$.

## Time Complexity

Hypothetical worst case occurs when there is 1000 lines of input where each value is $$2^{31}$$ and the smallest base $$b=\sqrt n$$.

Assuming $$10^8$$ operations per second and that $$2^{10} \approx 10^3$$.

Taking total operations over speed we get:

$$10^3*\sqrt {2^{31}}/10^8 = 2^{15.5}/10^5 =(2^{10})^{1.55}/10^5 \approx (10^3)^{1.55}/10^5 = 10^{4.65}/10^5 = 1/10^{.35}$$
$$\approx .4467$$ seconds < 5 seconds

The worst case takes less than 5 seconds and will operate under the CPU time limit.

## Source Code

``` cpp
#include <iostream>
#include <cmath>
using namespace std;

int main()
{
    int num;
    int base;
    
    while( cin>>num )
    {
        if( num == 0 ){
            break;
        }
        else if( num == 3 ){
            cout<<4<<endl;
        }
        else if( num < 7 ){
            cout<<"No such base"<<endl;
        }
        else{
            base = num-3;
            
            // check if num-3 has 2 or 3 as factors, updating base if needed
            if( (num-3)%2 == 0 && (num-3)/2 >= 4 ){
                base = (num-3)/2;
            }
            if( (num-3)%3 == 0 && (num-3)/3 >= 4){
                base = (num-3)/3;
            }
            // check all numbers from 4 to square root of num-3 for factors
            // updating base if needed
            int end = pow(num-3, .5);
            for(int i=4; i<=end; i++){
                if((num-3)%i == 0){
                        base = i;
                        break;
                }
            }
            cout<<base<<endl;
        }
    }
    return 0;
}

```

