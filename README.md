# Lab 08: Navigating Weighted Graphs with Dijkstra's Algorithm

Name(s): Issei Hasegawa 

## Introduction
You are already familiar with Breadth-First Search (BFS) and Depth-First Search (DFS) for exploring graphs. However, what happens when all edges are not created equal? In this lab, we will explore Dijkstra's Algorithm, which allows us to find the shortest path between two vertices in a *weighted* graph.

---

## Part 1: BFS with Weights

BFS is excellent for finding the shortest path in an unweighted graph. Let's see why it fails when weights are introduced. Consider the following weighted graph:

![Part 1 Graph](./part1.png)

**Questions:**
1. If you run a standard BFS starting at Node $A$, looking for Node $D$, which path will the BFS algorithm discover first? Why?
   > BFS finds **A → D** first because D is only one edge away from A. BFS checks nodes one level at a time and does not look at edge weights. So even though the direct edge is much heavier, BFS finds it before the three-edge route A → B → C → D.

2. What is the total weight of the path BFS found? What is the total weight of the alternative path? 
   > The path BFS finds, **A → D**, has a total weight of **100**. The other route, **A → B → C → D**, has a total weight of **2 + 3 + 4 = 9**. It takes more edges, but its total weight is much lower.

3. How does the definition of "shortest path" change when moving from unweighted graphs to weighted graphs?
   > In an unweighted graph, "shortest" means using the fewest edges. In a weighted graph, it means having the lowest total edge weight, even if that takes more edges. Here, A → D uses one edge but costs 100, while A → B → C → D uses three edges and costs only 9. That makes A → B → C → D the shortest path when we account for weights.

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

In each row, I have checked the current node's unvisited neighbors and added the current node to `visited`. Before step 0, $A$ has distance 0, every other node has distance $\infty$, and `visited` is empty.

| Step | Current Node | Neighbors to Check | `dist` Map | `visited` Set | 
| :--- | :--- | :--- | :--- | :--- | 
| 0 | $A$ | $B$, $C$ | $A$:0, $B$:4, $C$:2, $D: \infty$, $E: \infty$ | {A} |
| 1 | $C$ | $B$, $D$, $E$ | $A$:0, $B$:3, $C$:2, $D$:10, $E$:12 | {A, C} |
| 2 | $B$ | $D$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:12 | {A, C, B} |
| 3 | $D$ | $E$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {A, C, B, D} |
| 4 | $E$ | None | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {A, C, B, D, E} |

**Conceptual Question:**
When we add a vertex to the "visited" set, why are we 100% certain we have found the absolute shortest path to that node? 
> This works because all edge weights are nonnegative and we always choose the unvisited node with the smallest distance so far. Consider any possible path to the node we chose. The first unvisited node on that path comes after a visited node whose distance is already final. When we processed that visited node, we checked the edge between them, so the first unvisited node's recorded distance is at most the cost of reaching it along this path. The node we chose has a distance no larger than that, and the remaining edges cannot lower the path's cost because none are negative. So no alternative path can beat the distance we already have. Since we have also found an actual path with that distance, we can safely mark it as final.

---

## Part 3: Assumptions

Dijkstra's algorithm relies on an assumption about the weights. Let's see what happens when we break it. Consider the following weighted graph, which contains a negative edge:

![Part 3 Graph](./part3.png)

**Instructions:**
Trace Dijkstra's algorithm starting from Node $A$ to find the shortest path to Node $C$.

1. Which node does Dijkstra's algorithm visit first after $A$?
   > It visits **C** next. After checking A's neighbors, the distances are **1 for C** and **2 for B**, so C is chosen first.

2. What will Dijkstra's algorithm declare as the shortest path to $C$, and what is its weight?
   > It reports **A → C**, with a total weight of **1**. C is chosen and its distance is marked as final before B gets processed.

3. Look at the graph manually. What is the *actual* shortest path from $A$ to $C$, and what is its total weight? 
   > If we do not repeat vertices, the actual shortest path is **A → B → C**. Its total weight is **2 + (−5) = −3**, which is lower than the direct route's weight of 1.

4. **The Trap:** Explain exactly *why* Dijkstra's algorithm failed to find the correct shortest path in this scenario.
   > Dijkstra's algorithm relies on edge weights being nonnegative when it marks the closest unvisited node's distance as final. Here, it marks C's distance as 1 before checking B, since B's distance is 2. But the −5 edge from B to C makes the route through B cost only −3. C's distance was marked as final too early. Even though B is farther from A at first, going through B gives a shorter route to C, so the usual reason for trusting a finalized distance no longer holds.

**Bonus: Why did the original weights fail to demonstrate the intended problem?**

> With the original weights, AB = 2, BC = −4, and AC = 5, the algorithm checks B before C because 2 is less than 5. While processing B, it changes C's distance to 2 + (−4) = −2. This happens before C's distance is marked as final, so it finds the correct shortest simple path, A → B → C. That is why the original example does not show the intended failure. A negative edge means Dijkstra's algorithm is no longer guaranteed to work, but it can still give the right answer in some cases.

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

   > I would leave stairways out of the graph, or remove their edges from the adjacency lists if the graph already exists. If a sidewalk has an edge in each direction, both need to be removed. That way, the algorithm has no way to choose a stairway. The intersections themselves and their other accessible paths can stay in the graph.

2. How can you account for the $3$-minute waiting time at busy crosswalk intersections using *only* edge weights? (Hint: Think about how you could modify the weights of edges connecting to that intersection).

   > I would give each accessible two-way sidewalk two directed edges, one in each direction. For u → v, I would add the waiting time at u to the 2-minute travel time. The edge would cost **5 minutes if u is a busy crosswalk, or 2 minutes otherwise**. This counts the extra 3 minutes when the robot leaves a busy intersection. The reverse edge uses the waiting time at v, so the two directions can have different weights. For example, if u is busy and v is not, u → v costs 5 and v → u costs 2. This also counts the wait if the robot starts at a busy intersection, but adds no wait just for arriving at its destination, since it does not leave that intersection. All the weights are still nonnegative, so Dijkstra's algorithm can use them.
