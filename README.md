# Optimal Survey Sampling and Fieldwork Allocation

### Neyman Allocation with Minimum-Cost Network Flow Optimization

An optimization project that combines **stratified sampling**, **Neyman allocation**, and **minimum-cost network flow** to determine an efficient fieldwork plan for survey data collection.

The project studies the trade-off between **statistical precision** and **fieldwork cost** when assigning survey interviews to field teams with limited capacities.

---

## 1. Project Overview

In survey sampling, a population can be divided into different groups or **strata**. Neyman allocation determines how many observations should be sampled from each stratum to minimize the variance of the stratified estimator for a fixed total sample size.

However, the statistically optimal allocation does not directly determine how the interviews should be collected in practice.

Field teams may have:

* Limited interview capacity
* Different costs for different strata
* Different travel or fieldwork difficulties
* Different operational constraints

This project therefore combines the statistical allocation obtained from **Neyman allocation** with an **optimization model** that assigns interviews to field teams at minimum cost.

The overall framework is:

**Survey Data → Neyman Allocation → Fieldwork Optimization → Sensitivity Analysis → Cost–Precision Trade-off**

---

## 2. Objectives

The main objectives of the project are:

1. Calculate the statistically optimal sample allocation using Neyman allocation.
2. Formulate fieldwork assignment as a minimum-cost flow / transportation problem.
3. Assign interviews to field teams while respecting team capacity constraints.
4. Minimize total fieldwork cost.
5. Compare exact Neyman allocation with flexible allocations.
6. Study the effect of team capacity on the optimal cost.
7. Analyze the trade-off between fieldwork cost and statistical precision.
8. Implement the optimization model using both **PuLP** and **Pyomo**.

---

## 3. Statistical Foundation

### Stratified Sampling

Suppose the population is divided into (L) strata.

Let:

* (N_h) = population size of stratum (h)
* (S_h) = within-stratum standard deviation
* (n_h) = sample size allocated to stratum (h)
* (n) = total sample size

with

[
N = \sum_{h=1}^{L} N_h
]

and

[
\sum_{h=1}^{L} n_h = n.
]

For the stratified mean estimator, the approximate variance is

[
\operatorname{Var}(\bar{Y}*{st})
\approx
\sum*{h=1}^{L}
\left(\frac{N_h}{N}\right)^2
\frac{S_h^2}{n_h}.
]

### Neyman Allocation

Neyman allocation minimizes this variance subject to a fixed total sample size.

The optimal allocation is

[
n_h^*
=====

n
\frac{N_h S_h}
{\sum_{j=1}^{L} N_jS_j}.
]

Thus, strata with larger populations and greater variability receive larger sample sizes.

---

## 4. Fieldwork Allocation Model

After obtaining the Neyman targets (n_h^*), the next problem is to determine **which field team should collect those interviews**.

Let:

* (k = 1,\ldots,K) denote field teams
* (h = 1,\ldots,L) denote strata
* (x_{kh}) = number of interviews assigned to team (k) in stratum (h)
* (C_k) = maximum interview capacity of team (k)
* (c_{kh}) = cost per interview when team (k) works in stratum (h)

The exact Neyman allocation model can be formulated as the minimum-cost transportation problem:

[
\min
\sum_{k=1}^{K}
\sum_{h=1}^{L}
c_{kh}x_{kh}
]

subject to

[
\sum_{k=1}^{K}x_{kh}=n_h^*
\qquad \forall h,
]

[
\sum_{h=1}^{L}x_{kh}\leq C_k
\qquad \forall k,
]

[
x_{kh}\geq0.
]

This has a natural network-flow interpretation:

**Source → Team Nodes → Stratum Nodes → Sink**

where:

* Team nodes represent available field teams.
* Team capacities represent supply.
* Stratum requirements represent demand.
* Team–stratum costs represent transportation / fieldwork costs.

---

## 5. Flexible Neyman Allocation

Exact Neyman allocation may not always be the cheapest operational solution.

To study the trade-off between fieldwork cost and statistical accuracy, the project allows deviations from the Neyman targets.

Let

* (d_h^+) = positive deviation from the Neyman target
* (d_h^-) = negative deviation from the Neyman target

The allocation constraint becomes

[
\sum_{k=1}^{K}x_{kh}
====================

n_h^* + d_h^+ - d_h^-
\qquad \forall h.
]

The objective function is

[
\min
\left[
\sum_{k=1}^{K}\sum_{h=1}^{L}c_{kh}x_{kh}
+
\lambda
\sum_{h=1}^{L}
(d_h^+ + d_h^-)
\right].
]

Here, (\lambda) controls how strongly deviations from the Neyman allocation are penalized.

* Smaller (\lambda): greater emphasis on reducing fieldwork cost.
* Larger (\lambda): greater emphasis on staying close to Neyman allocation.

This allows the model to explore a practical **cost–precision trade-off**.

---

## 6. Dataset and Computational Experiment

A synthetic survey dataset containing **1,000 respondents** is used.

The population is divided into three strata:

* Rural
* Semi-Urban
* Urban

The study variable is **Monthly Income**, and the total desired sample size is **300**.

The resulting Neyman allocation is:

| Stratum    | Sample Size |
| ---------- | ----------: |
| Rural      |          59 |
| Semi-Urban |          56 |
| Urban      |         185 |
| **Total**  |     **300** |

The dataset also contains fieldwork-related variables used to construct the optimization inputs, including interview cost, travel distance, survey time, and travel difficulty.

---

## 7. Optimization Results

The minimum-cost fieldwork model is solved using **PuLP** and independently implemented using **Pyomo** for comparison.

The optimal allocation satisfies the Neyman sample requirements exactly:

| Stratum    | Required Interviews |
| ---------- | ------------------: |
| Rural      |                  59 |
| Semi-Urban |                  56 |
| Urban      |                 185 |
| **Total**  |             **300** |

The minimum fieldwork cost obtained for the exact Neyman allocation is approximately:

[
\boxed{6053.83}
]

The independent PuLP and Pyomo implementations provide a useful validation of the optimization result.

---

## 8. Sensitivity Analysis

### Effect of Team Capacity

The project studies the effect of changing the available interview capacity of the field teams.

The results show that increasing team capacity can reduce the minimum fieldwork cost because the optimization model has greater flexibility to assign interviews to lower-cost team–stratum combinations.

---

### Effect of the Penalty Parameter

The flexible Neyman model is evaluated for different values of (\lambda).

As (\lambda) increases, the optimized allocation gradually moves toward the exact Neyman allocation.

In the computational experiment, (\lambda = 0.90) was the smallest tested value that produced the exact Neyman allocation.

---

## 9. Statistical Precision

The variance of the stratified mean estimator is calculated for the different optimized allocations.

For the exact Neyman allocation:

[
\operatorname{Var}(\bar{Y}_{st})
================================

752660.94.
]

For the cheapest allocation:

[
(44,70,186),
]

the variance is

[
\operatorname{Var}(\bar{Y}_{st})
================================

783085.85.
]

This represents approximately a **4.04% increase in variance** relative to the Neyman benchmark.

An intermediate allocation,

[
(50,65,185),
]

produces only about a **1.46% increase in variance**.

Therefore, a small increase in statistical variance can potentially produce a reduction in fieldwork cost.

---

## 10. Cost–Precision Trade-off

The main conclusion of the computational experiment is that there is a clear trade-off between operational cost and statistical precision.

The project compares:

| Allocation              | Fieldwork Cost | Relative Variance |
| ----------------------- | -------------: | ----------------: |
| Cheapest allocation     |        6031.24 |            +4.04% |
| Intermediate allocation |        6038.35 |            +1.46% |
| Exact Neyman allocation |        6053.83 |          Baseline |

The results demonstrate that the statistically optimal allocation is not necessarily the cheapest operational allocation.

Instead, the optimization framework allows a decision-maker to choose an allocation that provides an appropriate balance between:

**Fieldwork Cost ↔ Statistical Precision**

---

## 11. Methodology

The complete computational workflow is:

```text
Synthetic Survey Dataset
          │
          ▼
   Stratified Analysis
          │
          ▼
   Neyman Allocation
          │
          ▼
  Target Sample Sizes
          │
          ▼
Minimum-Cost Flow Model
          │
          ├── PuLP
          │
          └── Pyomo
          │
          ▼
   Optimal Team Assignment
          │
          ▼
 Sensitivity Analysis
          │
          ▼
 Flexible Neyman Allocation
          │
          ▼
 Cost–Precision Analysis
```

---

## 12. Technologies Used

* **Python**
* **Jupyter Notebook**
* **Pandas**
* **NumPy**
* **Matplotlib**
* **Seaborn**
* **PuLP**
* **Pyomo**
* **NetworkX**
* **SciPy**

---

## 13. Repository Structure

```text
neyman-survey-fieldwork-optimization/
│
├── README.md
├── requirements.txt
├── neyman_fieldwork_allocation.ipynb
│
└── data/
    └── synthetic_survey_dataset.csv
```

---

## 14. How to Run

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/neyman-survey-fieldwork-optimization.git
cd neyman-survey-fieldwork-optimization
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open:

```text
neyman_fieldwork_allocation.ipynb
```

and run the cells sequentially.

---

## 15. Key Takeaways

This project demonstrates how **statistical sampling theory and mathematical optimization can be combined to solve a practical survey-planning problem**.

The main findings are:

* Neyman allocation provides the statistically efficient target sample sizes.
* Minimum-cost flow determines how field teams can practically collect those samples.
* Team capacity constraints significantly affect operational cost.
* Allowing controlled deviations from Neyman allocation can reduce fieldwork cost.
* These cost savings come with some loss of statistical precision.
* The penalty parameter (\lambda) provides a mechanism for controlling the balance between cost and statistical accuracy.
* Implementing the model using both PuLP and Pyomo provides an additional computational validation.

---

## 16. Future Improvements

Possible extensions include:

* Real-world survey datasets
* Geographic travel-time constraints
* Team availability by day
* Multiple survey waves
* Integer restrictions on assignments
* Team-specific fixed costs
* Maximum travel distance constraints
* Multi-objective optimization
* Robust optimization under uncertain costs
* Larger network-flow instances
* Interactive visualization of team–stratum assignments

---

## Author

**Dipendu Pal**

IE 501 — Optimization Models

Department of Industrial Engineering and Operations Research

---

## Project Status

**Completed computational prototype — further visualization and extensions planned.**

