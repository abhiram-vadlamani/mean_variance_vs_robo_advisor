# Mean-Variance Portfolio Optimization on a Multi-Asset ETF Universe

End-to-end implementation of the Markowitz mean-variance framework on a
seven-asset ETF universe, benchmarked against a commercial robo-advisor's
published model allocation. The project estimates return moments, solves for the
minimum-variance and tangency (maximum-Sharpe) portfolios under a no-shorting
constraint, traces the efficient frontier, and derives the optimal capital
allocation between risky and risk-free assets under quadratic utility.

## Problem

Given $N$ assets with expected return vector $\mu$ and covariance matrix
$\Sigma$, find the weight vector $w$ (with $w^\top\mathbf{1}=1$,
$0.05 \le w_i \le 1$) that maximises the Sharpe ratio

$$S(w) = \frac{w^\top \mu - r_f}{\sqrt{w^\top \Sigma\, w}},$$

then determine how an investor with mean-variance utility
$U = E(r) - 4\alpha\sigma^2$ should split wealth between that tangency portfolio
and the risk-free asset. The 5% weight floor enforces diversification and limits
corner solutions driven by estimation noise in $\mu$.

## Data

- **Universe (7 ETFs):** `STW.AX` (AU equities), `VGS.AX` (intl equities,
  unhedged), `VGAD.AX` (intl equities, hedged), `DJRE.AX` (global property),
  `IFRA.AX` (global infrastructure), `IAF.AX` (AU fixed income), `AAA.AX` (cash /
  risk-free proxy).
- **Source:** Yahoo Finance daily adjusted close, via `yfinance`.
- **Window:** 1 Jul 2016 – 30 Jun 2021.
- **Transform:** daily simple returns; moments annualised by the 252-day
  convention.

## Method

1. Estimate $r_f$ from the cash ETF; estimate $\mu$ and $\Sigma$ for the six
   risky assets.
2. Solve the minimum-variance and tangency portfolios with SLSQP
   (`scipy.optimize`) under the budget and box constraints.
3. Trace the efficient frontier by minimising variance at each target return.
4. Derive the closed-form optimal risky weight
   $y^\* = \dfrac{E(r_p) - r_f}{8\alpha\sigma_p^2}$ and evaluate it at
   $\alpha = 1.7$.
5. Score the tangency portfolio and a commercial robo-advisor benchmark on the
   same universe and window.

## Results

| Portfolio | Ann. return | Ann. volatility | Sharpe |
|---|---|---|---|
| **Tangency (this study)** | 7.88% | 9.03% | **0.87** |
| Commercial robo-advisor benchmark | 5.74% | 8.46% | 0.68 |

- Tangency allocation concentrates in unhedged international equities (~51%) and
  Australian fixed income (~29%), with remaining sleeves at the 5% floor.
- Optimal capital allocation at $\alpha = 1.7$: **71% risky / 29% risk-free**.
- The benchmark sits inside the efficient frontier; the ~28% Sharpe improvement
  comes entirely from re-weighting the same assets. A commercial allocation whose
  *risky* composition changes with stated risk tolerance is inconsistent with the
  two-fund separation theorem, and that inconsistency is visible in the Sharpe
  differential.

## Limitations

- **In-sample.** Moments are estimated and evaluated on the same window, so the
  tangency portfolio's edge is an upper bound on out-of-sample performance.
- **Estimation error.** Mean-variance optimisation is highly sensitive to $\mu$;
  the weight floor mitigates but does not remove this.
- **No frictions.** Transaction costs, turnover, and tax are not modelled.

## Possible extensions

- Walk-forward (out-of-sample) backtesting with periodic rebalancing.
- Ledoit–Wolf shrinkage of the covariance matrix.
- Transaction-cost and turnover constraints; robust/Black-Litterman priors on
  $\mu$.

## Repository contents

```
Mean_Variance_Portfolio_Optimization.ipynb   # full analysis, plots, results
README.md
```

## Running

```bash
pip install numpy pandas scipy matplotlib yfinance
jupyter notebook Mean_Variance_Portfolio_Optimization.ipynb
```

Saved cell outputs (efficient-frontier plot, return time series, allocations)
are included in the notebook; a live run re-pulls data from Yahoo Finance and
may differ slightly as the API back-adjusts historical prices.
