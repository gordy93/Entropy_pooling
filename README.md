# Entropy Pooling vs Sequential Entropy Pooling (H1)
Simulation study and portfolio backtests.

This repository contains an end-to-end research notebook for a side-by-side comparison of:

- Entropy pooling (EP), (Meucci, 2008).
- Sequential entropy pooling (SeqEP) (H1), (Vorobets, 2021).

The notebook uses Fortitudo's open-source implementation (`fortitudo.tech`) to demonstrate the methodology, run controlled simulations and evaluate portfolio implications across multiple market regimes.

Third-party dependency (GPL-3.0): this notebook imports `fortitudo.tech` (GPL-3.0). All notebook code, simulation design and analysis are my own. The package is used strictly as a dependency to implement EP mechanics.

---

## What this project does

### 1) Simulation study (EP vs SeqEP)
Runs EP and SeqEP under 5 return regimes with 4 assets and evaluates:

- Relative entropy (RE) as a distortion measure.
- Effective number of scenarios (ENS). 
- Sensitivity to confidence-posterior _c_ in {1.00, 0.75, 0.50, 0.25}.
- Robustness via Monte Carlo random regime sampling.

### 2) Portfolio simulation
Builds naive long-only tilt portfolios using posterior expected returns and backtests:

- EP portfolios vs SeqEP portfolios vs equal-weighted Buy & Hold (B&H) benchmark.
- Daily, weekly and monthly rebalancing.
- Transaction costs.

### 3) Bayesian updating of confidence (extension)
Implements a Bayesian update to dynamically adjust the confidence weight _c_t_ based on how well recent realised returns match the "prior" vs "posterior" model.

---

## Core concepts and equations

### EP
EP constructs posterior scenario probabilities q by minimizing KL divergence to a prior probability vector p, while enforcing views:

`q* = argmin_q sum_{s=1..S} q_s * log(q_s / p_s)`

Subject to:
- Simplex constraint: sum_s q_s = 1, q_s >= 0
- Equality views: A q = b
- Inequality views: G q <= h

### SeqEP 
SeqEP applies constraints in stages (e.g., C0 then C1).

### Opinion pooling ("confidence-posterior") (Meucci, 2006)
If practitioners are less confident in their views, posterior distribution of the factors must shrink towards the reference factor distribution. For a given confidence level _c_ in [0, 1]:

`q(c) = (1 - c) p + c q_full`

With the pooling parameter _c_ being the confidence level in the views.

### Method comparison metrics

RE:
`RE(q||p) = sum_{s=1..S} q_s * log(q_s / p_s)`

ENS:
`ENS = exp(-RE)`

---

## Return regime simulator 

The notebook simulates returns using:
- A base mean vector mu and volatility vector sigma.
- A fixed correlation structure.
- Regime-dependent shock generations.

Specifically, crashes are simulated by:
- Probability _p_crash_, draw from a crash shock distribution.

---

## Views used in the notebook (set as example only)

Rank views are applied:

`E[R1] >= E[R2]
E[R3] <= E[R4]`

Expressed as inequality moment constraints:

`E[R2 - R1] <= 0
E[R3 - R4] <= 0`

---

## Portfolio construction

### Naive tilt allocation
Posterior expected returns are converted into long-only weights.

### Rebalancing and transaction costs
At each rebalancing date, turnover is calculated as:

`turnover = sum_i |w_i_target - w_i_current|`

Transaction cost is modeled as:

`cost = 0.0005 * turnover`

---

## Bayesian updating of confidence (dynamic _c_t_)

At each rebalance, the notebook can update the confidence weight _c_t_ with:
- A prior model (mu_p, Sigma_p).
- A posterior model (mu_q, Sigma_q).

Let E be the recent return, define:

`P(E|H) proportional to exp(ll_q)
P(E|not H) proportional to exp(ll_p)`

Then Bayes' rule updates:

`c_t = P(H|E)
    = (P(E|H) c_t) / (P(E|H) c_t + P(E|not H) (1 - c_t))`

A forgetting factor is used to prevent overreaction and to revert partially toward the initial confidence.

---

## How to run

Open and run:
- `Entropy_pooling_comparison.ipynb`

---

## Outputs

- Regime-by-regime plots of RE and ENS.
- Portfolio equity curves for EP, SeqEP, B&H across rebalancing frequencies and confidence-posterior.
- Performance tables.

---

## License / attribution

This notebook uses the `fortitudo.tech` package (GPL-3.0) as a third-party dependency.  
All notebook code in this repository are not related to Fortitudo; the package is only imported to illustrate its capabilities and its potential implementation.  
For the license terms, see the `fortitudo.tech` project’s GPL-3.0 license.
