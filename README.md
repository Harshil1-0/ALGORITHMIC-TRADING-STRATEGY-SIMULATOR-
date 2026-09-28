# Algorithmic Trading Strategy Simulator

**Stock Price Prediction & Portfolio Optimization**

A Python simulator that predicts next-day prices for Indian large-cap stocks using Linear Regression, then allocates a fixed budget across those stocks using two algorithms, **Greedy** and **Dynamic Programming (unbounded knapsack)**, and compares the results. Output is printed to the console and saved as an Excel report with embedded charts.

Course project for **Design and Analysis of Algorithms**, Department of Mathematics, Mahindra University.
**Instructor:** Asst. Prof. Sayani Das

## Team

| Member | Roll No. | Contribution |
|--------|----------|--------------|
| Harshil Pansala | SE24UCAM051 | Final report, presentation, algorithm development |
| Pearl Mendapara | SE24UCAM043 | Coding and algorithm implementation |
| Amaan Rehman | SE24UCAM059 | Presentation and theory |
| Akash Raj | SE24UCAM054 | Presentation, theory and algorithm comparison |

## Features

- Fetches live and historical prices from Yahoo Finance (NSE tickers) via `yfinance`
- Predicts the next-day closing price with scikit-learn `LinearRegression` (80/20 train-test split, MSE reported)
- Optimizes portfolio allocation under a budget with:
  - **Greedy:** sort by value-to-cost ratio, buy the maximum of the best stock first. O(n log n)
  - **DP (unbounded knapsack):** optimal allocation with traceback of quantities. O(n × W)
- Prints a side-by-side comparison of both strategies
- Generates an Excel report (`.xlsx`) with prediction results, portfolio results, share quantities and charts

## Companies Analysed

| Key | Company | Ticker | Base expected return |
|-----|---------|--------|----------------------|
| `TCS` | Tata Consultancy Services | `TCS.NS` | 50% |
| `HDFC` | HDFC Bank | `HDFCBANK.NS` | 45% |
| `HUL` | Hindustan Unilever | `HINDUNILVR.NS` | 30% |
| `RELIANCE` | Reliance Industries | `RELIANCE.NS` | 35% |
| `MARUTI` | Maruti Suzuki | `MARUTI.NS` | 40% |

## How It Works

```
User Input → Fetch Prices → Predict Price → Optimize Portfolio → Excel Report
```

1. **Fetch prices:** current closing prices for all five companies.
2. **Build the stock list:** each stock becomes `(ticker, cost, value)`, where `cost` is the rounded current price and `value = cost × (1 + expected_return / 100)`.
3. **User input:** choose a company for prediction and enter a budget (₹).
4. **Predict:** trains Linear Regression on closing prices (`y = mx + b`, with x = days since the start of the window) and predicts the next day.
5. **Optimize:** runs Greedy and DP on the same budget.
6. **Report:** prints the comparison and saves an Excel file.

### Algorithms

**Greedy**
```python
stocks_sorted = sorted(stocks, key=lambda x: x[2] / x[1], reverse=True)
for ticker, cost, value in stocks_sorted:
    qty = remaining_budget // cost
    remaining_budget -= qty * cost
```

**Dynamic Programming (unbounded knapsack)**
```python
dp = [0] * (budget + 1)
for w in range(1, budget + 1):
    for ticker, cost, value in stocks:
        if cost <= w:
            dp[w] = max(dp[w], dp[w - cost] + value)
```
A `parent` array tracks which stocks were added so the exact share quantities can be recovered.

### Complexity

| | Greedy | Dynamic Programming |
|---|---|---|
| Time | O(n log n) | O(n × W) |
| Space | O(n) | O(W) |
| Solution quality | Near-optimal | Guaranteed optimal |
| Best for | Quick decisions, large budgets | Maximum returns |

(`n` = number of stocks, `W` = budget.)

## Installation

Requires Python 3.x and an internet connection (for Yahoo Finance data).

```bash
pip install yfinance numpy pandas scikit-learn matplotlib openpyxl
```

## Usage

```bash
python trading_simulator.py
```

You will be prompted for:

```
Enter company for price prediction: TCS
Enter your budget for portfolio optimization (₹): 500000
```

The full source is in **Appendix A** of the project report. Save it as `trading_simulator.py`.

### Output

- Console: current prices, predicted price with MSE, both portfolios, and the Greedy vs DP comparison
- `<TICKER>_prediction.png`: historical prices with the 1-day prediction
- `portfolio_comparison.png`: bar chart of Greedy vs DP total value
- `trading_simulator_results.xlsx` with three sheets: **Prediction**, **Portfolio**, **Stock Quantities**. If the file is open, a timestamped filename is used instead.

## Sample Results

From the report's sample run (Company: TCS, Budget: ₹500,000):

| Metric | Greedy | Dynamic Programming |
|--------|--------|---------------------|
| Total value | 1,245,680 | 1,268,920 |
| Budget used | 97.5% | 98.4% |
| Execution time | 0.003 s | 0.245 s |
| Optimality | Near-optimal | Optimal |

TCS predicted price: 3,268.75 (current: 3,245.50). DP improved on Greedy by about **1.9%**.

Greedy tends to concentrate the budget in one or two stocks with the best ratio, while DP spreads the budget across stocks to use it more fully.

## Limitations

- **Simple prediction model:** Linear Regression on time alone assumes a linear trend and will not capture volatility or market events. Predictions are not investment advice.
- **Prediction and optimization are independent:** the predicted price is reported for the chosen company but does not feed into the portfolio optimizer. The optimizer uses fixed base expected-return percentages.
- **DP scales with budget:** memory and time grow with `W`, so very large budgets get slow.
- **Educational project:** not intended for real trading.

## Future Scope

- LSTM, ARIMA and Random Forest models for prediction
- Risk metrics (Sharpe ratio, volatility, VaR) and genetic algorithms
- Multi-day forecasting (7 and 30 days) and a larger stock universe
- Backtesting, automated rebalancing, sentiment analysis
- Broker API integration, web dashboard and mobile app

## References

1. Markowitz, H. (1952). *Portfolio Selection.* Journal of Finance.
2. Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). *Introduction to Algorithms.* MIT Press.
3. Yahoo Finance API documentation.
4. scikit-learn: Machine Learning in Python.
5. National Stock Exchange of India (NSE), market data.
