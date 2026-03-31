# Entropy_pooling

# Entropy Pooling vs Sequential Entropy Pooling (H1)
Simulation Study, Information Costs, and Portfolio Backtests

This repository contains my end-to-end research notebook for a side-by-side comparison of:

- Entropy Pooling (EP)
- Sequential Entropy Pooling (SeqEP), heuristic H1

The notebook uses Fortitudo's open-source implementation (`fortitudo.tech`) to demonstrate the methodology, run controlled simulations, and evaluate portfolio implications across multiple market regimes.

Third-party dependency (GPL-3.0): this notebook imports `fortitudo.tech` (GPL-3.0). All notebook code, simulation design, and analysis are my own. The package is used strictly as a dependency to implement EP mechanics.

---

## What this project does

### 1) Simulation study (EP vs SeqEP)
Runs EP and SeqEP under 5 return regimes with 4 assets, and evaluates:

- Relative Entropy (RE) as an information cost / distortion measure
- Effective Number of Scenarios (ENS) as a concentration/robustness proxy
- Sensitivity to view confidence c in {1.00, 0.75, 0.50, 0.25}
- Robustness via Monte Carlo regime sampling

### 2) Portfolio simulation
Builds naive long-only tilt portfolios using posterior expected returns and backtests:

- EP portfolio vs SeqEP portfolio vs Buy & Hold benchmark
- Daily / Weekly / Monthly rebalancing
- Transaction costs (5 bps per unit turnover)

### 3) Bayesian updating of confidence (optional extension)
Implements a Bayesian update to dynamically adjust the confidence weight c_t based on how well recent realized returns match the "prior" vs "posterior" model.

---

## Core concepts and equations

### Entropy Pooling (EP)
EP constructs posterior scenario probabilities q by minimizing KL divergence to a prior probability vector p, while enforcing views:

q* = argmin_q sum_{s=1..S} q_s * log(q_s / p_s)

Subject to:
- simplex constraint: sum_s q_s = 1, q_s >= 0
- equality views: A q = b
- inequality views: G q <= h

### Sequential Entropy Pooling (H1)
SeqEP (H1) applies constraints in stages (e.g., C0 then C1) instead of all at once. This can reduce unnecessary distortion when views interact.

### Confidence blending ("confidence-posterior")
For a given confidence level c in [0, 1], the notebook blends the prior and full posterior weights:

q(c) = (1 - c) p + c q_full

This provides a controlled way to test robustness to partial trust in the views.

### Information cost and robustness metrics

Relative entropy (RE):
RE(q||p) = sum_{s=1..S} q_s * log(q_s / p_s)

Effective number of scenarios (ENS):
ENS = exp(-RE)

(ENS is typically reported as a percentage relative to S.)

---

## Return regime simulator (4 assets)

The notebook simulates returns using:
- a base mean vector mu and volatility vector sigma
- a fixed correlation structure
- regime-dependent shock generation (Gaussian and non-Gaussian variants)

Non-Gaussian regimes are implemented via a mixture:
- with probability p_crash, draw from a crash shock distribution (shifted mean, higher variance)
- otherwise draw from the base shock distribution

---

## Views used in the notebook (example)

Rank views are applied, for example:

E[R1] >= E[R2]
E[R3] <= E[R4]

Expressed as inequality moment constraints for EP:

E[R2 - R1] <= 0
E[R3 - R4] <= 0

---

## Portfolio construction

### Naive tilt allocation
Posterior expected returns are converted into long-only weights:

w_i proportional to max(mu_i, 0), and sum_i w_i = 1

### Rebalancing and transaction costs
At each rebalancing date, turnover is:

turnover = sum_i |w_i_target - w_i_current|

Transaction cost is modeled as:

cost = 0.0005 * turnover

---

## Bayesian updating of confidence (dynamic c_t)

At each rebalance, the notebook can update the confidence weight c_t using the likelihood of recent returns under:
- a prior model (mu_p, Sigma_p)
- a posterior model (mu_q, Sigma_q)

Let E be the recent return block (chunk). Define:

P(E|H) proportional to exp(ll_q)
P(E|not H) proportional to exp(ll_p)

Then Bayes' rule updates:

c_t = P(H|E)
    = (P(E|H) c_t) / (P(E|H) c_t + P(E|not H) (1 - c_t))

A forgetting factor is used to prevent overreaction and to revert partially toward the initial confidence c0.

---

## How to run

Open and run:
- `Entropy_pooling_comparison.ipynb`

Typical dependencies:
- Python 3.9+
- numpy, pandas, scipy, matplotlib, seaborn, tqdm
- fortitudo.tech

---

## Outputs

- Regime-by-regime plots of RE vs confidence and ENS vs confidence
- Portfolio equity curves for EP / SeqEP / benchmark across rebalancing frequencies
- Performance tables (annualized return, Sharpe, Sortino, drawdown, turnover, skew, kurtosis)

---

## License / attribution

- All notebook code and analysis in this repository are my own.
- This project uses `fortitudo.tech` as an external dependency, which is licensed under GPL-3.0.

## Third-party dependency (GPL-3.0)

This notebook uses the `fortitudo.tech` package (GPL-3.0) as a third-party dependency.  
All notebook code in this repository is my own; the package is only imported to illustrate its capabilities and my implementation skills.  
For the license terms, see the `fortitudo.tech` project’s GPL-3.0 license.
