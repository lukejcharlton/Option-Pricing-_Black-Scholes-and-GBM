# Option-Pricing-_Black-Scholes-and-GBM
A Python project comparing two independent methods for pricing European options — the closed-form Black-Scholes formula and Monte Carlo simulation of geometric Brownian motion (GBM) — applied to real historical data for Apple (AAPL) and Tesla (TSLA), with an out-of-sample validation of the historical volatility estimates used.

## What the project covers
- Data pipeline: 2 years of daily price data for Apple and Tesla via the yfinance API, split into a training period (volatility estimation) and a held-out test period (validation)
- Black-Scholes pricing: closed-form call/put pricing, validated against known reference values
- Monte Carlo pricing: GBM-based simulation of terminal stock prices, used to independently price the same options
- Convergence analysis: demonstrating that the Monte Carlo estimate converges to the Black-Scholes price as simulation count increases, consistent with the Law of Large Numbers (error shrinking ∝ 1/√N)
- GBM path simulation: visualising simulated future price trajectories against the real, realised price path for each stock
- Out-of-sample volatility validation: comparing training-period volatility estimates against realised volatility over the held-out period — the project's key empirical finding
- The Greeks: Delta, Gamma, and Vega, calculated three independent ways (Black-Scholes analytical, Black-Scholes finite-difference, Monte Carlo finite-difference) and cross-validated against each other
Greek sensitivity plots: Delta/Gamma/Vega curves across a range of hypothetical stock prices for both stocks

## Methodology
Train/test split. Both stocks' 2-year price histories are split at a shared cutoff date (13 March 2026). Volatility is estimated using only the training period (~75% of the data); the held-out ~6-month test period is used purely for validation, never for estimation — avoiding look-ahead bias. 

Volatility estimation. Daily log returns are calculated from the training period, and annualised historical volatility is derived as the standard deviation of those returns, scaled by √252 (trading days per year).

Pricing inputs. Both options are priced at-the-money (strike = spot price on the first day of the test period), with a 6-month time to expiry and a risk-free rate of 3.7% (US 3-month Treasury bill yield, 12 March 2026). 

Validation approach. Every method used has an independent cross-check built in:

- Black-Scholes is validated against known textbook reference prices
- Monte Carlo is validated against Black-Scholes via the convergence analysis
- Finite-difference Greeks are validated against analytical Greeks (Black-Scholes), then applied to the Monte Carlo pricer, which has no closed-form Greeks of its own
- Historical volatility estimates are validated against realised volatility over the held-out test period

## Key Results

### Estimated Volatility (Training Period)

| Stock | Annualised Volatility |
|-------|------------------------|
| Apple | 29.15% |
| Tesla | 62.24% |

---

### Option Prices (At-the-money, 6-month expiry, r = 3.7%)

| Stock | Type | Black-Scholes | Monte Carlo (N = 1,000,000) |
|--------|------|----------------|-----------------------------|
| Apple | Call | 22.67 | 22.69 |
| Apple | Put  | 18.10 | 18.11 |
| Tesla | Call | 71.15 | 71.57 |
| Tesla | Put  | 63.98 | 64.15 |

Black-Scholes and Monte Carlo agree closely across every combination, confirming both pricing implementations are consistent. Tesla is priced meaningfully higher than Apple throughout, directly reflecting its higher volatility.

---

### Out-of-sample Volatility Validation

| Stock | Estimated (training) | Realised (test period) | % Difference |
|--------|----------------------|------------------------|--------------|
| Apple | 29.15% | 27.51% | 5.61% |
| Tesla | 62.24% | 50.89% | 18.24% |

Historical volatility was a considerably more reliable estimator for Apple than for Tesla. This gap is consistent with real events over the period: the training window captured Tesla's decline from its December 2025 all-time high amid its first-ever annual delivery decline and mounting delivery-miss concerns, while the test window saw a comparatively calmer partial recovery following a Q1 2026 earnings beat. This illustrates a genuine limitation of constant-volatility models — a single historical estimate may not hold up equally well across stocks or time periods, particularly for names whose volatility is closely tied to discrete, unpredictable news events.

## Project structure
option-pricing/
├── README.md
├── main.ipynb      # full analysis notebook
└── requirements.txt
## Tools and libraries

- `numpy` — numerical computation, vectorised simulation  
- `pandas` — data handling and results tables  
- `scipy.stats` — normal distribution functions for Black-Scholes and the Greeks  
- `matplotlib` — visualisation  
- `yfinance` — historical stock price data  

## Limitations

- **Constant volatility.** Black-Scholes and the GBM simulation both assume volatility is fixed over the life of the option. Real volatility fluctuates, as directly demonstrated by the Tesla out-of-sample comparison above.  
- **Historical vs. implied volatility.** Volatility here is estimated from historical price data, not backed out from current option market prices (implied volatility) — the two are known to diverge, particularly around specific news events.  
- **European-style options only.** The Black-Scholes formula used applies to European options (exercisable only at expiry); no early-exercise (American-style) modelling is included.  
- **No dividends.** The model assumes the underlying pays no dividends over the option's life.  
- **Frictionless markets.** No transaction costs, bid-ask spreads, or liquidity constraints are modelled.  

## Possible extensions

- Compare model prices directly against real, currently-listed option chain data (via `yfinance`'s `.option_chain()`), and back out implied volatility to examine the volatility smile/skew that a constant-volatility model cannot capture

- Extend to a stochastic volatility model (e.g. Heston) to address the volatility-clustering behaviour observed in the Tesla results

- Add Theta and Rho to complete the standard Greek set

- Extend to American-style options via a binomial tree or least-squares Monte Carlo approach

