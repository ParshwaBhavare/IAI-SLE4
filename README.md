# SLE-4: BFS vs DFS — Architecture Decision Record (ADR)

## 1. Project Overview

This project documents the architectural decision for comparing **Breadth-First Search (BFS)** and **Depth-First Search (DFS)** on different graph structures.

The main objective of SLE-4 is to explain how the graph data structure, traversal algorithms, visited array, and profiler are organized to ensure a fair and consistent performance comparison.

The project builds on the previous stages:

- **SLE-1:** Implementation of BFS and DFS.
- **SLE-2:** Profiling and performance comparison on three graphs.
- **SLE-3:** C4 architecture model and identification of software components.
- **SLE-4:** Documentation of the architecture decision, alternatives, rationale, and consequences.

## 2. Objectives

- Compare BFS and DFS using the same graph inputs.
- Use an adjacency-list representation to store graph data.
- Prevent repeated processing of vertices using a visited array.
- Measure execution time using a separate profiler/timer module.
- Understand the time complexity of both traversal algorithms.
- Document the reasons for selecting one architecture over alternative designs.

## 3. Technologies Used

- **Programming language:** Python
- **Graph representation:** Adjacency list
- **Traversal algorithms:** BFS and DFS
- **Timing module:** Python `time.perf_counter()`
- **Architecture documentation:** C4 model and Architecture Decision Record (ADR)
- **Profiling concept:** Repeated timing measurements; py-spy may be used separately for sampling-based profiling.

## 4. Architecture Design

The project uses a shared Graph Store and a separate Profiler/Timer module.

### Architecture Flow

```text
             Input
               |
               v
          Graph Store
       (Adjacency List)
               |
               v
       Traversal Engine
          /       \
         BFS      DFS
          \       /
           Visited
             Set
               |
               v
        Output Module

      Profiler / Timer
      (Separate Module)
```

### Main Components

| Component | Responsibility |
|---|---|
| Input Module | Provides the graph and starting vertex. |
| Graph Store | Stores graph vertices and their neighbours using an adjacency list. |
| Traversal Engine | Executes BFS or DFS. |
| Visited Set | Ensures each vertex is processed at most once. |
| Profiler/Timer | Measures execution time independently of traversal logic. |
| Output Module | Displays traversal results and timing measurements. |

Both algorithms use the same graph representation and starting vertex, making the comparison more consistent.

## 5. BFS and DFS

### Breadth-First Search (BFS)

BFS visits vertices level by level, exploring nearby vertices before moving farther away. It uses a **queue (FIFO — First In, First Out)**.

General steps:

1. Mark the starting vertex as visited.
2. Insert it into the queue.
3. Remove a vertex from the front of the queue.
4. Visit its unvisited neighbours and add them to the queue.
5. Continue until the queue is empty.

### Depth-First Search (DFS)

DFS explores one path as deeply as possible before returning to explore other paths. It can use recursion or an explicit stack.

General steps:

1. Mark the starting vertex as visited.
2. Visit an unvisited neighbour.
3. Continue exploring deeper vertices.
4. Backtrack when no unvisited neighbours remain.
5. Continue until all reachable vertices have been processed.

## 6. Test Graphs

Three deterministic graphs are used for the performance comparison.

| Graph | Vertices (V) | Edges (E) | Description |
|---|---:|---:|---|
| Tree | 10 | 9 | Sparse graph |
| Complete graph K10 | 10 | 45 | Every pair of distinct vertices is connected |
| Complete graph K20 | 20 | 190 | Larger complete graph |

The starting vertex for traversal is **vertex 0**.

A visited array or set is necessary for graphs containing cycles because it prevents vertices from being processed repeatedly.

## 7. Profiling Methodology

The profiler uses Python's `time.perf_counter()` to measure elapsed time.

For each graph and algorithm:

1. Use the same graph input and starting vertex.
2. Perform 10,000 traversals per timing run.
3. Repeat the timing measurement three times.
4. Calculate the average of the three runs.
5. Compare the measured times for BFS and DFS.

Repeated traversals make the measurement more reliable than timing a single, very short traversal. However, execution times can still vary with the computer, Python environment, and system activity.

### What Is py-spy?

`py-spy` is a sampling profiler for Python programs. It periodically samples the program's call stacks while the program runs, helping identify which functions consume execution time.

A flame graph visualizes sampled call stacks. Wider sections generally indicate that more samples included that function or call path. Flame graph colours, including red, orange, and yellow, do not by themselves indicate whether a function is slow or problematic.

**Note:** The timing results in this report use `time.perf_counter()`. Sampling-based profiling with py-spy is a separate technique and should only be described as an actual measurement if it was performed.

## 8. Performance Results

The following measurements are reported in the SLE-4 report.

| Graph | BFS Time (ms) | DFS Time (ms) | Faster Algorithm |
|---|---:|---:|---|
| Tree | 11.888 | 15.827 | BFS |
| K10 | 24.991 | 26.793 | BFS |
| K20 | 73.183 | 67.249 | DFS |

These measurements are the reported results of the repeated timing experiment, not guaranteed execution times on every computer.

### Observations

- BFS was faster on the tree graph.
- BFS was also faster on K10, although the difference was smaller.
- DFS was faster on K20.
- The winner changes depending on the tested graph and runtime conditions.
- Neither algorithm is consistently faster in every tested case.

The results demonstrate why both algorithms should be evaluated on identical inputs rather than declaring one universally faster.

## 9. Time and Space Complexity

With an adjacency-list representation, both BFS and DFS have a traversal time complexity of:

**Time complexity: O(V + E)**

Where:

- `V` is the number of vertices.
- `E` is the number of edges.

Each reachable vertex is processed at most once, and the algorithm examines its adjacency-list entries.

| Algorithm | Time Complexity | Auxiliary Space |
|---|---|---|
| BFS | O(V + E) | O(V) |
| DFS | O(V + E) | O(V) |

The space complexity includes the queue for BFS or the stack/recursion used by DFS and the visited structure.

## 10. Architecture Decision

### Decision: Use One Shared Adjacency-List Graph Store

The selected architecture stores each test graph once and allows both BFS and DFS to read the same graph. A visited array/set prevents repeated processing, while a separate Profiler/Timer measures performance.

### Why Was the Adjacency Matrix Rejected?

An adjacency matrix stores graph connections in a two-dimensional `V × V` array. It requires **O(V²)** space and typically makes traversal scan every possible neighbour position.

An adjacency list stores the actual neighbours of each vertex and requires **O(V + E)** space.

For the 10-vertex tree:

- Adjacency-list entries for an undirected graph: `V + 2E = 10 + 18 = 28`.
- Adjacency-matrix cells: `V² = 10 × 10 = 100`.

The adjacency list is therefore more suitable for the sparse tree because it avoids scanning positions for nonexistent edges during traversal.

For K20, the adjacency-list entry count is 400 and the matrix has 400 cells. The list is not guaranteed to use fewer entries for every graph; its main advantage is that it scales with vertices and edges rather than always allocating a full square matrix.

### Other Alternatives Rejected

1. **Separate graph structures for BFS and DFS:** Rejected because differences in graph construction could make the comparison unfair.
2. **Timing code inside each traversal function:** Rejected because it mixes performance measurement with algorithm logic and makes the design less modular.
3. **Only one traversal per timing measurement:** Rejected because a single traversal may finish too quickly for stable timing.

## 11. Consequences of the Decision

### Advantages

- Both algorithms use the same input graph.
- The timing method is separate from traversal logic.
- The architecture is modular and easier to maintain.
- The adjacency list supports efficient traversal in O(V + E) time.
- Repeated measurements reduce the influence of timing noise.

### Limitations

- Timing results depend on the machine and runtime environment.
- Repeated traversals require additional measurement time.
- Recursive DFS may encounter Python's recursion limit on very deep graphs.
- Only three graph configurations were tested, so broader performance conclusions require additional experiments.

## 12. Conclusion

SLE-4 documents the decision to use a shared adjacency-list Graph Store, a visited structure, and a separate Profiler/Timer for comparing BFS and DFS.

The design provides a consistent basis for comparing the two algorithms while keeping graph storage, traversal logic, and performance measurement separate.

The results show that the faster algorithm depends on the tested graph and runtime conditions. Therefore, the report combines measured timings with complexity analysis rather than claiming that BFS or DFS is always faster.

## 13. AI Contribution

AI assistance was used to help draft and organize the Architecture Decision Record and explain the design alternatives. The project decisions, reported measurements, module names, and final documentation were reviewed against the earlier SLE work.

## 14. Repository Contents

The repository may contain the following files, depending on what you upload:

```text
SLE-4/
├── README.md
├── report.pdf
├── bfs.py
├── dfs.py
├── graph_store.py
├── profiler.py
└── architecture/
    └── c4-diagram.png
```

Only include filenames that actually exist in your repository. The structure above is a suggested organization, not a claim that every file is already present.

## 15. References

- Python documentation: [`time.perf_counter()`](https://docs.python.org/3/library/time.html#time.perf_counter)
- py-spy documentation: [https://github.com/benfred/py-spy](https://github.com/benfred/py-spy)
