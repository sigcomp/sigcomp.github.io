---
layout: page
title: Pokemon Go Go - Difficulty Level 5.8
---

### **Solved by Adam Gausmann on 09/29/2019.**

[**Link to the Kattis problem**](https://open.kattis.com/problems/pokemongogo)

## Problem

Given a list of available Pok&eacute;mon at various locations
(Pok&eacute;Stops), find the shortest route to catch one of every
Pok&eacute;mon species and return to the starting point.

There is an upper bound on the number of unique Pok&eacute;mon species (15) and
on the total number of Pok&eacute;mon available (20).

## Solution

This problem is related to the [traveling purchaser problem], a
generalization of the traveling salesman problem where the goal is to visit
markets to buy the needed items while minimizing the total cost of traveling
and purchasing items.

For this problem, the markets are the locations of the Pok&eacute;mon and
the items are the species of Pok&eacute;mon; the cost of traveling is the
[Manhattan distance] between locations and the cost of "purchasing"
Pok&eacute;mon is zero.

A naive brute force search over all potential solutions has
a time complexity of $$O(n!)$$. This is not good enough for our range of
input, but we can do better.

The [Held-Karp algorithm] is an algorithm that applies dynamic programming to
obtain an exact solution to the TSP with a time complexity of $$O(2^nn^2)$$ and
a space complexity of $$O(2^nn)$$. It is derived from the property
that every subpath of an optimal path is itself optimal. One formulation of the
algorithm is as follows:

> Let $$V = \{0, 1, ..., n\}$$ be a set of destinations where $$s = 0$$ is the
> start and end point, and $$c_{ij}$$ be the cost of traveling from $$i$$ to
> to $$j$$. Then the cost function is:
>
> $$f_k(S, j) = \min_{m \in S - \{j\}} (f_{k-1}(S - \{m\}, k) + c_{kj})$$
> $$f_1(\{j\}, j) = c_{sj}$$
>
> where $$S$$ is the set of destinations visited starting from $$s$$, and $$j$$
> is the last destination visited.

This is almost enough to solve the problem, but not quite. The goal is not to
find a circuit through all destinations, but instead find one that
obtains every required item. Formally, the goal of TSP would be finding
the value of $$f_{|V|}(V, s)$$; but instead I extend it like so:

> Let $$B = \{1, ..., m\}$$ be the set of required items, $$M(j) \subseteq
> B$$ be the items available at destination $$j$$, and $$P(S) =
> \cup_{s \in S}{M(s)}$$ be the items purchased after traveling to all
> destinations in S.

Using these additional values, I can now formulate the goal I want: Find some $$S
\subseteq V$$ that minimizes the cost $$f_{|S|}(S, s)$$
with the constraint $$P(S) = B$$.

We can apply this to an implementation of the Held-Karp algorithm fairly easily.
The algorithm generates optimal paths of a certain length and uses them to
generate longer optimal paths. It will eventually iterate over every subset of $$V$$, finding the optimal path for each one. During that iteration, I can also check every
path to see if it satisfies the constraint $$P(S) = B$$, where $$S$$ is
the set of destinations in that path. If so, I add it as a candidate solution.
At the end, I take the candidate solution with the minimum cost, and this
becomes the solution to the problem.

Note that this is not a solution to the more general traveling purchaser
problem. This algorithm depends on the purchase cost
being constant; if it were not, then it would additionally have to consider
where each item should be purchased, and the set of purchased items would become
part of the state space. In this case it does not matter (the cost is equal to
zero) so I chose a greedy solution that "purchases" at the first given
opportunity.

[traveling purchaser problem]: https://en.wikipedia.org/wiki/Traveling_purchaser_problem
[Manhattan distance]: https://en.wikipedia.org/wiki/Taxicab_geometry
[Held-Karp algorithm]: https://en.wikipedia.org/wiki/Held%E2%80%93Karp_algorithm

## Source Code

```rust
#![allow(dead_code)]

use std::collections::HashMap;
use std::cmp::min;
use std::io::{stdin, BufRead};
use std::iter::FromIterator;
use std::ops;

fn main() {
    // Parse input:
    let stdin = stdin();
    let handle = stdin.lock();
    let mut lines = handle.lines().map(Result::unwrap);

    while let Some(header) = lines.next() {
        let line_count = header.parse().unwrap();
        let mut pokestops = HashMap::with_capacity(20);
        let mut pokemon = HashMap::with_capacity(15);

        for line in lines.by_ref().take(line_count) {
            // Parse row, column, Pokemon name from line:
            let mut words = line.split_whitespace();
            let r: i16 = words.next().unwrap().parse().unwrap();
            let c: i16 = words.next().unwrap().parse().unwrap();
            let name = words.next().unwrap();

            // Convert Pokemon name to a unique integer:
            let len = pokemon.len();
            let x = *pokemon.entry(name.to_string()).or_insert(len);

            // Add this pokestop:
            pokestops.entry((r, c)).or_insert(BitSet::new()).insert(x);
        }

        let num_pokemon = pokemon.len();

        // Convert pokestops into a list of nodes, with node 0 being the origin/depot:
        let mut nodes = Vec::with_capacity(21);
        nodes.push(((0, 0), BitSet::new()));
        nodes.extend(pokestops);

        // Calculate the cost matrix for traveling from i to j:
        let c: Vec<Vec<i16>> = nodes
            .iter()
            .map(|&((ri, ci), _)| {
                nodes
                    .iter()
                    .map(|&((rj, cj), _)| (ri - rj).abs() + (ci - cj).abs())
                    .collect()
            })
            .collect();

        // Dynamic programming algorithm for traveling purchaser.
        let all_pokestops: BitSet = (1..nodes.len()).collect();
        let all_pokemon: BitSet = (0..num_pokemon).collect();

        let mut solution = i16::max_value();

        // Calculate the initial state values for k = 1:
        // f({j}, j) = (M(j), c[0][j])
        let mut f = StateValues::new();

        for j in 1..nodes.len() {
            let mut s = BitSet::new();
            s.insert(j);
            let b: BitSet = nodes[j].1;

            f.insert(s, j, b, c[0][j]);

            // In the trivial case where there is a single pokestop, this will set the solution value.
            if b == all_pokemon {
                solution = min(solution, c[0][j] + c[j][0]);
            }
        }


        // Work through successive stages of the state space:
        for k in 2..nodes.len() {
            // for every combination of pokestops with length k:
            // f(s, j) = min(f(s - {j}, i) + c[j][i] for i in s - {j})
            for s in all_pokestops.subset_len(k) {
                for j in s {
                    let mut s_minus_j = s;
                    s_minus_j.remove(j);

                    let (b, cost) = s_minus_j.into_iter()
                        .filter_map(|i| {
                            let (b, old_cost) = f.get(s_minus_j, i);
                            Some((b | nodes[j].1, old_cost + c[j][i]))
                        })
                        .min_by_key(|&(_, c)| c)
                        .unwrap();

                    f.insert(s, j, b, cost);

                    // Handle solution candidates:
                    if b == all_pokemon {
                        solution = min(solution, cost + c[j][0]);
                    }
                }
            }

            if k >= all_pokestops.len() {
                break;
            }
        }

        assert!(solution != i16::max_value());
        println!("{}", solution);
    }
}

/// A set with integer keys that tracks membership using bitfields.
#[derive(Debug, Copy, Clone, PartialEq, Eq, Hash, Default)]
struct BitSet(u32);

impl BitSet {
    fn new() -> BitSet {
        Default::default()
    }

    fn len(&self) -> usize {
        self.0.count_ones() as usize
    }

    fn is_empty(&self) -> bool {
        self.len() == 0
    } 
    fn clear(&mut self) {
        *self = BitSet::new();
    }

    fn contains(&self, x: usize) -> bool {
        self.0 & (1 << x) != 0
    }

    fn insert(&mut self, x: usize) {
        self.0 |= 1 << x;
    }

    fn remove(&mut self, x: usize) {
        self.0 &= !(1 << x);
    }

    fn subset_len(self, len: usize) -> SubsetLen {
        SubsetLen {
            len,
            acc: BitSet::new(),
            iter: self.into_iter(),
            stack: Vec::new(),
        }
    }
}

impl FromIterator<usize> for BitSet {
    fn from_iter<I>(iter: I) -> Self
    where
        I: IntoIterator<Item = usize>,
    {
        iter.into_iter().fold(BitSet::new(), |mut acc, x| { acc.insert(x); acc })
    }
}

impl IntoIterator for BitSet {
    type Item = usize;
    type IntoIter = IntoIter;

    fn into_iter(self) -> Self::IntoIter {
        IntoIter(self.0)
    }
}

/// An iterator over all the elements of a set.
#[derive(Debug, Copy, Clone)]
struct IntoIter(u32);

impl Iterator for IntoIter {
    type Item = usize;

    fn next(&mut self) -> Option<Self::Item> {
        if self.0 == 0 {
            None
        } else {
            let x = self.0.trailing_zeros() as usize;
            self.0 &= !(1 << x);
            Some(x)
        }
    }
}

impl ops::BitOr for BitSet {
    type Output = BitSet;

    fn bitor(self, rhs: Self) -> Self::Output {
        BitSet(self.0 | rhs.0)
    }
}

impl ops::BitAnd for BitSet {
    type Output = BitSet;

    fn bitand(self, rhs: Self) -> Self::Output {
        BitSet(self.0 & rhs.0)
    }
}

impl ops::Not for BitSet {
    type Output = BitSet;

    fn not(self) -> Self::Output {
        BitSet(!self.0)
    }
}

impl ops::Sub for BitSet {
    type Output = BitSet;

    fn sub(self, rhs: Self) -> Self::Output {
        self & !rhs
    }
}

/// An iterator over subsets of a set with some given length.
#[derive(Debug, Clone)]
struct SubsetLen {
    len: usize,
    acc: BitSet,
    iter: IntoIter,
    stack: Vec<(BitSet, IntoIter)>,
}

impl Iterator for SubsetLen {
    type Item = BitSet;

    fn next(&mut self) -> Option<Self::Item> {
        while self.acc.len() < self.len - 1 {
            if let Some(next) = self.iter.next() {
                self.stack.push((self.acc, self.iter));
                self.acc.insert(next);
            } else if let Some((acc, iter)) = self.stack.pop() {
                self.acc = acc;
                self.iter = iter;
            } else {
                return None;
            }
        }
        if let Some(next) = self.iter.next() {
            let mut acc = self.acc;
            acc.insert(next);
            Some(acc)
        } else if let Some((acc, iter)) = self.stack.pop() {
            self.acc = acc;
            self.iter = iter;
            self.next()
        } else {
            None
        }
    }
}

/// A map for the state value function that prioritizes efficient, constant-time lookup.
///
/// This is more efficient than `HashMap`/`BTreeMap` because those prioritize efficiency for
/// random, sparse data. Since the input parameters for the state space are relatively dense in
/// this problem, it makes more sense to define a mapping from the state space to an integer
/// range, and use that range as indices into an array.
#[derive(Debug, Clone)]
struct StateValues {
    map: Vec<(BitSet, i16)>,
}

impl StateValues {
    fn new() -> StateValues {
        StateValues {
            map: vec![Default::default(); state_size()],
        }
    }

    fn insert(&mut self, s: BitSet, i: usize, b: BitSet, c: i16) {
        self.map[to_idx(s, i)] = (b, c);
    }

    fn get(&self, s: BitSet, i: usize) -> (BitSet, i16) {
        self.map[to_idx(s, i)]
    }
}

fn state_size() -> usize {
    1 << (SET_BITS + IDX_BITS)
}

fn to_idx(s: BitSet, i: usize) -> usize {
    debug_assert!(s.0 < (1 << SET_BITS));
    debug_assert!(i < (1 << IDX_BITS));
    (s.0 << IDX_BITS) as usize | i
}

const SET_BITS: u32 = 21;
const IDX_BITS: u32 = 5;
```