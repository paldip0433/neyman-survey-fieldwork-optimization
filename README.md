# Optimal Survey Sampling and Fieldwork Allocation

## Neyman Allocation with Minimum-Cost Network Flow Optimization

A statistical optimization project that combines **Neyman allocation** from stratified sampling with **minimum-cost fieldwork allocation** using linear/integer optimization.

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
- Produces tables and publication-ready figures for the main optimization and sensitivity experiments.

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
- the final allocation must satisfy the required total sample size;
- in the flexible model, deviations from the Neyman targets are allowed but penalized.

This combines:

**Statistical efficiency → Operational feasibility → Optimization**

The project presentation describes the same framework as a network in which teams act as supply nodes, strata as demand nodes, and interviews as flow units.

---

## Mathematical Formulation

### 1. Neyman allocation

For stratum $h$, let:

- $N_h$ = population size,
- $S_h$ = within-stratum standard deviation,
- $n$ = total sample size.

The Neyman allocation used in the project is


$$
n_h^*
=
$$
n
$$
\frac{N_hS_h}
{\sum_{j=1}^{L}N_jS_j}.
$$

The presentation states that this allocation minimizes the variance of the stratified estimator subject to a fixed total sample size.

### 2. Minimum-cost fieldwork allocation

Let


$$
x_{kh}
$$

be the number of interviews assigned to team $k$ in stratum $h$, and let $c_{kh}$ be the corresponding unit fieldwork cost.

The exact Neyman model minimizes


$$
\min
\sum_k\sum_h c_{kh}x_{kh}
$$

subject to


$$
\sum_k x_{kh}=n_h^*
\qquad \forall h,
$$


$$
\sum_h x_{kh}\le C_k
\qquad \forall k,
$$


$$
x_{kh}\ge0.
$$

The project presentation identifies this as a transportation/minimum-cost-flow formulation.

### 3. Flexible Neyman allocation

The notebook additionally introduces positive and negative deviations $d_h^+$ and $d_h^-$:


$$
\sum_k x_{kh}
=
n_h^*+d_h^+-d_h^-.
$$

The objective becomes


$$
\min
\left[
\sum_k\sum_h c_{kh}x_{kh}
+
\lambda\sum_h(d_h^++d_h^-)
\right].
$$

Here, $\lambda$ controls how strongly departures from Neyman allocation are penalized.

This formulation creates a direct operational trade-off between fieldwork cost and statistical precision.

---

## Dataset

The computational experiment uses a synthetic survey dataset containing **1,000 respondents**.

The strata and observed population sizes in the notebook are:

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

### Data placement

Place the CSV at:

```text
data/synthetic_survey_dataset.csv
```

The uploaded notebook expects this path after the project is organized as a repository.

> The dataset itself is not embedded inside the notebook; it must be uploaded separately with the repository.

---

## Main Results

For a total sample size of **300**, the notebook obtains the Neyman allocation:

| Stratum | Neyman sample |
|---|---:|
| Rural | 59 |
| Semi-Urban | 56 |
| Urban | 185 |
| **Total** | **300** |

The project presentation reports the same allocation.

### Team capacities

| Team | Capacity |
|---|---:|
| Team A | 85 |
| Team B | 80 |
| Team C | 75 |
| Team D | 70 |

Total capacity = **310 interviews**.

### Optimal exact-Neyman fieldwork allocation

The minimum-cost solution is:

| Team | Rural | Semi-Urban | Urban |
|---|---:|---:|---:|
| Team A | 0 | 0 | 85 |
| Team B | 0 | 0 | 80 |
| Team C | 9 | 56 | 0 |
| Team D | 50 | 0 | 20 |

The required stratum totals are therefore exactly:

- Rural = 59
- Semi-Urban = 56
- Urban = 185

The minimum fieldwork cost is **6053.83**.

The project presentation reports that the model was solved using PuLP and independently implemented in Pyomo, with the same optimal solution and objective value.

---

## PuLP vs Pyomo

The notebook solves the exact allocation problem using both:

- **PuLP + CBC**
- **Pyomo + HiGHS (`appsi_highs`)**

Both implementations return the same objective:

```text
6053.83
```

This provides a useful cross-check of the optimization model.

---

## Capacity Sensitivity Analysis

The notebook changes total team capacity using multiplicative capacity factors.

| Capacity factor | Total capacity | Minimum cost |
|---:|---:|---:|
| 1.00 | 310 | 6053.83 |
| 1.05 | 324 | 6033.21 |
| 1.10 | 340 | 6017.09 |
| 1.15 | 355 | 6002.37 |
| 1.20 | 372 | 5986.38 |
| 1.30 | 402 | 5959.74 |

The result shows that additional field-team capacity gives the optimizer more freedom to use lower-cost team–stratum assignments, reducing minimum fieldwork cost. This is also the conclusion presented in the project slides.

---

## Cost–Precision Trade-off

The flexible model allows the allocation to move away from exact Neyman allocation.

The key benchmark is:


$$
\operatorname{Var}(\bar Y_{st})
\approx 752660.94
$$

under the exact Neyman allocation.

At the lowest-cost allocation tested:


$$
(44,70,186),
$$

the fieldwork cost is


$$
6031.24
$$

and the variance is


$$
783085.85,
$$

which is approximately **4.04% higher** than the Neyman benchmark.

An intermediate allocation,


$$
(50,65,185),
$$

has cost


$$
6038.35
$$

with only about a **1.46% increase in variance**.

The exact Neyman allocation becomes optimal in the tested penalty grid at


$$
\lambda=0.90.
$$

Thus the optimization exposes a practical decision frontier:

> a small reduction in fieldwork cost can be obtained by accepting a controlled increase in statistical variance.

These numerical conclusions match the project presentation's cost–precision analysis.

---

## Generated Outputs

The notebook creates:

### Results

```text
results/
├── neyman_allocation.csv
├── cost_matrix.csv
├── team_capacity.csv
├── optimal_allocation.csv
├── pyomo_allocation.csv
├── sensitivity_results.csv
├── lambda_sensitivity.csv
├── lambda_allocations.csv
├── lambda_team_allocations.csv
└── cost_precision_frontier.csv
```

### Figures

```text
figures/
├── neyman_allocation.png
├── optimal_allocation.png
├── network_flow.png
├── sensitivity_analysis.png
├── lambda_vs_cost.png
├── lambda_vs_deviation.png
├── lambda_vs_variance.png
├── lambda_allocation.png
└── cost_variance_frontier.png
```

The project presentation also emphasizes the network-flow interpretation and the cost–precision trade-off.

---

## Repository Structure

```text
neyman-survey-fieldwork-optimization/
│
├── README.md
├── requirements.txt
├── .gitignore
├── neyman_fieldwork_allocation.ipynb
│
├── data/
│   └── synthetic_survey_dataset.csv
│
├── results/
│   ├── neyman_allocation.csv
│   ├── cost_matrix.csv
│   ├── team_capacity.csv
│   ├── optimal_allocation.csv
│   ├── pyomo_allocation.csv
│   ├── sensitivity_results.csv
│   ├── lambda_sensitivity.csv
│   ├── lambda_allocations.csv
│   ├── lambda_team_allocations.csv
│   └── cost_precision_frontier.csv
│
└── figures/
    ├── neyman_allocation.png
    ├── optimal_allocation.png
    ├── network_flow.png
    ├── sensitivity_analysis.png
    ├── lambda_vs_cost.png
    ├── lambda_vs_deviation.png
    ├── lambda_vs_variance.png
    ├── lambda_allocation.png
    └── cost_variance_frontier.png
```

For the first GitHub upload, it is completely reasonable to start with only:

```text
README.md
requirements.txt
neyman_fieldwork_allocation.ipynb
data/synthetic_survey_dataset.csv
```

The `results/` and `figures/` directories can be committed later.

---

## Installation

Python 3.10+ is recommended.

```bash
pip install -r requirements.txt
```

For Pyomo, the notebook uses the HiGHS interface:

```python
SolverFactory("appsi_highs")
```

The required Python package is therefore `highspy`.

---

## Running the Project

Launch Jupyter:

```bash
jupyter notebook
```

Then open:

```text
neyman_fieldwork_allocation.ipynb
```

Run the cells from top to bottom.

The notebook:

1. loads the survey data;
2. computes Neyman allocation;
3. constructs team–stratum costs;
4. solves the minimum-cost fieldwork allocation with PuLP;
5. verifies the solution with Pyomo;
6. generates the network-flow visualization;
7. performs capacity sensitivity analysis;
8. performs λ-based flexible Neyman optimization;
9. calculates stratified-estimator variance;
10. generates the cost–precision frontier.

---

## Technologies

- Python
- NumPy
- Pandas
- Matplotlib
- NetworkX
- PuLP
- Pyomo
- HiGHS

---

## What This Project Demonstrates

This project connects two different decision layers:

### Statistical layer

**Neyman allocation**

determines how many observations should ideally be collected from each stratum.

### Operations-research layer

**Minimum-cost network flow / integer linear programming**

determines how field teams should be assigned to achieve those targets under operational constraints.

The combined model therefore balances:


$$
\boxed{\text{Statistical Accuracy}}
\quad\leftrightarrow\quad
\boxed{\text{Fieldwork Cost}}
$$

rather than treating sampling design and field operations as separate problems.

---

## Limitations

The current experiment is deliberately based on synthetic data and simplified operational assumptions.

In particular:

- team–stratum costs are constructed using a baseline cost and team-specific efficiency factors;
- team capacities are fixed;
- the current model does not explicitly include travel routes between individual respondents;
- interviewer availability is represented through aggregate capacity;
- the flexible model uses an absolute-deviation penalty from Neyman targets.

These assumptions make the optimization problem transparent and reproducible while leaving room for future extensions.

---

## Future Extensions

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

## Author

**Dipendu Pal**

IE 501 — Optimization Models  
Department of Industrial Engineering and Operations Research  
IIT Bombay

Project title used in the accompanying presentation:

**Network Flow Models for Optimal Survey Sampling and Fieldwork Allocation**.

