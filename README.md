# Optimal Survey Sampling and Fieldwork Allocation

## Neyman Allocation with Minimum-Cost Network Flow Optimization

A statistical optimization project combining **Neyman allocation** from stratified sampling with **minimum-cost fieldwork allocation** using linear/integer optimization.

The project asks a practical question:

> Given statistically desirable sample sizes across survey strata, how should limited field teams be assigned to those strata so that fieldwork cost is minimized without sacrificing too much statistical precision?

The model is demonstrated on a **synthetic survey dataset of 1,000 respondents**, divided into three strata: **Rural, Semi-Urban, and Urban**.

---

## Project Highlights

- Computes statistically efficient sample sizes using **Neyman allocation**.
- Models field teams as supply/capacity nodes and survey strata as demand nodes.
- Solves the fieldwork assignment problem as a **minimum-cost flow / integer linear program**.
- Implements the optimization independently using **PuLP** and **Pyomo**.
- Performs a **team-capacity sensitivity analysis**.
- Introduces a flexible Neyman formulation controlled by a penalty parameter **λ**.
- Quantifies the **cost–statistical-precision trade-off** using the variance of the stratified sample mean.

---

## Problem Setup

The survey population is divided into:

1. Rural
2. Semi-Urban
3. Urban

The study variable is **Monthly Income**.

Neyman allocation gives the statistically preferred sample size for each stratum. The project then adds operational constraints:

- each field team has a limited interview capacity;
- the cost of an interview depends on the team and stratum;
- the final allocation must satisfy the required sample size;
- in the flexible model, deviations from the Neyman targets are allowed but penalized.

The overall framework is:

**Statistical efficiency → Operational feasibility → Optimization**

---

# Mathematical Formulation

## 1. Neyman Allocation

For stratum $h$, let:

- $N_h$ = population size
- $S_h$ = within-stratum standard deviation
- $n$ = total sample size

The Neyman allocation used in the project is

$$
n_h^*
= $$

$$
\frac{N_h S_h}
{\sum_{j=1}^{L} N_j S_j}
$$

Thus, strata with larger populations and greater variability receive larger sample sizes.

---

## 2. Minimum-Cost Fieldwork Allocation

Let:

- $k=1,\ldots,K$ denote field teams
- $h=1,\ldots,L$ denote strata
- $x_{kh}$ = number of interviews assigned to team $k$ in stratum $h$
- $C_k$ = maximum interview capacity of team $k$
- $c_{kh}$ = cost per interview when team $k$ works in stratum $h$

The exact Neyman model minimizes

$$
\min
\sum_{k=1}^{K}
\sum_{h=1}^{L}
c_{kh}x_{kh}
$$

subject to

$$
\sum_{k=1}^{K} x_{kh} = n_h^*
\qquad \forall h
$$

$$
\sum_{h=1}^{L} x_{kh} \leq C_k
\qquad \forall k
$$

$$
x_{kh} \geq 0
$$

This is a transportation / minimum-cost-flow formulation.

The network interpretation is:

**Source → Team Nodes → Stratum Nodes → Sink**

---

## 3. Flexible Neyman Allocation

The flexible model allows the allocation to deviate from the exact Neyman targets.

Let:

- $d_h^+$ = positive deviation
- $d_h^-$ = negative deviation

The allocation constraint becomes

$$
\sum_{k=1}^{K}x_{kh}
$$
$$
n_h^* + d_h^+ - d_h^-
\qquad \forall h
$$

The objective becomes

$$
\min
\left[
\sum_{k=1}^{K}\sum_{h=1}^{L}c_{kh}x_{kh}
+
\lambda
\sum_{h=1}^{L}(d_h^+ + d_h^-)
\right]
$$

Here, $\lambda$ controls how strongly departures from Neyman allocation are penalized.

- Smaller $\lambda$ → greater emphasis on reducing fieldwork cost.
- Larger $\lambda$ → greater emphasis on staying close to Neyman allocation.

---

# Dataset

The computational experiment uses a synthetic survey dataset containing **1,000 respondents**.

| Stratum | Population |
|---|---:|
| Rural | 350 |
| Semi-Urban | 230 |
| Urban | 420 |
| **Total** | **1,000** |

The notebook uses the following variables:

- `Respondent_ID`
- `Stratum`
- `Monthly_Income`
- `Interview_Cost`
- `GPS_Distance_km`
- `Survey_Time_Min`
- `Travel_Difficulty`

Place the CSV at:

```text
data/synthetic_survey_dataset.csv
```

---

# Main Results

For a total sample size of **300**, the Neyman allocation is:

| Stratum | Neyman Sample |
|---|---:|
| Rural | 59 |
| Semi-Urban | 56 |
| Urban | 185 |
| **Total** | **300** |

### Team Capacities

| Team | Capacity |
|---|---:|
| Team A | 85 |
| Team B | 80 |
| Team C | 75 |
| Team D | 70 |

Total capacity = **310 interviews**.

### Optimal Exact-Neyman Fieldwork Allocation

| Team | Rural | Semi-Urban | Urban |
|---|---:|---:|---:|
| Team A | 0 | 0 | 85 |
| Team B | 0 | 0 | 80 |
| Team C | 9 | 56 | 0 |
| Team D | 50 | 0 | 20 |

The required stratum totals are exactly:

- Rural = 59
- Semi-Urban = 56
- Urban = 185

The minimum fieldwork cost is:

$$
\boxed{6053.83}
$$

---

# PuLP vs Pyomo

The exact allocation problem is solved independently using:

- **PuLP + CBC**
- **Pyomo + HiGHS**

Both implementations return the same optimal objective:

$$
6053.83
$$

This provides a computational cross-check of the optimization model.

---

# Capacity Sensitivity Analysis

| Capacity Factor | Total Capacity | Minimum Cost |
|---:|---:|---:|
| 1.00 | 310 | 6053.83 |
| 1.05 | 324 | 6033.21 |
| 1.10 | 340 | 6017.09 |
| 1.15 | 355 | 6002.37 |
| 1.20 | 372 | 5986.38 |
| 1.30 | 402 | 5959.74 |

Increasing team capacity gives the optimizer more freedom to use lower-cost team–stratum assignments, reducing minimum fieldwork cost.

---

# Cost–Precision Trade-off

The flexible model allows the allocation to move away from exact Neyman allocation.

The variance benchmark under the exact Neyman allocation is

$$
\mathrm{Var}(\bar{Y}_{st})
\approx 752660.94
$$

At the lowest-cost allocation tested,

$$
(44,70,186)
$$

the fieldwork cost is

$$
6031.24
$$

and the variance is

$$
783085.85
$$

This is approximately **4.04% higher** than the Neyman benchmark.

An intermediate allocation,

$$
(50,65,185)
$$

has cost

$$
6038.35
$$

with only about a **1.46% increase in variance**.

The exact Neyman allocation becomes optimal in the tested penalty grid at

$$
\lambda = 0.90
$$

Therefore, the optimization exposes a practical decision frontier:

> A small reduction in fieldwork cost can be obtained by accepting a controlled increase in statistical variance.

---

# Methodology

```text
Synthetic Survey Dataset
          |
          v
   Stratified Analysis
          |
          v
   Neyman Allocation
          |
          v
  Target Sample Sizes
          |
          v
Minimum-Cost Flow Model
          |
          +------ PuLP
          |
          +------ Pyomo
          |
          v
   Optimal Team Assignment
          |
          v
 Sensitivity Analysis
          |
          v
 Flexible Neyman Allocation
          |
          v
 Cost–Precision Analysis
```

---

# Repository Structure

```text
neyman-survey-fieldwork-optimization/
│
├── README.md
├── requirements.txt
├── .gitignore
├── neyman_fieldwork_allocation.ipynb
│
└── data/
    └── synthetic_survey_dataset.csv
```

For the first GitHub upload, this structure is sufficient. Results and figures can be added later.

---

# Installation

Python 3.10+ is recommended.

```bash
pip install -r requirements.txt
```

The Pyomo implementation uses the HiGHS interface and therefore requires `highspy`.

---

# Running the Project

Launch Jupyter:

```bash
jupyter notebook
```

Open:

```text
neyman_fieldwork_allocation.ipynb
```

Run the notebook cells from top to bottom.

The notebook:

1. loads the survey data;
2. computes Neyman allocation;
3. constructs team–stratum costs;
4. solves the minimum-cost fieldwork allocation with PuLP;
5. verifies the solution with Pyomo;
6. generates the network-flow visualization;
7. performs capacity sensitivity analysis;
8. performs flexible Neyman optimization;
9. calculates stratified-estimator variance;
10. generates the cost–precision analysis.

---

# Technologies

- Python
- NumPy
- Pandas
- Matplotlib
- NetworkX
- PuLP
- Pyomo
- HiGHS

---

# What This Project Demonstrates

This project connects two decision layers.

### Statistical Layer

**Neyman allocation** determines how many observations should ideally be collected from each stratum.

### Operations Research Layer

**Minimum-cost network flow / integer linear programming** determines how field teams should be assigned to achieve those targets under operational constraints.

The combined model balances:

$$
\boxed{\text{Statistical Accuracy}}
\quad\longleftrightarrow\quad
\boxed{\text{Fieldwork Cost}}
$$

rather than treating sampling design and field operations as separate problems.

---

# Limitations

The current experiment uses synthetic data and simplified operational assumptions.

In particular:

- team–stratum costs are constructed using a baseline cost and team-specific efficiency factors;
- team capacities are fixed;
- individual respondent routing is not explicitly modeled;
- interviewer availability is represented through aggregate capacity;
- the flexible model uses an absolute-deviation penalty from Neyman targets.

These assumptions make the optimization problem transparent and reproducible while leaving room for future extensions.

---

# Future Extensions

Possible extensions include:

- integer and mixed-integer formulations with additional fieldwork constraints;
- explicit travel-routing costs;
- interviewer scheduling;
- geographic clustering;
- multi-period fieldwork planning;
- uncertainty in travel cost and interviewer availability;
- robust optimization;
- stochastic programming;
- multi-objective optimization;
- Pareto-front analysis of cost versus variance;
- real survey datasets;
- interactive dashboards.

---

# Author

**Dipendu Pal**

IE 501 — Optimization Models  
Department of Industrial Engineering and Operations Research  
IIT Bombay

### Project Title

**Network Flow Models for Optimal Survey Sampling and Fieldwork Allocation**
