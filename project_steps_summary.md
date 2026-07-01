# Pairs Trading Project: Step-by-Step Summary

This file records the steps completed so far in the pairs trading project. It is meant to be a future reference guide, so each step includes what was done, why it was done, the result, and what the result means.

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

Result explanation:

All 6 stocks had the same number of observations, which means the price table was aligned correctly. There were no missing values, so we did not need to clean or drop any stock columns before continuing.

## 9. Plotted Historical Prices

Plotted all stock prices to visually inspect the data:

```python
prices.plot(figsize=(12, 6))

plt.title("Historical Closing Prices")
plt.xlabel("Date")
plt.ylabel("Price")
plt.show()
```

Result explanation:

The chart showed the stocks moving over time, but the raw prices were not always easy to compare because different stocks trade at different dollar levels. For example, one stock can trade near $50 while another trades near $150. This is why we later normalized prices.

## 10. Calculated Daily Returns

Calculated daily percentage returns:

```python
returns = prices.pct_change().dropna()

returns.head()
```

Result explanation:

The output showed daily percentage returns for each stock. For example, a value like `0.014` means the stock increased by about `1.4%` that day.

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

Result explanation:

KO and PEP had the strongest return correlation in the first group. A correlation of about `0.737` means their daily returns often moved in the same direction.

This suggested KO and PEP were worth investigating first.

Important note:

Correlation is not the same as cointegration. Correlation checks whether two stocks move together day to day. Cointegration checks whether their long-term price relationship is stable.

## 12. Plotted KO and PEP Prices

Plotted KO and PEP closing prices:

```python
prices[["KO", "PEP"]].plot(figsize=(12, 6))

plt.title("KO vs PEP Closing Prices")
plt.xlabel("Date")
plt.ylabel("Price")
plt.show()
```

Result explanation:

This showed both stocks moving upward over time, but their raw prices were on different scales. PEP traded at a much higher price level than KO, so the chart was not ideal for comparing percentage movement.

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

Result explanation:

After normalization, both KO and PEP started at `100`. This made it easier to compare their relative performance over time.

The chart showed that KO and PEP moved somewhat similarly, but visual similarity alone was not enough. We still needed a cointegration test.

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

Testing rule:

- If the p-value is below `0.05`, the pair may be cointegrated.
- If the p-value is above `0.05`, the pair is not strongly cointegrated.

Result explanation:

The KO/PEP p-value was about `0.508`, which is much higher than `0.05`.

This means KO and PEP were not strongly cointegrated during this time period, even though they had the highest return correlation in the small starter group.

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

This tested every possible pair from the 6-stock starter group. With 6 stocks, there are 15 possible pairs.

The best results from the first pair test were:

```text
KO / MCD      p-value ≈ 0.209986
SBUX / TGT    p-value ≈ 0.279586
MCD / PEP     p-value ≈ 0.316161
KO / PEP      p-value ≈ 0.508195
```

Result explanation:

The best p-value was about `0.21`, which is still higher than `0.05`.

That means none of the pairs in the small starter group were strongly cointegrated.

This was still a useful result because it showed the testing process worked. The data simply told us that the first small group did not contain a strong candidate pair.

## 16. Conclusion from the First Testing Round

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

The next step was to expand the ticker list to include more related stocks. The reason for expanding the list was that the first 6-stock group did not produce a cointegrated pair.

The expanded list included restaurants, retail companies, energy companies, and banks:

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

Then reran the same testing process:

1. Downloaded prices for the expanded ticker list.
2. Checked the data.
3. Calculated returns.
4. Tested every possible pair for cointegration.

With 16 stocks, the all-pairs test produced:

```text
120 rows x 3 columns
```

This means 120 different stock pairs were tested.

After rerunning the all-pairs cointegration test, the best candidate pair was:

```text
MCD and YUM
```

The p-value was:

```text
0.033396
```

Other near-candidates included:

```text
KO / LOW      p-value ≈ 0.064171
CMG / HD      p-value ≈ 0.064830
JPM / WMT     p-value ≈ 0.072478
LOW / YUM     p-value ≈ 0.092595
```

Result explanation:

MCD/YUM had a p-value of about `0.033396`, which is below `0.05`.

This means MCD and YUM are a stronger candidate pair for the pairs trading strategy.

The near-candidates were interesting, but their p-values were above `0.05`, so they were not selected at this stage.

## 19. Plotted MCD and YUM Normalized Prices

Plotted MCD and YUM on a normalized scale so both stocks started at 100.

This made it easier to compare how the two stocks moved over time.

The normalized chart showed that MCD and YUM moved together fairly closely, especially after 2020.

Result explanation:

This visual check supported the cointegration test result. The lines were not identical, but they showed a reasonably similar long-term movement pattern. That made MCD/YUM a reasonable pair to continue with.

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

Result explanation:

The hedge ratio was about `2.222`.

This means the strategy will compare MCD against approximately `2.222` units of YUM when building the spread.

This does not mean we are literally buying 2.222 shares in every case yet. At this stage, it is a statistical relationship used to create the spread. Later, it can be translated into a trading position.

## 21. Current Status After Testing

The project has now completed the first major research phase:

1. A small starter group was tested.
2. No cointegrated pair was found in that small group.
3. The stock list was expanded.
4. All possible pairs in the expanded list were tested.
5. MCD/YUM was selected as the first strong candidate pair.
6. A hedge ratio was estimated for MCD/YUM.

Main result so far:

```text
Selected pair: MCD / YUM
Cointegration p-value: 0.033396
Hedge ratio: 2.2219990094114594
```

Interpretation:

MCD and YUM passed the cointegration screen because their p-value was below `0.05`. This suggests their prices may have a stable long-term relationship. The hedge ratio gives us the scaling needed to build the spread between the two stocks.

## 22. Next Step

The next step is to build and plot the spread:

```python
spread = prices["MCD"] - hedge_ratio * prices["YUM"]
```

After that, we will calculate the spread's z-score. The z-score will help identify when the spread is unusually high or unusually low, which is what creates possible pairs trading signals.

## 23. Built the MCD/YUM Spread

Created the spread using the hedge ratio:

```python
spread = prices["MCD"] - hedge_ratio * prices["YUM"]

spread.head()
```

The first few spread values were negative, for example:

```text
2018-01-02   -13.360123
2018-01-03   -13.824718
2018-01-04   -14.412421
2018-01-05   -15.038669
2018-01-08   -15.403159
```

Result explanation:

The spread is the relationship we are actually trading. The negative values are not a problem. What matters is not whether the spread is positive or negative, but whether it moves away from its normal level and then returns.

## 24. Plotted the MCD/YUM Spread

Plotted the spread:

```python
spread.plot(figsize=(12, 6))

plt.title("MCD/YUM Spread")
plt.xlabel("Date")
plt.ylabel("Spread")
plt.show()
```

Result explanation:

The spread moved up and down over time instead of trending smoothly in only one direction. It showed large deviations followed by returns toward a more normal range.

This is useful because pairs trading depends on mean reversion. Mean reversion means the relationship temporarily moves away from normal and then comes back.

## 25. Calculated the Spread Z-Score

Calculated the z-score of the spread:

```python
spread_mean = spread.mean()
spread_std = spread.std()

z_score = (spread - spread_mean) / spread_std

z_score.head()
```

The first few z-score values were:

```text
2018-01-02    0.572676
2018-01-03    0.539532
2018-01-04    0.497606
2018-01-05    0.452931
2018-01-08    0.426928
```

Result explanation:

The z-score tells us how far the spread is from its average level.

- `0` means the spread is close to normal.
- `2` means the spread is unusually high.
- `-2` means the spread is unusually low.

The first few values were close to `0`, so there was no extreme trading signal at the beginning of the dataset.

## 26. Plotted the Z-Score with Trading Thresholds

Plotted the z-score and added threshold lines:

```python
z_score.plot(figsize=(12, 6))

plt.axhline(0, color="black", linestyle="--")
plt.axhline(2, color="red", linestyle="--")
plt.axhline(-2, color="green", linestyle="--")

plt.title("MCD/YUM Spread Z-Score")
plt.xlabel("Date")
plt.ylabel("Z-Score")
plt.show()
```

Result explanation:

The chart showed the spread crossing above `+2` several times and below `-2` a few times. These are the areas where the spread was unusually far from normal.

This is important because those threshold crossings can become trading signals.

## 27. Created Basic Trading Signals

Created a basic signal table:

```python
signals = pd.DataFrame(index=z_score.index)
signals["z_score"] = z_score
signals["position"] = 0

signals.loc[signals["z_score"] > 2, "position"] = -1
signals.loc[signals["z_score"] < -2, "position"] = 1

signals.head()
```

The first few rows showed:

```text
              z_score    position
2018-01-02    0.572676   0
2018-01-03    0.539532   0
2018-01-04    0.497606   0
2018-01-05    0.452931   0
2018-01-08    0.426928   0
```

Result explanation:

The position column uses:

- `1` for long the spread
- `-1` for short the spread
- `0` for no trade

The first few rows were all `0` because the z-score was not above `2` or below `-2`.

This is the first version of the trading signal logic. Later, we will improve it so positions stay open until the spread returns closer to normal.

## 28. Next Step

The next step is to count how many long and short signals were created, then improve the position logic so trades do not open and close too abruptly.
