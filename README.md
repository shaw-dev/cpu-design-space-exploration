# Processor Design Space Exploration

Searching an 18-dimension out-of-order CPU design space for energy/delay/area-optimal configurations under a fixed simulation budget.

**Stack:** C++ · SimpleScalar (sim-outorder + power/area models) · Python (matplotlib) for analysis
**Context:** Computer architecture course project, Penn State (solo)

---

## The problem

A superscalar out-of-order core has many coupled knobs: fetch/decode width, RUU and LSQ sizes, L1/L2 cache geometry, branch predictor, functional units, and more. Here the space spans **18 dimensions and more than 10¹² configurations**, and each evaluation is a full cycle-accurate simulation across 5 benchmarks. The budget was **1,000 evaluations per objective**, so exhaustive search was out of the question.

Four objectives, each normalized to a baseline core and aggregated by geometric mean across benchmarks:

| Metric | Formula | What it rewards |
|---|---|---|
| EDP | Energy × Delay | Balanced efficiency |
| ED²P | Energy × Delay² | Speed (delay counts twice) |
| EDAP | Energy × Delay × Area | Compact, efficient designs |
| ED²AP | Energy × Delay² × Area | Fast *and* compact |

## Approach

1. **Constraint validation.** Many configs in the raw space aren't physically sensible, or they crash the simulator. A validator rejects them before they spend any budget. It checks:
   - IL1 block size = IFQ width
   - DL1/IL1 block sizes match, and L2 block ≥ 2× L1 block
   - L2 capacity ≥ DL1 + IL1, and L1 ≤ 64 KB
   - LSQ ≤ RUU ≤ 128 entries
2. **Search.** For each objective, sample 1,000 *unique, valid* configurations with a Mersenne Twister RNG, shuffle them, and evaluate them in sequence. This is a deliberately simple **constrained random-search baseline**: every evaluation is a legal design and nothing is evaluated twice.
3. **Analysis.** For each objective, plot the per-benchmark breakdown for the best design and the metric across the run, then compare how the winning designs differ by objective.

## Results

- **Best EDP design: ~0.42× baseline geomean (~58% reduction)**
- The winning design changes with the objective, which is the main takeaway:

| Objective | What the best design looked like |
|---|---|
| EDP | Narrow-to-moderate width, compact L1/L2, no perfect predictor |
| ED²P | Wider issue and more speculation, with moderately sized caches. It accepts higher energy for lower delay |
| EDAP | Avoids area-heavy features: low-associativity caches, smaller RUU/LSQ |
| ED²AP | Moderate on every axis, with mid-tier associativity and predictor |
<img width="531" height="570" alt="Screenshot 2026-11-19 at 3 16 13 PM" src="https://github.com/user-attachments/assets/733088d6-c61b-46cc-9150-e2f40c7522c1" />

<img width="651" height="658" alt="Screenshot 2026-11-19 at 3 17 08 PM" src="https://github.com/user-attachments/assets/f9ed8b22-34a3-40d3-a5bc-2496da823095" />
<img width="662" height="602" alt="Screenshot 2026-11-19 at 3 16 54 PM" src="https://github.com/user-attachments/assets/8e8398bc-289e-402f-aca8-870923d7fb3b" />
<img width="720" height="617" alt="Screenshot 2026-11-19 at 3 16 37 PM" src="https://github.com/user-attachments/assets/f76c557c-f3c3-4686-aa4c-d6de5153ab8a" />



There's no single "best CPU", only the best CPU for a given cost function.

## Limitations & what I'd do next

- **Random search ignores feedback.** It doesn't use the best design found so far. Next steps:
  - coordinate descent, sweeping one dimension at a time around the current best
  - then simulated annealing, or a surrogate model such as a GP or random forest over configs
- **Baseline comparison.** Plot best-so-far for guided search against this random baseline at the same 1,000-evaluation budget.
- **Tighter validation.** Encode more architectural constraints, such as decode/issue width consistency and TLB sizing, to shrink the space further.

## Code

The source is a graded course assignment, so it isn't public. I'm happy to share it or walk through it on request: shauryachauhan1111@gmail.com
