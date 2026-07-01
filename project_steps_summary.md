# Pairs Trading Project: Step-by-Step Summary

This file records the steps completed so far in the pairs trading project.

## 1. Created the Project Folder

Created a project folder called `pairs_trading_project`.

The project structure includes folders for:

- `data/`
- `notebooks/`
- `src/`
- `outputs/`
- `reports/`

The purpose of this structure is to keep code, data, charts, and write-ups organized.

## 2. Added Project Setup Files

Added setup files including:

- `README.md`
- `requirements.txt`
- `.gitignore`

The `requirements.txt` file lists Python libraries needed for the project, such as:

- pandas
- numpy
- yfinance
- statsmodels
- matplotlib
- seaborn
- jupyter

## 3. Created the Main Jupyter Notebook

Created the main notebook for the project:

`notebooks/pairs_trading_strategy.ipynb`

This notebook is where the analysis, charts, cointegration tests, and future backtest will be built.

## 4. Added a Simple Notebook Title

Added a short title and summary at the top of the notebook:

```markdown
# Simple Pairs Trading Strategy Using Cointegration

Testing a mean-reversion pairs trading strategy using historical stock price data.
```

## 5. Imported Python Libraries

Imported the main Python libraries:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import yfinance as yf
import statsmodels.api as sm

from statsmodels.tsa.stattools import coint
```

These libraries are used for:

- working with stock price tables
- doing calculations
- creating charts
- downloading Yahoo Finance data
- running cointegration tests

## 6. Installed Missing Packages

The notebook first showed missing package errors for:

- `yfinance`
- `statsmodels`

These packages needed to be installed before the import cell could run successfully.

## 7. Downloaded Historical Stock Prices

Started with a small test list of stocks:

```python
tickers = ["KO", "PEP", "MCD", "SBUX", "WMT", "TGT"]
```

Downloaded historical closing prices from Yahoo Finance:

```python
prices = yf.download(
    tickers,
    start="2018-01-01",
    end="2024-12-31"
)["Close"]
```

This gave us daily closing prices from 2018 through the end of 2024.

## 8. Checked the Data

Used:

```python
prices.info()
```

The result showed:

- 1760 trading days
- 6 stock columns
- no missing values

This confirmed the downloaded data was clean enough to continue.

## 9. Plotted Historical Prices

Plotted all stock prices to visually inspect the data:

```python
prices.plot(figsize=(12, 6))

plt.title("Historical Closing Prices")
plt.xlabel("Date")
plt.ylabel("Price")
plt.show()
```

This helped us see how the stocks moved over time.

## 10. Calculated Daily Returns

Calculated daily percentage returns:

```python
returns = prices.pct_change().dropna()

returns.head()
```

Returns are useful because they put stocks on a more comparable scale than raw prices.

## 11. Created a Correlation Matrix

Calculated correlations between the stock returns:

```python
correlation_matrix = returns.corr()

correlation_matrix
```

The strongest correlation in the first small group was between:

```text
KO and PEP
```

Their correlation was about:

```text
0.737
```

This suggested KO and PEP were worth investigating first.

## 12. Plotted KO and PEP Prices

Plotted KO and PEP closing prices:

```python
prices[["KO", "PEP"]].plot(figsize=(12, 6))

plt.title("KO vs PEP Closing Prices")
plt.xlabel("Date")
plt.ylabel("Price")
plt.show()
```

This showed both stocks moving upward over time, but their raw prices were on different scales.

## 13. Normalized KO and PEP Prices

Normalized KO and PEP so both start at 100:

```python
normalized_prices = prices[["KO", "PEP"]] / prices[["KO", "PEP"]].iloc[0] * 100

normalized_prices.plot(figsize=(12, 6))

plt.title("KO vs PEP Normalized Prices")
plt.xlabel("Date")
plt.ylabel("Normalized Price")
plt.show()
```

This made it easier to compare their percentage movement over time.

## 14. Ran a Cointegration Test on KO and PEP

Tested whether KO and PEP were cointegrated:

```python
score, p_value, critical_values = coint(prices["KO"], prices["PEP"])

print("Cointegration test p-value:", p_value)
```

The p-value was about:

```text
0.508
```

Since this is higher than `0.05`, KO and PEP were not strongly cointegrated during this time period.

Important lesson:

```text
High correlation does not always mean cointegration.
```

## 15. Tested All Pairs in the Small Stock List

Tested every possible pair from the small stock list:

```python
from itertools import combinations

pair_results = []

for stock1, stock2 in combinations(prices.columns, 2):
    score, p_value, critical_values = coint(prices[stock1], prices[stock2])
    
    pair_results.append({
        "Stock 1": stock1,
        "Stock 2": stock2,
        "P-Value": p_value
    })

pair_results = pd.DataFrame(pair_results).sort_values("P-Value")

pair_results
```

The best p-value was around:

```text
0.21
```

That means none of the pairs in the small starter group were strongly cointegrated.

## 16. Current Conclusion

The project is on the right track.

So far, we have:

- downloaded stock data
- checked data quality
- visualized prices
- calculated returns
- checked correlations
- tested one pair for cointegration
- tested all pairs in the starter group

The main finding so far is:

```text
The first small stock group did not contain a strongly cointegrated pair.
```

This is not a problem. It means the research process is working.

## 17. Next Planned Step

The next step is to expand the ticker list to include more related stocks, such as:

```python
tickers = [
    "KO", "PEP",
    "WMT", "TGT", "COST",
    "MCD", "SBUX", "YUM", "CMG",
    "LOW", "HD",
    "CVX", "XOM",
    "JPM", "BAC", "WFC"
]
```

Then we will rerun:

1. data download
2. data check
3. returns calculation
4. correlation matrix
5. all-pairs cointegration test

The goal is to find a pair with a cointegration p-value below `0.05`.

## 18. Expanded the Ticker List and Found a Candidate Pair

Expanded the ticker list to include more related companies across restaurants, retail, energy, and banking.

After rerunning the all-pairs cointegration test, the best candidate pair was:

```text
MCD and YUM
```

The p-value was:

```text
0.033396
```

Since this is below `0.05`, MCD and YUM are a stronger candidate pair for the pairs trading strategy.

## 19. Plotted MCD and YUM Normalized Prices

Plotted MCD and YUM on a normalized scale so both stocks started at 100.

This made it easier to compare how the two stocks moved over time.

The normalized chart showed that MCD and YUM moved together fairly closely, especially after 2020.

## 20. Estimated the Hedge Ratio

Ran a regression of MCD prices on YUM prices:

```python
X = sm.add_constant(prices["YUM"])
y = prices["MCD"]

model = sm.OLS(y, X).fit()

hedge_ratio = model.params["YUM"]

print("Hedge ratio:", hedge_ratio)
```

The hedge ratio was:

```text
2.2219990094114594
```

The hedge ratio tells us how much of one stock is used to offset the other stock when building the spread.

In this project, the regression estimates the relationship:

```text
MCD ≈ constant + 2.222 × YUM
```

That means changes in YUM are scaled by about `2.222` when comparing it to MCD.

The hedge ratio will be used next to create the spread:

```text
spread = MCD - hedge_ratio × YUM
```

The spread is important because pairs trading looks for moments when this relationship temporarily moves away from normal and then returns toward normal.
