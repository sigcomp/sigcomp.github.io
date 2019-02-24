---
layout: page
title: Boss Battle - Difficulty Level 1.8
---

## Explanation

The question asks for the number of bombs thrown _in the worst case_. In this case the bomb is thrown two pillars in front of the boss, and subsequent bomb throws are away from the boss's initial location. The boss moves in the same direction that the bombs are being thrown. This is best demonstrated visually. In the image below, grey circles represent the bomb-affected pillars and the orange circle is the location of the boss.

![visual of boss battle](/assets/solution_img/boss_battle/diagram1.png "Boss battle visualization")

Now, visualize the circle unravelling into a straight line. It would look like the following image.

![visual of boss battle](/assets/solution_img/boss_battle/diagram2.png "Flatten out the circles")

Let n be the number of pillars. Notice that, given the way the circles have been numbered, the boss starts at position n - 2. Let x represent how many bombs have been thrown so far. Then the expression for where the bomb is targeted at is 2x. The expression for where the boss is at is x + n - 2. Setting the equations equal to each other will yield how many bombs must be thrown before the boss is finally hit: 2x = x + n - 2. Subtract x from both sides to get x = n - 2. So it will take n - 2 bombs being thrown to hit the boss in the worst case scenario. Graphically, this looks like the following image:

![graphical representation of boss battle](/assets/solution_img/boss_battle/graph1.png "graphical representation")

Note that for values less than 3, the boss will always be hit on the first throw, since the bomb radius is 3.

## Source Code

```c++
#include <iostream>

using namespace std;

int main()
{
    int n;
    cin >> n;

    int ans = n - 2;

    if (ans < 1) {
        ans = 1;
    }

    cout << ans << endl;

    return 0;
}

```