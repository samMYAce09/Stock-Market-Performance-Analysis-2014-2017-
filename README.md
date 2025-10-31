# Stock-Market-Performance-Analysis-2014-2017
 “The stock market is filled with individuals who know the price of everything, but the value of nothing.”  
> — *Philip Fisher*

## Introduction

Financial markets are living systems. They breathe, rise, fall, stall, and surge again in cycles shaped by human sentiment, innovation, global news, and economic structure. To study the behavior of stocks is to study motion over time — not a single number, but a dance of many. In this analysis, I focused on stock price performance across a four-year range, from 2014 to 2017. The objective was not only to measure price changes, but to interpret them — to understand the rhythm of movement through indicators like daily returns, volatility, rolling averages, and cumulative price growth.

To work through these questions, I combined Python and Power BI. Python provided the analytical foundation, enabling me to clean raw data, compute time-based trends, and engineer meaningful metrics that help quantify stability, growth, and risk. Power BI then transformed these numerical results into an interactive narrative, allowing patterns to be seen rather than merely calculated. The two tools complement each other: Python handles precision and transformation; Power BI handles storytelling and exploration.

This project is therefore both a technical and interpretive journey. It invites the reader not just to look at numbers, but to understand them.


## About the Dataset

The dataset contains daily stock price information for multiple companies traded in the U.S. market. Each row represents a trading day for a given stock and includes columns such as:

- symbol (the ticker code, such as AAPL or AMZN for stock name)

- date (the trading session date, weekends and holidays excluded since the stock market only operates on weekdays)

- open, high, low, close (the recorded price levels for that session)

- volume (the number of shares traded) 

Because the market is closed on weekends and holidays, the dataset is naturally missing those days. This is normal and crucial in how we calculate time-dependent statistics. When computing change from "previous day," we must use the previous trading day, not merely the previous calendar date.

The time range spans 2014 to 2017, a period of steady market expansion and strong performance among several major technology and retail companies. This dataset is large enough to observe real trends and long enough to meaningfully separate short-term noise from long-term movement.

## Data Cleaning and Preparation in Python

The first step in analysis is always making the dataset usable. Before analysis, the dataset needed cleaning. This step ensures the data is consistent, complete, and ready for computation. I handled missing or inconsistent values, standardized date formats, and confirmed that each record had valid numerical entries.  

In Python, data cleaning was performed using pandas. For example:

```python
import pandas as pd

stock_data = pd.read_csv("stock_prices.csv")
stock_data['date'] = pd.to_datetime(stock_data['date'])
stock_data = stock_data.dropna()
```
This ensures that the date column is in datetime format and that rows containing empty values are removed. It’s often said that “clean data is half the analysis,” and that’s especially true here

We also ensured that the date column is formatted correctly. Sorting ensures that calculations involving previous-day values work accurately.

```python
stock_data = stock_data.sort_values(['symbol', 'date']).reset_index(drop=True)
```
This part seems simple, but in time-series analysis it is foundational. If the dates are not sorted, we cannot calculate daily differences. If the dates are not real datetime values, rolling averages and period-based comparisons will fail. Data preparation is quiet work, but it is the scaffolding upon which the entire analysis rests.

## Feature Engineering and Computations

Once the dataset was clean, I computed several financial indicators that form the foundation of this analysis.

- Moving Averages

To identify stock trends, I calculated 7-day and 30-day moving averages using the following Python code:

```python
stock_data['MA_7'] = stock_data['close'].rolling(window=7).mean()
stock_data['MA_30'] = stock_data['close'].rolling(window=30).mean()
```
These moving averages help smooth out daily price fluctuations. The 7-day moving average highlights short-term momentum, while the 30-day moving average provides a long-term view. When the short-term average crosses above the long-term average, it often signals a potential buy opportunity. Conversely, when it drops below, it may indicate a sell signal.

This approach helps investors avoid reacting emotionally to day-to-day volatility, focusing instead on broader patterns. As the famous saying goes, “In the short run, the market is a voting machine, but in the long run, it is a weighing machine.”

- Trend Analysis

To quantify each stock’s overall performance across the four-year window, I calculated the percentage change in closing prices:

```python
price_change = ((stock_data['close'].iloc[-1] - stock_data['close'].iloc[0]) /
                stock_data['close'].iloc[0]) * 100
trend = "UPWARD" if price_change > 0 else "DOWNWARD"
print(f"Overall trend: {trend} ({price_change:+.2f}% change)")
```
This calculation identifies whether a company’s stock experienced growth or decline. For example, Apple’s stock price rose by more than 114% from 2014 to 2017  a strong upward trajectory that would attract investors.

- Volatility Analysis

Volatility represents how much a stock’s price fluctuates daily. High volatility suggests a riskier stock, while low volatility signals stability. Using standard deviation of daily returns, volatility was computed as follows:

```python
stock_data['DailyReturn'] = stock_data['close'].pct_change()
volatility = stock_data['DailyReturn'].std() * 100
```
A volatility of 1.43%, for example, means that on average, the stock’s daily returns deviate by 1.43% from the mean. This is crucial for risk assessment — investors with low risk tolerance may prefer lower volatility stocks, even if they offer smaller returns.

- Average Daily Volume

The volume column tells how many shares are traded per day. I calculated the average daily volume to understand market activity:

```python
avg_volume = stock_data['volume'].mean()
```
High trading volume suggests strong investor interest and liquidity. For instance, Apple’s daily volume averaged around 45 million shares, highlighting its dominant position in the market.


## Power BI Integration and Visualization

After performing the analysis in Python, I exported the cleaned dataset and imported it into Power BI for visualization. Power BI allows users to interact with data through slicers, filters, and visual storytelling.

In Power BI, I created several calculated columns and measures using DAX. These included metrics like Volatility (%), Price Change (%), and Investment Signal, each designed to help users make data-driven investment decisions.

```DAX
Volatility (%) =
VAR CleanReturns =
    FILTER(
        StockPrices,
        NOT ISBLANK(StockPrices[DailyReturn])
    )
VAR StdDevValue =
    STDEVX.P(CleanReturns, StockPrices[DailyReturn])
RETURN
IF(ISBLANK(StdDevValue), BLANK(), StdDevValue * 100)
```
This measure captures the daily variation in returns while respecting the absence of weekend data.

Investment Signal:
```DAX
Investment Signal =
VAR Change = [Price Change (%)]
VAR Vol = [Volatility (%)]
VAR TextColor = "#000000"
VAR BuyColor = "#C7F2C4"
VAR HoldColor = "#FFF3B0"
VAR SellColor = "#F4B6B6"
VAR StatusText =
    SWITCH(
        TRUE(),
        Change > 20 && Vol < 2, "BUY",
        Change > 0 && Vol <= 5, "HOLD",
        Change <= 0, "SELL",
        "HOLD"
    )
VAR FillColor =
    SWITCH(
        TRUE(),
        StatusText = "BUY", BuyColor,
        StatusText = "HOLD", HoldColor,
        StatusText = "SELL", SellColor
    )
RETURN
"data:image/svg+xml;utf8,
<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 200 60' width='100%' height='25'>
    <rect x='0' y='5' width='200' height='50' rx='25' ry='25' fill='" & FillColor & "' />
    <text x='100' y='38' font-size='25' text-anchor='middle' fill='" & TextColor & "' font-family='Segoe UI, sans-serif'>
        " & StatusText & "
    </text>
</svg>"
```
This measure visually encodes “BUY,” “HOLD,” and “SELL” recommendations in a way that users can immediately recognize through color and text.


## Visual Insights and Interpretation

- Chart 1: Stock Trend Over Time



This chart displays how each stock’s price evolved between 2014 and 2017. Clear upward trajectories, like Apple’s, reveal strong investor confidence, while flatter or downward slopes indicate underperformance or market correction periods.

- Chart 2: Moving Averages Comparison



The 7-day and 30-day moving averages show how short-term momentum interacts with long-term stability. When the short-term line crosses above the long-term line, it suggests positive momentum — a potential buying opportunity.

- Chart 3: Volatility Distribution


This visualization shows which stocks are more volatile. A high volatility percentage warns investors of possible rapid price swings — useful for risk-conscious investors.

- Chart 4: Volume Trend


Analyzing average trading volume highlights which stocks attract consistent investor attention. High volume suggests liquidity, making it easier to buy and sell shares without large price impacts.

- Chart 5: Investment Summary Table



The summary table includes columns such as Symbol, Number of Trading Days, Lowest Price, Highest Price, Trend, Volatility (%), Average Daily Volume, and Investment Signal.

This table offers investors a clear side-by-side comparison of multiple companies. The Investment Signal column acts as a guide — showing whether a stock is worth buying, holding, or selling based on combined performance metrics.

For example, a stock with a high price increase and low volatility would likely display “BUY,” while one with flat or negative growth and high volatility would show “SELL.”


## Conclusion

This project demonstrates how combining Python and Power BI can turn complex stock data into actionable insights. Python provided the computational backbone for analyzing historical trends and volatility, while Power BI transformed these numbers into a visual narrative accessible to all.

Understanding stocks isn’t about predicting the future — it’s about observing patterns, managing risk, and recognizing opportunities. This analysis allows investors to do just that by revealing which companies are stable, which are risky, and which are poised for growth.

As Warren Buffett once said, “The best investment you can make is in yourself.” In this context, learning to interpret financial data is one of the most valuable investments of all.

**Author**

**Falola** **Samuel**
**Data Analyst** | **Python** & **Power BI** **Expert**
