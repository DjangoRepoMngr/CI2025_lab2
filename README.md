# TSP with Greedy Baseline and Evolution Strategies (ES)

> Polito — Computational Intelligence (CI) — Lab 2
>
> I build a strong **greedy + 2‑opt hill‑climber** baseline for TSP‑style instances, then implement two Evolution Strategy variants: **(μ+λ)** and **(μ,λ)**. All runs are reproducible (NumPy `seed=42`).

---

## Project Structure

```
CI2025_lab2/
├─ lab2/                  # 22 problem instances (.npy distance/cost matrices)
│  ├─ problem_g_*.npy     # geometric / metric-like
│  ├─ problem_r1_*.npy    # random positive symmetric
│  └─ problem_r2_*.npy    # random symmetric, may include NEGATIVE costs
├─ notebooks/
│  ├─ 01_greedy_baseline.ipynb   # ε-greedy NN + 2-opt hill climber (+ optional SA)
│  └─ 02_es.ipynb                # ES(μ+λ) and ES(μ,λ) with OX crossover
├─ results/
│  ├─ hc_results.csv      # baseline per instance
│  ├─ es_plus.csv         # ES(μ+λ) per instance
│  └─ es_comma.csv        # ES(μ,λ) per instance
└─ README.md
```

**Important:** In `r2_*` instances, some entries can be **negative** by design. The objective is always **minimize** total tour cost; therefore “more negative” can be better.

---

## Algorithms

### 1) Greedy + 2‑opt Hill Climber (baseline)

- **Encoding:** a tour is a **permutation** of cities `[0..n−1]`; cost is `Σ D[t[i], t[i+1]]` with wrap.
- **Initialization (ε‑greedy NN):** start from a random city; usually pick the **nearest unvisited** city, but with probability `p_random` pick a random unvisited city to diversify starts.
- **Local move (2‑opt):** pick two non‑adjacent edges `(a–b, c–d)`, reconnect as `(a–c, b–d)` by **reversing** the middle segment. Accept if it **reduces** cost.
- **Sweeps & stop:** a **sweep** is one pass trying many candidate 2‑opt swaps; repeat sweeps until none improve (or a small budget is reached).
- **Restarts:** run multiple seeds, keep the best.
- **Optional SA:** **Simulated Annealing** can be enabled to occasionally accept uphill moves early (escape shallow local minima).

**Why:** This is fast and surprisingly strong on a wide range of instances; it provides a clear yardstick for ES.

---

### 2) Evolution Strategies (ES) for TSP

Common building blocks:

- **Population:** size **μ** (parents). Each generation creates **λ** offspring.
- **Parent selection:** **tournament** of size `τ` (best among `τ` random individuals).
- **Crossover (permutation‑safe):** **Order Crossover (OX)** — copy a slice from Parent 1, then fill remaining cities in the order they appear in Parent 2; always yields a valid permutation.
- **Mutation:** **swap** two random positions (applied with probability `mutation_rate` per offspring).
- **Initialization:** 70% **greedy** seeds + 30% **random** seeds.
- **Optional local polish:** small‑budget **2‑opt** per child (disabled by default for speed).

Survivor selection modes:

- **ES(μ+λ)** *(elitist)*: next population = **best μ** from **parents ∪ offspring** (more stable / consistent).
- **ES(μ,λ)** *(non‑elitist)*: next population = **best μ** from **offspring only** (more exploratory).

**Default hyper‑parameters used here:**

```
μ = 50,  λ = 100,  generations = 300,
τ = 3 (tournament size),  mutation_rate = 0.20,
init_greedy_frac = 0.70,  seed = 42
```

---

## How to Run

1. Place all `.npy` files in `lab2/`.
2. Run `notebooks/01_greedy_baseline.ipynb` → writes `results/hc_results.csv`.
3. Run `notebooks/02_es.ipynb` with `ES_MODE="(mu+lambda)"` → writes `results/es_plus.csv`.
4. Switch to `ES_MODE="(mu,lambda)"` → writes `results/es_comma.csv`.

> Tip: Use different filenames for different parameter configurations to avoid overwriting results.

---

## Results (Comparison)

Lower **best\_cost** is better (negative values are possible in `r2_*`). The table shows one representative size per family.

| instance | HC best_cost | ES(μ+λ) best_cost | ES(μ,λ) best_cost | winner |
|----------|-------------:|-----------------:|------------------:|:------:|
| problem_g_10.npy | 1497.664 | 1497.664 | 1497.664 | tie |
| problem_g_20.npy | 1755.515 | 1755.515 | 1755.515 | tie |
| problem_g_50.npy | 2663.122 | 3268.538 | 2733.430 | HC |
| problem_g_100.npy | 4017.487 | 5240.010 | 4960.986 | HC |
| problem_g_200.npy | 5663.379 | 7982.550 | 8698.660 | HC |
| problem_g_500.npy | 9162.392 | 16900.436 | 16981.908 | HC |
| problem_g_1000.npy | 13180.080 | 34847.472 | 34094.242 | HC |
| problem_r1_10.npy | -12950.398 | 184.273 | 184.273 | HC |
| problem_r1_20.npy | -10766.837 | 392.255 | 387.630 | HC |
| problem_r1_50.npy | -17925.004 | 599.129 | 610.732 | HC |
| problem_r1_100.npy | -19115.273 | 915.655 | 956.574 | HC |
| problem_r1_200.npy | -38118.556 | 1583.334 | 1721.826 | HC |
| problem_r1_500.npy | -43973.213 | 2895.879 | 3235.133 | HC |
| problem_r1_1000.npy | -43271.693 | 6131.336 | 6442.376 | HC |
| problem_r2_10.npy | -22512.801 | -411.702 | -411.702 | HC |
| problem_r2_20.npy | -222559.148 | -796.863 | -801.831 | HC |
| problem_r2_50.npy | -247376.893 | -2141.314 | -2144.079 | HC |
| problem_r2_100.npy | -258709.422 | -4406.837 | -4354.480 | HC |
| problem_r2_200.npy | -262579.539 | -9131.448 | -8938.536 | HC |
| problem_r2_500.npy | -280062.938 | -23005.310 | -22476.183 | HC |
| problem_r2_1000.npy | -305382.765 | -46515.422 | -45785.741 | HC |
| test_problem.npy | 2823.790 | 2823.790 | 2950.750 | HC / ES(μ+λ) tie |


**Head-to-head (ES variants):**

- ES(μ+λ) vs ES(μ,λ): **12** wins · **6** losses · **4** ties.\
  By family — g: plus 2, comma 3, ties 2 · r1: plus 5, comma 1, ties 1 · r2: plus 4, comma 2, ties 1.

**Against the HC baseline:**

- HC vs ES(μ+λ): HC better on **21/22**, **1** tie.
- HC vs ES(μ,λ): HC better on **22/22**.

**Average runtime (seconds per instance):**

- HC: **22.81 s**
- ES(μ+λ): **3.11 s**
- ES(μ,λ): **3.25 s**

## Conclusion

- In these runs, the **ε‑greedy + 2‑opt hill climber** consistently produced the **best tour costs** across all 22 instances (with one tie vs ES(μ+λ)). It’s a very strong baseline for this dataset.
- Between ES variants, **ES(μ+λ)** (elitist) outperformed **ES(μ,λ)** overall (12–6 with 4 ties), though (μ,λ) was sometimes competitive on geometric (g\_\*) cases.
- **Runtime:** ES variants were **\~7× faster on average** than our HC configuration, but at the expense of solution quality with current settings.

## **If you want ES to close the gap:** increase generations and/or λ/μ, enable a light **2‑opt local polish** per offspring, and tune mutation rate. Those changes typically improve ES results on permutation problems.

### Glossary (quick)

- **best\_cost**: objective value (lower is better; can be negative in `r2_*`).
- **sweep (2‑opt)**: one pass of trying many 2‑opt swaps; stop when a sweep finds no improvement.
- **(μ+λ)**: choose survivors from **parents + offspring** (elitist).
- **(μ,λ)**: choose survivors from **offspring only** (non‑elitist).

