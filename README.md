# NSGA-II for Desk Assignment and Group Meeting-Day Scheduling

A population-based multi-objective metaheuristic (NSGA-II) that decides **which day each group meets** and, from that decision, **which desk each employee uses each day**. The project was developed for the *Heuristics* course at Universidad EAFIT and compares NSGA-II against five other solution methods.

## Authors

- Johan Alexander Álvarez Osorio – jaalvarez5@eafit.edu.co
- Santiago Vera Ramírez – sverar1@eafit.edu.co

Universidad EAFIT

---

## Problem Overview

In a hybrid-office setting, employees belong to groups, and each group must be assigned a meeting day within a planning horizon of `T` days (e.g., the working days of a week). Employees must then be assigned to desks (`J` desks) on each day. The solution is evaluated on four **simultaneous, independent objectives**:

| Objective | Description |
|-----------|-------------|
| `f1` | Number of isolated employees per zone |
| `f2` | Violations of desk preferences (invalid desk assignments) |
| `f3` | Violations of day preferences |
| `f4` | Desk reuse / number of desk changes |

All objectives are minimized and are treated as separate objectives (no weighted sum inside NSGA-II). The algorithm returns an approximation of the Pareto front.

---

## Solution Representation

A full solution is a pair `(x, y)`:

- **`y = (d_0, ..., d_{G-1})`**: the meeting day of each group, with `d_g ∈ {0, ..., T-1}`. This is the **only genotype** evolved by NSGA-II.
- **`x = (x_tij)`**: binary matrix of size `T × I × J`, where `x_tij = 1` if employee `i` uses desk `j` on day `t`. It is **not encoded in the chromosome**. It is rebuilt deterministically from `y` by a constructive procedure (`group_day`), with repair when needed.

This design shrinks the search space and keeps crossover and mutation simple, while still producing feasible, problem-consistent solutions.

Example: for `G = 4` groups and `T = 5` days, `y = [2, 0, 4, 1]` means G0 → Wednesday, G1 → Monday, G2 → Friday, G3 → Tuesday.

---

## Algorithm Components

- **Initialization:** constructive and controlled-random generation of `y`, followed by `group_day` and a feasibility check/repair, so the initial population is not highly infeasible.
- **Evaluation:** `FO(x, y) = (f1, f2, f3, f4)`.
- **Selection:** binary tournament based on (Pareto rank, crowding distance).
- **Crossover:** one- or multi-point crossover on the day vector `y` (probability `p_c`).
- **Mutation:** simple mutation that reassigns the meeting day of one or more groups (probability `p_m`).
- **Replacement:** fast non-dominated sorting on parents ∪ offspring, filling the next population front by front and using crowding distance to break ties in the last front.
- **Output:** the first Pareto front `F1` of the final population, with full `(x, y)` solutions reconstructed.


## Experimental Setup

### Parameter tuning (F-race)

Search space: `(P, Gmax, p_c, p_m) ∈ {80,120,160,200} × {20,60,100,140} × (0,1)²`.
Eight random configurations were raced over 10 instances, with progressive elimination based on the Friedman test and pairwise comparisons. The Pareto front was reduced to a single solution using a lexicographic order (isolated employees → invalid desk assignments → non-preferred days → desk changes) before applying the weighted scalarization.

**Selected parameters:**

| Parameter | Value |
|-----------|-------|
| Population size `P` | 160 |
| Generations `Gmax` | 60 |
| Crossover probability `p_c` | 0.4 |
| Mutation probability `p_m` | 0.6 |

### Compared methods

1. Deterministic constructive heuristic
2. Randomized constructive heuristic
3. Simulated Annealing
4. Local Search (Best Improvement)
5. GRASP
6. NSGA-II

### Statistical analysis

Objectives are min-max scaled and combined into a weighted sum for the non-parametric tests (weights: 0.5 isolated employees, 0.3 invalid desk assignments, 0.2 non-preferred days, 0.1 desk changes). Methods are compared with the **Friedman test** (5% significance) and **pairwise post-hoc comparisons**, following [Villegas](https://juangvillegas.com/wp-content/uploads/2011/08/friedman-test-24062011.pdf).

---

## Key Results

- **Weighted-sum criterion:** Local Search (Best Improvement) is the most effective method, followed by GRASP and NSGA-II, with no statistically significant difference between GRASP and NSGA-II.
- **Isolated employees only (primary objective):** NSGA-II is the non-dominated method, followed by GRASP and then Local Search.
- Simulated Annealing is not significantly different from the deterministic constructive method.
- **Runtime:** GRASP and NSGA-II are the most expensive (~30 s on average), Local Search is much cheaper (~3–4 s), and the constructive methods and SA are almost instantaneous.
- **Takeaway:** the ranking depends on the evaluation criterion. Local Search gives the best quality/time trade-off overall, while NSGA-II or GRASP are justified when minimizing isolated employees is strictly critical.

### Limitations noted by the authors

- NSGA-II only evolves part of the problem (the day vector) and delegates desk assignment to a constructive method.
- The Pareto front was not exploited during tuning, which limited a more robust parameterization.

---

## Reference

J. G. Villegas. *Using nonparametric test to compare the performance of metaheuristics.*
https://juangvillegas.com/wp-content/uploads/2011/08/friedman-test-24062011.pdf

## License

Specify a license (e.g., MIT) before publishing.
