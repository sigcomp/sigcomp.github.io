---
layout: page
title: Treehouses - Difficulty Level 4.8
---

Author: Thomas McKanna

## Problem Breakdown

The scenario is this: there is a forest surrounded by open land. Both the forest
and the open land have treehouses, some of which are already connected by cables.
The goal is to ensure all treehouses are reachable from any given treehouse by
adding the minimum amount of cabling necessary to achieve this goal.

We are given which edges are in the open land and which edges are in the forest.

## Solution

There are a number of ideas you must be familiar with in order to understand
this solution:

1. Heap/Priortiy Queue ([resource](https://en.wikipedia.org/wiki/Heap_(data_structure)))
2. Disjoint Sets ([resource](https://en.wikipedia.org/wiki/Disjoint_sets))
3. Minimum Spanning Trees ([resource](https://en.wikipedia.org/wiki/Minimum_spanning_tree))
4. Kruskal's Algorithm ([resource](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm))

Check out the linked resources for an introduction to these concepts.

The input to this problem can be thought of as a partially completed 
(and potentially improper) execution of Kruskal's algorithm. The reason
it is "potentially improper" is because Kruskal's algorithm
will never produce a cycle in the graph, but the edges given in the input may
have cycles - a possibility that must be addressed in the solution.

In addition, there is an added twist to this problem in that all of the treehouses
in the open land are considered connected. For the purposes of the problem, they
can all be considered to be one giant, single treehouse.

Start by creating some variables: `effective_edges` to store the number of non-cycle-creating
edges, `effective_vertices` to store the number of vertices when treating all
of the treehouses in the open land as a single treehouse, and a dictionary `position_map` to map each treehouse to its
position. Now, read in the first e treehouses (which are in the open land), adding
them all to a single set. Then read in the remaining n - e treehouses, adding each to 
its own new set. Add all treehouses to `position_map` so that distances can
be calcualted later.

Then, each of the pre-existing p edges are read in and dealt with as follows:
if the two treehouses are in different sets, union those sets and increment
`effective_edges`. In this way, we ignore any edges that create cycles.

Now, make a new min-heap `potential_edges`, and add all the possible edges (edges
between each pair of treehouses) to the heap. The weight of each edge should 
be the distance between the two treehouses. Note that this part is what takes
longest in the algorithms (O(|V|^2), where |V| is the number of treehouses).

Finally, make a variable to store the answer and then continuously pop from the 
min-heap until the number of `effective_edges` is equal to the number of 
`effective_vertices - 1` (recall that for a tree, the number of edges is the 
number of vertices minus one). The criteria for whether or not an edge is accepted 
is the same criteria as Kruskal's algorithm: if the two treehouses of the edge are in different 
sets, accept the edge by added the distant of the edge to the answer, unioning 
the two sets, and incrementing `effective_edges`.

To visualize the use of disjoint sets in this solution, consider the following
example. We start with the set of outer treehouses and sets for each of six
treehouses that are in the forest, labeled A-F. Suppose that six cables already
exist between (A,B), (A,C), (C,D), (C,B), (E,F) and one from F to one of the treehouses
in the open land. Refer to the images below to see how it plays out. Notice
that in image (e), both of the treehouses in edge (C,D) are already in the same
disjoint set, and so effective edges is not incremented.

In this example, since we end with 5 effective edges and their are 7 effective
vertices, we only need to add one more edge, the shortest edge the connected the two
remaining disjoint sets.

![a](/assets/solution_img/treehouses/a.png "a")
![b](/assets/solution_img/treehouses/b.png "b")
![c](/assets/solution_img/treehouses/c.png "c")
![d](/assets/solution_img/treehouses/d.png "d")
![e](/assets/solution_img/treehouses/e.png "e")
![f](/assets/solution_img/treehouses/f.png "f")
![g](/assets/solution_img/treehouses/g.png "g")


## Source Code

```python
import math
import heapq

class SetCollection:
    """ Used to store a collection of disjoint sets. """
    def __init__(self):
        self.sets = []

    def add_set(self, s):
        """ Adds an already existing set. """
        self.sets.append(s)

    def make_set(self, v):
        """ Makes a new set and puts v in it. """
        self.sets.append({v})

    def find_set(self, v):
        """ Returns the set which v is in, or None. """
        for s in self.sets:
            if v in s:
                return s
        return None

    def union(self, u, v):
        """ Unions the sets containing u with the set containing v. """
        s1 = self.find_set(u)
        s2 = self.find_set(v)
        if s1 is not s2:
            self.sets.append(set.union(s1, s2))
            self.sets.remove(s1)
            self.sets.remove(s2)

class Edge:
    """ Represents a cable between two treehouses. """
    def __init__(self, u, v, dist):
        self.u = u
        self.v = v
        self.dist = dist

    def __lt__(self, other):
        """ Definition of the < operator for use in priority queue. """
        return self.dist < other.dist

def distance(x1, y1, x2, y2):
    """ Calculates the distance between two coordinates. """
    return math.sqrt((x1 - x2)**2 + (y1 - y2)**2)

treehouses = SetCollection()
open_land_set = set() # Will contain first e treehouses (see problem statement)
position_map = {} # Maps (treehouse number) => (x, y)
ans = 0 # Will contain the minimum amount of cable needed to connect treehouses

n, e, p = map(int, input().split())

effective_edges = 0 # All edges minus ones that cause cycles
effective_vertices = n - e + 1 # Considers all of e to be a single vertex

# Reading in the first e treehouses (in open land)
for i in range(1, e + 1):
    x, y = map(float, input().split())
    position_map[i] = (x, y)
    open_land_set.add(i)

treehouses.add_set(open_land_set) # Add e to collection of sets

# Reading in the remaining treehouses (in forest)
for i in range(e + 1, n + 1):
    x, y = map(float, input().split())
    position_map[i] = (x, y)
    treehouses.make_set(i) # Add each treehouses to a new set

# Reading in the already existing cables
for _ in range(p):
    i, j = map(float, input().split())
    # Check if the two treehouses are in disjoint sets
    if treehouses.find_set(i) is not treehouses.find_set(j):
        treehouses.union(i, j)
        effective_edges += 1
    # else: they are not in disjoint sets => CYCLE! Do not count the edge.

potential_edges = [] # Will contain every possible edge combination

# Adding edges for *every* pair of treehouses (slowest part)
for i in position_map.keys():
    for j in position_map.keys():
        # Ignore edges to self
        if i != j:
            # Find distace between two treehouses
            dist = distance(
                position_map[i][0],
                position_map[i][1],
                position_map[j][0],
                position_map[j][1]
            )
            # Add newly created edge to the min-heap
            heapq.heappush(potential_edges, Edge(i, j, dist))

# Note: if a tree has n vertices, it must have n - 1 edges
while effective_edges != effective_vertices - 1:
    # Pop next shortest edge from heap
    curr_edge = heapq.heappop(potential_edges)
    u = curr_edge.u
    v = curr_edge.v
    # Check if the two treehouses are in disjoint sets
    if treehouses.find_set(u) is not treehouses.find_set(v):
        ans += curr_edge.dist # Add to the answer
        treehouses.union(u, v) # Combine the two disjoint sets
        effective_edges += 1 
    
print('{:.6f}'.format(ans)) # format answer to six decimal places
```