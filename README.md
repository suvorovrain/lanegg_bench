# Results of LaneggRPQ benchmarks
This repository contains results of regular path querry solvers benchmarks.

## Solvers
In this work, we consider the following RPQ solvers.

1. **Lanegg_simp** --- [LaneggRPQ](https://github.com/SparseLinearAlgebra/la-n-egg-rpq) solver without any plan optimizations.
2. **Lanegg_card** --- [LaneggRPQ](https://github.com/SparseLinearAlgebra/la-n-egg-rpq) solver with optimized plan via cost function based on cardinality of matrices.
3. **RPQ-matrix** --- [RPQMatrix](https://github.com/adriangbrandon/rpq-matrix) solver presented in [following work](https://link.springer.com/article/10.1007/s00778-024-00885-6).

## Datasets
We choose following datasets for RPQ solvers benchmarking.
1. **Wikidata** --- This dataset was present in [MillenniumDB path query challenge](https://github.com/MillenniumDB/path-query-challenge). It contains:
    - 610 millions of edges
    - 91 millions of vertices
    - 1400 unique labels
    - 45 `any-to-any` queries
    - 94 `con-to-any` queries

2. **RPQBecnh** --- generated synthetical [database](https://www.researchgate.net/publication/386377372_RPQBench_A_Benchmark_for_Regular_Path_Queries_on_Graph_Data). It contains:
    - 150 millions of edges
    - 57 millions of vertices
    - 9 unique labels
    - 9 `any-to-any` queries
    - 7 `con-to-any` queries

## Machine
The benchmarks were conducted on a machine with the following specifications.
- CPU: AMD Ryzen 9 7900X, 12 physical, 24 logical cores;
- RAM: 128 Gb, 3600 MHz, DDR5;
- OS: Linux Ubuntu 22.04;

## Benchmarks
- ****Speedup*** is computed as the performance improvement of  **LaneggRPQ_card** relative to **RPQ-Matrix**.*
- *The standard deviation of measurements for all queries does not exceed 5 % of the mean.*
### any-to-any queries
In this section we consider **any-to-any RPQ queries**, where neither the start vertex nor the target vertex is fixed.  
Such queries compute all pairs of vertices in the graph that are connected by a path matching the given regular expression.

**1. RPQBench**

Table below demonstrates the result obtained from RPQBench dataset on each query. Summary statistic is available in the [rpqbench](rpqbench/any-any/) folder. All values are presented in milliseconds.
![Wikidata con-to-any result](rpqbench/any-any/anyany.png)

- `Lanegg_simp` failed with OOM on the 7th query presented in table

**2. Wikidata**

The table below shows summary statistics for all wikidata queries. Detailed results for each query are available in the [wikidata](wikidata/any-any/) folder.
![Wikidata any-to-any result](wikidata/any-any/anyany.png)


### con-to-any queries

In this section we consider **constrained-to-any RPQ queries**, where the **start vertex is fixed**, but the target vertex is not specified.  
Such queries compute all vertices reachable from the given start vertex by paths matching the regular expression.

**1. RPQBench**

Table below demonstrates the result obtained from RPQBench dataset on each query. Summary statistic is available in the [rpqbench](rpqbench/con-any/) folder. All values are presented in milliseconds.
![Wikidata con-to-any result](rpqbench/con-any/conany.png)
- Column `Solver` and `Planner` shows how much time was spent on plan execution and plan optimization in `Lanegg_card` case.

- `Lanegg_simp` failed with OOM on the last query presented in table

**2. Wikidata**

The table below shows summary statistics for all wikidata queries. Detailed results for each query are available in the [wikidata](wikidata/con-any/) folder.
![Wikidata con-to-any result](wikidata/con-any/con-any.png)

## Results

The following conclusions can be drawn from the experimental results.

1. On average, the optimized **LaneggRPQ** demonstrates a significant performance improvement over the considered baselines.  
   In some cases the speedup reaches up to **543,000×**, while the average speedup across different datasets and queries ranges from **7.8× to 13,403.2×**.

2. On highly sparse matrices, particularly for **con-to-any queries**, LaneggRPQ may be slower than **RPQ-Matrix**. This behavior can be explained by two factors:

   - **RPQ-Matrix** relies on a relatively lightweight linear algebra implementation. As a result, for simple queries the overhead of matrix operations is minimal. In contrast, **SuiteSparse:GraphBLAS**, used by LaneggRPQ, is a highly optimized and feature-rich production-grade library that supports a wide range of linear algebra operations and optimizations, which introduces additional constant overhead on very small computations.

   - The optimization strategy of **RPQ-Matrix** is based on dynamically computing matrix characteristics during execution. For extremely sparse matrices, such computations are very cheap and can lead to faster execution. However, for more complex queries involving denser intermediate matrices, RPQ-Matrix may suffer from severe slowdowns, while LaneggRPQ maintains significantly better performance.

3. In the vast majority of cases, the **query plan optimization time** is negligible compared to the **query execution time**. This indicates that additional improvements to the optimizer (e.g., more sophisticated cost models or heuristics) can be introduced without significantly affecting the overall runtime.

4. As can be observed from the tables and [detailed](lanegg_bench/wikidata/con-any/benchmark_summary_compact_ms.txt) Wikidata results, the **Kleene star operation** often leads to slower execution of LaneggRPQ. This behavior suggests possible optimizations for the computation of transitive closure for specific cases.

## Reproducibility

The solvers used in the experiments, along with instructions for running them, can be found in the following GitHub repositories:

- [LaneggRPQ](https://github.com/SparseLinearAlgebra/la-n-egg-rpq)
- [RPQ-Matrix](https://github.com/adriangbrandon/rpq-matrix)

If you encounter memory usage errors when running **RPQ-Matrix**, consider applying [this fix](https://github.com/adriangbrandon/rpq-matrix/pull/1).

Note that for some queries the original implementation of **RPQ-Matrix** may produce incorrect results due to an issue in the matrix construction function.  
A corrected version based on **SuiteSparse:GraphBLAS**, which produces consistent results, is available [here](https://github.com/suvorovrain/rpq-matrix/tree/gbmod).

All datasets used in the experiments, as well as the tools required for preparing them, are available in the [la-rpq](https://github.com/SparseLinearAlgebra/la-rpq) repository (thanks to [George Belyanin](https://github.com/georgiy-belyanin)).

The queries used in the benchmarks are provided in the corresponding dataset folders in this repository.