---
layout: page
title: Magical 3 - Difficulty Level 6.0
---

### **Solved by Nicholas Jacobson on 05/30/2019.**

## Source Code
```c++
#include <iostream>
#include <cmath>
using namespace std;

int main()
{
    int num;
    int base;
    
    while( cin>>num )
    {
        if( num == 0 )
        {
            break;
        }
        else if( num == 3 )
        {
            cout<<4<<endl;
        }
        else if( num < 7 )
        {
            cout<<"No such base"<<endl;
        }
        else
        {
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