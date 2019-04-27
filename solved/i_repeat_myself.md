---
layout: page
title: I Repeat Myself I Repeat Myself I Repeat - Difficulty Level 2.4
---

## Problem Breakdown
> For every test case, print a single output line giving the length of the shortest pattern that explains the given input string.

Essentially, the problem asks you to find the shortest pattern the repeats from the beggining of the string to the end of the string, and possibly, the pattern is cut off on its final iteration. Take, for instance, the following string:

`abbcabbcabbabbcabb`

For this string, the longest "repeated" pattern is `abbcabbcabb` since it is followed by `abbcabb`, which follows the beginning of the pattern, but is cut off (hence the quotes around repeated). The length of the pattern is 11, and so that would be the answer in this case.

As another example, consider the following string:

`I Repeat Myself I Repeat Myself I Repeat`

The pattern in this case is `I Repeat Myself `, which is repeated twice and only partially completing during its last iteration.

## Solution Explanation
The problem can be solved in linear time (i.e. increasing the size of the string to find a pattern in will increase the run time by a constant factor) by using the following method. Make two variables to store indeces and a variable to store the candidate pattern (this will store our current best guess for the pattern). Then proceed as visualized below:

![graphical representation of boss battle](/assets/solution_img/i_repeat_myself/i_repeat_gif.gif "visual representation of algorithm")

As search continues one character at a time through the string, it is checked whether of not the characters pointed to by both indeces match. If they do not, the candidate string is updated to the string up until the index of the right-most marker, and the other marker is set to zero. This process is continued until all the characters of the string have been examined.

There is one important special case that must be checked for. If the character which differs happens to be the same character at the beginning of the candidate string, the candidate string should only be updated until *just before* the right-most marker. This is because it may be the beginning of the pattern, as is the case in the GIF above.

## Source Code

```python
n = int(input())
for _ in range(n):
    s = input()
    candidate = s[0]
    left = 0 # left index marker
    right = 0 # right index marker
    while right < len(s):
        if s[left % len(candidate)] != s[right]:
            # special case: non-match is same as start of pattern
            if s[right] == candidate[0]:
                candidate = s[:right] # don't include non-match
                right -= 1 # counter-act moving forward
            else:
                candidate = s[:right + 1]
            left = 0 # start matching pattern at beginning again
        else:
            left += 1 # move left marker forward
        right += 1 # move right marker forward
    print(len(candidate))
```
