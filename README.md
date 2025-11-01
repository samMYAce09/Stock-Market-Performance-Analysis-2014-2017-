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

The Power BI dashboard provides the interactive layer of the analysis. While Python helped us compute trends, averages, and volatility, Power BI allows these insights to be seen, compared, and explored dynamically. This section explains what the visuals represent and how they connect to the story of stock performance between 2014 and 2017.

- Dashboard Overview

![KPIs](images/chartstockpriceKPI.png)
At the top of the dashboard, three key summary metrics provide a high-level view of the market during the time period analyzed. The Average Stock Price of $86 reflects the central price tendency across all stocks studied. The Total Trading Volume of 2.12T highlights the significant level of market activity and liquidity. The percentage of stocks that demonstrated positive performance, 75.45%, suggests a generally upward market direction during the time span. Together, these values help frame the narrative of the market as active and growth-oriented.

- How Stock Prices Evolved Over Time

Directly below the KPI metrics, a bar chart visualizes how selected stock prices changed across the years 2014 to 2017. The increasing height of the bars for many stocks indicates consistent upward price momentum, hinting at periods of strong corporate performance or broader economic optimism. For other stocks, flatter trends show stability without significant growth, while any noticeable declines suggest challenges unique to those businesses or sectors. This chart allows users to understand long-term performance at a glance.

- Monthly Trading Activity Patterns

On the right side of the dashboard, a line chart illustrates total trading activity across the months. The shape of the curve reveals that trading volume fluctuates with the calendar cycle. Certain months display increased volume, which may align with earnings releases, product announcements, or major market events. Lower months may indicate seasonal slowdowns. This pattern helps viewers recognize when the market tends to be more active, which supports timing-related investment decisions.

- Stocks with the Highest Average Prices

This visualization ranks the stocks by their average trading price. One stock clearly stands out with an average price exceeding $1,300, while the next tier falls between $500 and $700. This helps reinforce that not all high prices imply high performance; rather, prices reflect valuation, perceived value, and market position. The chart gives a sense of how stocks compare to one another in terms of capital weight and investor expectations.

- Trading Activity Distribution

A bar chart highlights which stocks experienced the greatest overall trading activity during the time period. Stocks with significantly higher activity may have attracted increased investor interest, media exposure, or strategic speculation. Lower-activity stocks, on the other hand, may trade in more stable or niche markets. This helps investors understand which stocks are frequently traded and therefore easier to enter and exit positions in.

- Detailed Stock Performance Overview

The stock performance table consolidates all computed metrics into a single view that supports interpretation and comparison.

Each column carries meaning:

- *Stocks* identifies the ticker symbol.
- *Number of Trading Days* shows dataset completeness and trading availability.
- *Lowest and Highest Price* reveal the price spread observed during the period.
- *Short-Term 7-Day Moving Averag* reflects the recent price trend.
- *Long-Term 30-Day Moving Average* captures broader market direction.
- *Price Change (%)* measures whether the stock gained or lost value overall.
- *Average Daily Volume* indicates how actively the stock trades.
- *Volatility (%)* measures how much prices fluctuate daily, representing investment risk.
- *Investment Signal* gives a simplified interpretation of the performance outlook.

- Understanding the Investment Signal
The Investment Signal is derived from Price Change (%) and Volatility (%), combining trend and risk into a single interpretation.

| Signal   | Interpretation                                                                                    |
| -------- | ------------------------------------------------------------------------------------------------- |
| **BUY**  | Strong upward trend with controlled volatility. Indicates confidence and potential future growth. |
| **HOLD** | Stable or moderately positive trend. Suggests monitoring for shifts.                              |
| **SELL** | Negative or inconsistent performance. Indicates caution or exit consideration.                    |


For example, a stock like AAPL showing steady price growth with low volatility receives a BUY recommendation. Conversely, AAP, which reflects price decline despite moderate volatility, receives a SELL recommendation.

This table allows investors to evaluate performance and stability together, illustrating not only how the stock behaved but also how reliable that performance may be.

## Conclusion

This project demonstrates how combining Python and Power BI can turn complex stock data into actionable insights. Python provided the computational backbone for analyzing historical trends and volatility, while Power BI transformed these numbers into a visual narrative accessible to all.

Understanding stocks isn’t about predicting the future — it’s about observing patterns, managing risk, and recognizing opportunities. This analysis allows investors to do just that by revealing which companies are stable, which are risky, and which are poised for growth.

As Warren Buffett once said, “The best investment you can make is in yourself.” In this context, learning to interpret financial data is one of the most valuable investments of all.

**Author**

**Falola** **Samuel**
**Data Analyst** | **Python** & **Power BI** **Expert**
