# Reverse Integer

[https://leetcode.com/problems/reverse-integer/](https://leetcode.com/problems/reverse-integer/)

### Convert Integer to string

While there are some easy ways to manipulate the digits of integers using mods and floors, strings are much easier to reverse. So storing a version of x as a string is our first step.

### Negative Case

If the number is negative, we can't simply flip the string, as the negative would be at the end rather than the beginning. So we must get the avoid the negative, and then add it on after.

### Trailing 0s

If the number 120 is reversed to 021, the 0 is supposed to be removed. This will be done automatically when converting the string to an int, since ints don't show leading 0s.

### Result exceeds 32 bits

This is the part that takes up the most space. Before you output the result, you must compute if it is less than 32 bits. While this is easy, remember that limits are different for positive and negative numbers.

## Solution (Python 3)
```
class Solution:
    def reverse(self, x: int) -> int:
        numStr = str(x)
        
        if int(numStr) < 0:
            if -int(numStr[:0:-1]) < -2**31:
                return(0)
            else:
                return("-"+str(int(numStr[:0:-1])))
        else:
            if int(numStr[::-1]) > 2**31-1:
                return(0)
            else:
                return(str(int(numStr[::-1])))
```
