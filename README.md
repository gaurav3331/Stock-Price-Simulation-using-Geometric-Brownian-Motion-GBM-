Stock Price Simulation using Geometric Brownian Motion (GBM)

A Monte Carlo simulation of stock prices using Geometric Brownian Motion. The notebook estimates drift and volatility from real historical data, simulates 1,000 possible one-year price paths, computes risk metrics (VaR and Expected Shortfall), and backtests the model on held-out data.

Note: This project is for learning and educational purposes only. It is not financial advice.

Table of Contents
Overview
The Math
Workflow
Results
Backtest and Model Limitations
Installation
Usage
Configuration
Project Structure
Future Improvements
Tech Stack
Overview

Nobody can predict the exact future price of a stock, but we can model the range of likely outcomes. This project:

Downloads historical prices from Yahoo Finance
Estimates annualized expected return (μ) and volatility (σ) from log returns
Generates Brownian motion (random walk) paths
Applies drift-diffusion dynamics to simulate future price trajectories
Summarizes the outcomes using distribution and risk metrics
Backtests the model on data it has not seen

If Yahoo Finance is unreachable, the notebook falls back to a synthetic price series so it still runs end to end.

The Math

Stock prices are modeled by the stochastic differential equation:

dS_t = μ · S_t · dt + σ · S_t · dW_t

which has the exact solution:

S_t = S_0 · exp[ (μ − ½σ²)·t + σ·W_t ]
Symbol	Meaning
S_t	Stock price at time t
μ	Annualized drift (average trend)
σ	Annualized volatility
W_t	Standard Brownian motion (Wiener process)

Parameter estimation from daily log returns r_t = ln(S_t / S_{t-1}):

σ̂ = std(r_t) · √252
μ̂ = mean(r_t) · 252 + ½σ̂² (the ½σ² correction comes from Itô's lemma)

The simulation uses the exact solution rather than an Euler approximation, so there is no discretization error.

Workflow
Step	Description
1. Data	Download adjusted close prices (2020 to present) with yfinance
2. Estimation	Compute log returns, annualized σ and μ, skewness and kurtosis
3. Brownian motion	Generate Wiener process paths and verify Var[W(T)] ≈ T
4. Simulation	Build 1,000 GBM price paths over 252 trading days
5. Risk analysis	Terminal price distribution, percentiles, VaR, Expected Shortfall
6. Backtest	Train on the first 80% of data, simulate the last 20%, compare to actual prices
Results

Sample output from one run (2020-01-01 to 2026-10-16, 1,683 daily observations for the chosen ticker):

Estimated parameters

Metric	Value
Annualized volatility (σ)	28.17%
Annualized mean log return	9.41%
GBM drift (μ)	13.38%
Skewness	-0.131
Excess kurtosis	5.358

1-year forward simulation (1,000 paths)

Metric	Value
Starting price	1,161.20
Mean terminal price	1,327.78
Theoretical mean S0·e^(μT)	1,327.39
Median terminal price	1,272.36
5th percentile	800.72
95th percentile	2,039.74
P(S_T > S_0)	62.2%
95% VaR (return)	-31.04%
95% Expected Shortfall	-37.66%

Key takeaways

The simulated mean (1,327.78) closely matches the theoretical mean (1,327.39), confirming the implementation is correct.
The mean is higher than the median because terminal prices are lognormally distributed, so a few very large upside paths pull the mean up.
In the worst 5% of scenarios, the simulated loss is at least 31%, and the average loss within that tail is about 38%.

Generated plots: historical prices, daily log returns with normal fit, Brownian motion paths, simulated GBM fan chart, history plus forecast with 5-95% band, terminal price histogram with lognormal overlay, and the backtest chart.

Backtest and Model Limitations

In the backtest, only 76.3% of actual prices fell inside the simulated 90% confidence band. A well-calibrated model would be close to 90%, so GBM is overconfident on this stock.

The main reasons:

Fat tails: the excess kurtosis is 5.36, while GBM assumes 0. Extreme moves happen more often than a normal distribution predicts.
Volatility clustering: real volatility is not constant. Calm and turbulent periods come in clusters.
Jumps: earnings releases and news cause sudden price gaps that GBM cannot produce.
Noisy drift: μ is very hard to estimate from a few years of data, so long-horizon forecasts are dominated by uncertainty.

GBM is best understood as a baseline model for describing uncertainty, not a tool for predicting prices.

Installation
bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install yfinance numpy pandas matplotlib scipy jupyter
Usage
bash
jupyter notebook gbm.ipynb

Then run all cells (Kernel > Restart & Run All).

Configuration

Edit the settings cell at the top of the notebook:

python
TICKER = "YOUR_TICKER"              # any Yahoo Finance ticker
START, END = "2020-01-01", None     # END=None means today
TRADING_DAYS = 252
HORIZON_DAYS = 252                  # forecast horizon in trading days
N_PATHS = 1000                      # number of simulated paths
SEED = 42                           # for reproducibility

The notebook works with any ticker available on Yahoo Finance, so you can swap in a different stock and re-run it.

Project Structure
.
├── gbm.ipynb      # main notebook (data, estimation, simulation, risk, backtest)
└── README.md
Future Improvements
Heston model for stochastic volatility
Merton jump-diffusion to capture sudden price gaps
GARCH for time-varying volatility
Correlated multi-asset GBM using Cholesky decomposition for portfolio simulation
Option pricing via Monte Carlo under the risk-neutral measure
Rolling-window backtests with formal coverage tests (e.g., Kupiec test)
Tech Stack

Python 3.11, NumPy, pandas, SciPy, Matplotlib, yfinance, Jupyter

Disclaimer

This project is for educational purposes only. Simulated results are based on historical data and a simplified model, and do not predict or guarantee future performance. Do not use it for investment decisions.
