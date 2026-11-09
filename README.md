# Lab 08: Navigating Weighted Graphs with Dijkstra's Algorithm

Name(s): _______________________  

## Introduction
You are already familiar with Breadth-First Search (BFS) and Depth-First Search (DFS) for exploring graphs. However, what happens when all edges are not created equal? In this lab, we will explore Dijkstra's Algorithm, which allows us to find the shortest path between two vertices in a *weighted* graph.

---

## Part 1: BFS with Weights

BFS is excellent for finding the shortest path in an unweighted graph. Let's see why it fails when weights are introduced. Consider the following weighted graph:

![Part 1 Graph](./part1.png)

**Questions:**
1. If you run a standard BFS starting at Node $A$, looking for Node $D$, which path will the BFS algorithm discover first? Why?
   > Standard BFS will discover **A → D** first. BFS explores vertices level by level according to the number of edges from the starting vertex, without considering edge weights. Since D is directly adjacent to A, it is discovered at depth 1, before the three-edge path A → B → C → D, even though the direct edge has a much larger weight.

2. What is the total weight of the path BFS found? What is the total weight of the alternative path? 
   > The total weight of the path found by BFS, **A → D**, is **100**. The alternative path, **A → B → C → D**, has a total weight of **2 + 3 + 4 = 9**. Although the alternative path uses more edges, its total weight is smaller.

3. How does the definition of "shortest path" change when moving from unweighted graphs to weighted graphs?
   > In an unweighted graph, the shortest path is the path with the fewest edges. In a weighted graph, the shortest path is the path with the smallest sum of edge weights, which may use more edges. For example, A → D uses only one edge but has a weight of 100, while A → B → C → D uses three edges but has a total weight of 9, making it the shortest path in the weighted graph.

---

## Part 2: Dijkstra's Algorithm

Dijkstra's algorithm uses a greedy approach. It maintains a set of "visited" (finalized) nodes and an array of the shortest known distances to all other nodes. Consider the following weighted graph:

![Part 2 Graph](./part2.png)

**Instructions:**
Starting at Node $A$, trace Dijkstra's algorithm. Fill in the tracker table below. 
*   **Current Node:** The unvisited node with the smallest current distance.
*   **Neighbors to Check:** The neighbors of the current node whose distances might need updating.
*   **`dist` Array:** The current known shortest distance from $A$ to each node. Use $\infty$ for unreachable/unknown nodes.
*   **`visited` Set:** The set of nodes that have been finalized and whose shortest distances are known.

Each row shows the state after checking the current node's unvisited neighbors and adding the current node to `visited`. Initially, $A$ has distance 0, all other nodes have distance $\infty$, and `visited` is empty.

| Step | Current Node | Neighbors to Check | `dist` Map | `visited` Set | 
| :--- | :--- | :--- | :--- | :--- | 
| 0 | $A$ | $B$, $C$ | $A$:0, $B$:4, $C$:2, $D: \infty$, $E: \infty$ | {A} |
| 1 | $C$ | $B$, $D$, $E$ | $A$:0, $B$:3, $C$:2, $D$:10, $E$:12 | {A, C} |
| 2 | $B$ | $D$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:12 | {A, C, B} |
| 3 | $D$ | $E$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {A, C, B, D} |
| 4 | $E$ | None | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {A, C, B, D, E} |

**Conceptual Question:**
When we add a vertex to the "visited" set, why are we 100% certain we have found the absolute shortest path to that node? 
> Dijkstra's algorithm assumes that all edge weights are nonnegative. At each step, it selects the unvisited vertex with the smallest tentative distance. To see why this distance is final, consider any path to that vertex and the first unvisited vertex on that path. The preceding vertex has already been finalized, so its outgoing edge has been relaxed, giving the first unvisited vertex a tentative distance no greater than the cost of the path prefix reaching it. The selected vertex's tentative distance is no greater than that tentative distance, and the remaining edges cannot reduce the path's cost because their weights are nonnegative. Therefore, no path can be shorter than the selected vertex's current distance. Since that distance is also the cost of an actual discovered path, it is the true shortest distance and can safely be finalized.

---

## Part 3: Assumptions

Dijkstra's algorithm relies on an assumption about the weights. Let's see what happens when we break it. Consider the following weighted graph, which contains a negative edge:

![Part 3 Graph](./part3.png)

**Instructions:**
Trace Dijkstra's algorithm starting from Node $A$ to find the shortest path to Node $C$.

1. Which node does Dijkstra's algorithm visit first after $A$?
   > *Your answer here*

2. What will Dijkstra's algorithm declare as the shortest path to $C$, and what is its weight?
   > *Your answer here*

3. Look at the graph manually. What is the *actual* shortest path from $A$ to $C$, and what is its total weight? 
   > *Your answer here*

4. **The Trap:** Explain exactly *why* Dijkstra's algorithm failed to find the correct shortest path in this scenario.
   > *Your answer here*

---

## Part 4: Implementing Dijkstra's Algorithm

In the real world, problems rarely look like neat mathematical graphs right away. You often have to translate real-world constraints into a standard graph format so that an algorithm like Dijkstra's can process it.

**The Scenario:**
You are building a navigation app for delivery robots on a college campus. 

* Moving between any two adjacent sidewalk intersections takes exactly $2$ minutes.

* Some paths have stairs. The robots *cannot* use stairs.

* Certain intersections are busy crosswalks. The robot must wait an additional $3$ minutes at these specific intersections before it can safely leave them and continue on its path.

**The Problem:**
Standard Dijkstra's algorithm only understands *edges* and *edge weights*. It does not natively understand "node weights" (like waiting at an intersection) or "impassable terrain".

**Questions:**

1. How should you represent the paths with stairs in your graph so the algorithm never tries to use them?

   > Omit paths with stairs from the graph, or remove their edges from the adjacency lists. If a sidewalk is represented by two directed edges, remove both directions. With no edge representing a stairway, Dijkstra's algorithm cannot include it in a route. Keep the intersection vertices and any other accessible paths connected to them.

2. How can you account for the $3$-minute waiting time at busy crosswalk intersections using *only* edge weights? (Hint: Think about how you could modify the weights of edges connecting to that intersection).

   > Represent each accessible two-way sidewalk with two directed edges. For an edge u → v, set its weight to 2 minutes plus the waiting time at u: **5 minutes if u is a busy crosswalk, and 2 minutes otherwise**. This charges the extra 3 minutes whenever the robot leaves a busy intersection. The reverse edge v → u uses the waiting time at v, so the two directions may have different weights. For example, if u is busy and v is not, u → v has weight 5 and v → u has weight 2. This also charges a wait when departing a busy starting intersection, but does not charge a wait merely for arriving at the destination, because the robot does not leave it. All edge weights remain nonnegative, so standard Dijkstra's algorithm applies.
