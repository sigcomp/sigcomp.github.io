---
layout: page
title: March of the Penguins - Difficulty Level 4.7
---

### **Solved by Adam Gausmann on 10/05/2019.**

[**Link to the Kattis problem**](https://open.kattis.com/problems/marchofpenguins)

## Source Code

```rust
// March of the Penguins
// Solved using flow networks and the Edmonds-Karp maximum flow algorithm.

use std::f64::INFINITY;
use std::io::{stdin, BufRead};
use std::collections::VecDeque;

fn main() {
    // Parse input.

    let stdin = stdin();
    let handle = stdin.lock();
    let mut lines = handle.lines().map(Result::unwrap);

    let header = lines.next().unwrap();
    let mut words = header.split_whitespace();

    // Number of floes.
    let n: usize = words.next().unwrap().parse().unwrap();
    // Maximum jump distance.
    let d: f64 = words.next().unwrap().parse().unwrap();

    #[derive(Debug, Clone, Copy)]
    struct Floe {
        // X-coordinate.
        x: f64,
        // Y-coordinate.
        y: f64,
        // Maximum jumps supported.
        m: f64,
        // Number of penguins starting here.
        n: f64,
    }

    let floes: Vec<_> = lines
        .map(|line| {
            let mut words = line.split_whitespace();
            let x = words.next().unwrap().parse().unwrap();
            let y = words.next().unwrap().parse().unwrap();
            let n = words.next().unwrap().parse().unwrap();
            let m = words.next().unwrap().parse().unwrap();
            Floe { x, y, m, n }
        })
        .collect();
    let num_penguins = floes.iter().map(|floe| floe.n).sum();

    // Produce the flow network from the input data.
    //
    // Even though the flow is made of purely integer values, this
    // implementation uses floating-point values because of the special
    // "infinity" value that can represent infinite capacity.

    // Two nodes are needed for every floe (in and out), plus one source and
    // one sink.
    let node_count = 2*n + 2;

    // The source and sink node indices.
    let src = 0;
    let sink = 1;

    // The index of the "input" node for each floe.
    fn floe_in(i: usize) -> usize {
        2 * (i + 1)
    }

    // The index of the "output" node for each floe.
    fn floe_out(i: usize) -> usize {
        2 * (i + 1) + 1
    }

    // The flow capacity matrix. (cap[i][j] is flow capacity from i to j)
    let mut cap: Vec<Vec<_>> = vec![vec![0.0; node_count]; node_count];

    for (i, floe) in floes.iter().enumerate() {
        // The floe is provided with a flow capacity from the source equal to
        // the number of penguins it currently has.
        cap[src][floe_in(i)] = floe.n;

        // Connect this floe's input to it's output with a flow capacity equal
        // to the maximum supported jumps.
        cap[floe_in(i)][floe_out(i)] = floe.m;

        // For every other floe in range, connect this floe's output to that
        // floe's input with an infinite flow capacity.
        for (j, target) in floes.iter().enumerate() {
            if i != j && (floe.x - target.x).powi(2) + (floe.y - target.y).powi(2) <= d.powi(2) {
                cap[floe_out(i)][floe_in(j)] = INFINITY;
            }
        }
    }

    // Test each floe to see if every penguin can reach it.
    let mut results = Vec::new();
    for floe in 0..n {
        // Connect the target floe's input to the sink.
        cap[floe_in(floe)][sink] = INFINITY;

        // Flow state matrix (flow[i][j] is current flow from i to j).
        let mut flow: Vec<Vec<_>> = vec![vec![0.0; node_count]; node_count];
        // Accumulated maximum flow.
        let mut max_flow = 0.0;

        loop {
            // BFS: Find the shortest path from source to sink with some
            // residual flow.
            let mut q = VecDeque::new();
            q.push_back(src);
            let mut pred = vec![None; node_count];

            while let Some(i) = q.pop_front() {
                for j in 0..node_count {
                    if j != src && pred[j] == None && cap[i][j] > flow[i][j] {
                        pred[j] = Some(i);
                        q.push_back(j);
                    }
                }
            }

            // Check if a path was found.
            if pred[sink] == None {
                break;
            }

            // Find the bottleneck (smallest capacity edge) of this path.
            let mut df = INFINITY;
            let mut j = sink;
            while let Some(i) = pred[j] {
                df = df.min(cap[i][j] - flow[i][j]);
                j = i;
            }

            assert!(df < INFINITY);

            // Apply a flow equal to the bottleneck to all edges in the path.
            j = sink;
            while let Some(i) = pred[j] {
                flow[i][j] += df;
                flow[j][i] -= df;
                j = i;
            }

            // Update the accumulated flow.
            max_flow += df;
        }

        // Done with the algorithm - disconnect the target floe from the sink
        // to restore it for the next iteration.
        cap[floe_in(floe)][sink] = 0.0;

        // Max flow is equal to the number of penguins that could reach that floe.
        // The goal is to find floes that all penguins can reach.
        if max_flow == num_penguins {
            results.push(floe);
        }
    }

    if results.is_empty() {
        println!("-1");
    } else {
        print!("{}", results[0]);
        for &x in &results[1..] {
            print!(" {}", x);
        }
        println!();
    }
}
```