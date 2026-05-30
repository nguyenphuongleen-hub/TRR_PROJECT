# TSP Brazil58 — Algorithm Comparison

A complete implementation and analysis of four combinatorial optimization algorithms applied to the **Brazil58** Traveling Salesman Problem dataset (58 cities).

---

## Table of Contents

1. [Problem Overview](#problem-overview)
2. [Dataset](#dataset)
3. [Project Structure](#project-structure)
4. [Algorithms Implemented](#algorithms-implemented)
5. [Implementation Details](#implementation-details)
6. [Requirements](#requirements)
7. [Usage](#usage)
8. [Results Summary](#results-summary)
9. [Outputs](#outputs)
10. [Key Takeaways](#key-takeaways)

---

## Problem Overview

The **Traveling Salesman Problem (TSP)** asks: given a set of cities and pairwise distances, find the shortest closed tour that visits every city exactly once.

This project compares two strategies for tackling TSP:
- **Greedy construction heuristics** — fast algorithms that build a valid tour from scratch.
- **Local search improvements** — iterative refinement methods that improve an existing tour until no better move exists.

Each greedy algorithm is paired with a local search that uses it as the initial solution, forming two *algorithm pipelines*:

| Pipeline | Construction | Improvement |
|----------|-------------|-------------|
| **Pair A** | Cheapest Link | Relocate |
| **Pair B** | Nearest Neighbor | 2-Opt |

---

## Dataset

**Name:** `brazil58`  
**Source:** Ferreira — 58 Brazilian cities  
**Format:** `.tsp` file with `UPPER_ROW` symmetric edge weight matrix  
**File:** `data/brazil58.tsp`

The distance matrix is stored in upper-triangular row format. Row `i` (0-indexed) contains distances from city `i` to cities `i+1, i+2, ..., n-1`. The parser reconstructs the full symmetric `n × n` distance matrix.

---

## Project Structure

```
tsp-brazil58/
│
├── data/
│   └── brazil58.tsp              # TSP input file (UPPER_ROW format)
│
├── tsp_brazil58_complete.ipynb   # Main notebook — all algorithms + analysis
│
├── tours_all4.png                # Output: MDS layout visualization of 4 tours
├── comparison_charts.png         # Output: Bar charts comparing distance, runtime, improvement
│
└── README.md                     # This file
```

### File Descriptions

| File | Description |
|------|-------------|
| `data/brazil58.tsp` | Input dataset. Contains the city count (`DIMENSION: 58`) and the upper-row symmetric distance matrix under `EDGE_WEIGHT_SECTION`. |
| `tsp_brazil58_complete.ipynb` | Self-contained Jupyter notebook. Loads the data, runs all four algorithms, validates tours, prints comparison tables, and generates visualizations. |
| `tours_all4.png` | 2×2 grid of MDS-projected tour plots, one per algorithm. Cities are positioned using Multidimensional Scaling (MDS) applied to the distance matrix so spatial relationships reflect true distances. |
| `comparison_charts.png` | Three-panel bar chart: (1) total tour distance per algorithm, (2) runtime in milliseconds, (3) percentage improvement each local search achieves over its greedy seed. |

---

## Algorithms Implemented

### 1. Cheapest Link *(Greedy)*

**Strategy:** Sort all `n(n-1)/2` edges by weight. Greedily add each edge to the tour, skipping it if it would give any city a degree greater than 2 or create a premature sub-cycle (checked via **Union-Find**). The final edge closes the Hamiltonian circuit.

**Complexity:** O(n² log n) — dominated by sorting all edges.

**Sub-tour prevention:** Path compression Union-Find with union-by-rank. An edge is only allowed to form a cycle when it is the very last edge being added.

---

### 2. Relocate *(Local Search — initialized by Cheapest Link)*

**Strategy:** For each city in the current tour, compute the *net gain* from removing it and re-inserting it at every other position. Apply the single best improving move, then restart the full scan. Terminate when a complete pass yields no improvement — a **local optimum**.

**Complexity:** O(n²) per iteration.

**Move mechanics:**
- Removal gain = cost of two edges incident to the city minus the shortcut edge that replaces them.
- Insertion cost = cost of two new edges minus the edge being split.
- Net delta = insertion cost − removal gain. A negative delta means the move reduces total distance.

---

### 3. Nearest Neighbor *(Greedy)*

**Strategy:** Starting from city 0, always extend the current open path to the **nearest unvisited city**. After all cities are visited, close the circuit back to the starting city.

**Complexity:** O(n²).

**Properties:** Impossible to form sub-tours by construction, since the tour is built as a single path. Very fast in practice, though the closing edge can be long.

---

### 4. 2-Opt *(Local Search — initialized by Nearest Neighbor)*

**Strategy:** For every pair of non-adjacent edges `(a→b)` and `(c→d)`, test whether swapping them to `(a→c)` and `(b→d)` — equivalently, reversing the sub-segment between `b` and `c` — reduces total tour length. Apply the first improving swap found and restart. Terminate when no improving swap exists.

**Complexity:** O(n²) per iteration.

**Skip condition:** The wrap-around pair `(i=0, j=n-1)` is skipped as it produces the same tour.

---

## Implementation Details

### Notebook Sections

| Section | Content |
|---------|---------|
| 1. Imports & Setup | Standard scientific stack: `numpy`, `matplotlib`, `sklearn.manifold.MDS` |
| 2. Data Loading | `parse_upper_row_matrix()` — reads `.tsp` file, reconstructs symmetric distance matrix |
| 3. Utility Functions | `tour_length()`, `is_valid_tour()`, `validate_tour()`, `print_result()` |
| 4. Cheapest Link | Full implementation with Union-Find for cycle detection |
| 5. Relocate | Local search with best-improvement strategy per pass |
| 6. Nearest Neighbor | Simple greedy path extension |
| 7. 2-Opt | Double-loop edge-swap local search |
| 8. Validation | Integrity check — length, duplicates, missing cities for all four tours |
| 9. Within-Pair Comparison | Pair A (CL → Relocate) and Pair B (NN → 2-Opt) side-by-side |
| 10. Cross-Algorithm Comparison | Ranked table of all four algorithms by distance and runtime |
| 11. Visualization | MDS tour plots + bar chart comparisons |
| 12. Final Summary | Formatted results table with best-solution indicator |
| 13. Conclusions | Algorithm property comparison tables and key takeaways |

### Tour Validation

Every computed tour is verified for:
- Correct length (exactly `N` cities)
- No duplicate cities
- No missing cities

Validation runs after all algorithms complete and any failure is reported with the specific offending cities.

### MDS Visualization

Since the dataset provides only pairwise distances (no geographic coordinates), city positions are approximated using **Multidimensional Scaling (MDS)** with `dissimilarity='precomputed'`. The MDS layout is computed once and shared across all four tour plots.

---

## Requirements

```
python >= 3.8
numpy
matplotlib
scikit-learn
```

Install dependencies:

```bash
pip install numpy matplotlib scikit-learn
```

The notebook uses only the Python standard library beyond these packages (`time`, `math`).

---

## Usage

1. **Clone or download** the project, preserving the `data/` folder structure.

2. **Verify the data file** is present:
   ```
   data/brazil58.tsp
   ```

3. **Open the notebook:**
   ```bash
   jupyter notebook tsp_brazil58_complete.ipynb
   ```

4. **Run all cells** (`Kernel → Restart & Run All`).

The notebook will:
- Load and parse the distance matrix
- Run all four algorithms and print results after each
- Validate all tours
- Print within-pair and cross-algorithm comparison tables
- Generate and save `tours_all4.png` and `comparison_charts.png`
- Print a final formatted summary

---

## Results Summary

> Results are deterministic — all algorithms are non-random.

| Algorithm | Type | Tour Distance | Runtime |
|-----------|------|:-------------:|:-------:|
| Cheapest Link | Greedy | ~25,395 | fast |
| **Relocate** (init: CL) | Local Search | improved | moderate |
| Nearest Neighbor | Greedy | higher | very fast |
| **2-Opt** (init: NN) | Local Search | **best** | moderate |

- Both local searches reduce their greedy seed by approximately **12–16%**.
- **2-Opt achieves the overall best tour distance** across all four algorithms.
- **Nearest Neighbor** is the fastest construction heuristic but yields a weaker starting solution than Cheapest Link.
- The choice of local search method has more impact on final quality than the choice of greedy initializer.

---

## Outputs

After a full notebook run, two image files are saved to the project root:

### `tours_all4.png`
A 2×2 grid showing each algorithm's tour on the MDS-projected city layout. Each panel shows the total distance, runtime, and labels a selection of city indices. The starting city (city 1) is marked with a red star.

Color coding:
- 🟠 Orange — Cheapest Link
- 🔵 Blue — Relocate
- 🟣 Purple — Nearest Neighbor
- 🟢 Green — 2-Opt

### `comparison_charts.png`
Three side-by-side bar charts:
1. **Tour Distance** — absolute total distance per algorithm, with a dashed reference line at the best value.
2. **Runtime (ms)** — wall-clock time for each algorithm.
3. **Local Search Improvement (%)** — percentage reduction each local search achieves over its greedy seed (greedy baselines shown at 0%).

---

## Key Takeaways

- **2-Opt + Nearest Neighbor** is the best pipeline: delivers the shortest tour with competitive runtime.
- **Cheapest Link** produces a marginally better greedy baseline at the cost of higher complexity (O(n² log n) vs O(n²) for Nearest Neighbor).
- **Local search dominates greedy alone**: a 12–16% improvement is consistently achieved regardless of which greedy algorithm is used to seed it.
- For small n (≤ 100), both local searches converge quickly; on larger instances, 2-Opt's segment-reversal moves tend to make bigger jumps per iteration than Relocate's single-city moves.
