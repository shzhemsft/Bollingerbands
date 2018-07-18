# ~~Data science~~ Stock trading with Python 

## What are Bollinger Bands?

Created in the early 80s and named after its developer, Bollinger Bands represent a key technical trading tool for financial traders. Bollinger bands are plotted by two (2) standard deviations (a measure of volatility) away from the moving average of a price. Bollinger Bands allow traders to monitor and take advantage of shifts in price volatilities. Let’s examine the main components of this tool.

The main components of a Bollinger Bands are: 
+ Upper Band: The upper band is simply two standard deviations above the moving average of a stock’s price.
+ Middle Band: The middle band is simply the moving average of the stock’s price.
+ Lower Band: Two standard deviations below the moving average is the lower band.

According to John Bollinger’s official website, this tool primarily answers the question:

>Are prices high or low on a relative basis? By definition price is high at the upper band and price is low at the lower band. That bit of information is incredibly valuable. It is even more powerful if combined with other tools such as other indicators for confirmation.

Traders typically use this tool to spot trading opportunities through looking out for the contraction or expansion of these bands. Don’t worry if you don’t get the picture for now. You will notice these contractions/expansions when we start making the plots. 

## The formula for a typical 20 day Bollinger Band

It is worthwhile mentioning that the number of days under consideration for the moving average is entirely up to the analyst. That being said, it is quite common to see 20/21 day moving averages in practice as there are usually 20/21 trading days in a month.

```
Middle Band = 20 day moving average
Upper Band = 20 day moving average + (20 Day standard deviation of price x 2) 
Lower Band = 20 day moving average - (20 Day standard deviation of price x 2)
```

## Bollinger Band in Python

Let’s begin by making a small script that calls for the Adjusted Closing Prices of Microsoft from the [Alpha Vantage](https://www.alphavantage.co/) Stock API. The script then calculates the upper, moving average and lower bands. Finally, the 20-day Bollinger Band for Microsoft is visualised by the script.

```python
# import needed libraries
import pandas as pd
import matplotlib.pyplot as plt

raw = pd.read_csv('https://www.alphavantage.co/query?function=TIME_SERIES_DAILY_ADJUSTED&symbol=MSFT&outputsize=full&apikey=Q7B8REQMCEOPI1CZ&datatype=csv')
data = pd.DataFrame(raw)
data['timestamp'] = data['timestamp'].astype('datetime64[ns]')
data = data.sort_values(by='timestamp',ascending=True)
data = data.tail(500)

data['20 day moving average'] = data['adjusted_close'].rolling(window=20).mean()
data['20 day moving std'] = data['adjusted_close'].rolling(window=20).std()
data['upper band'] = data['20 day moving average'] + (data['20 day moving std'] * 2)
data['lower band'] = data['20 day moving average'] - (data['20 day moving std'] * 2)

# 20 Day Bollinger Band for Microsoft
data.plot(x="timestamp",y=["adjusted_close", "20 day moving average", "upper band","lower band"])
plt.title('Bollinger Band for Microsoft Stock')
plt.ylabel('Price (USD)')
plt.xlabel('Time')
plt.show()
```

Yay! The script seems to be working. Here is the output image:

![alt text](https://i.imgur.com/8NrM6Lk.png "Bollinger bands Figure 1")

How about we make this plot more readable by shading the critical areas between the upper and lower bands? Matplotlib can handle plot shading as well. Let’s tweak our basic plot with the help of Matplotlib’s fill_between method.

```python
# set style, empty figure and axes
plt.style.use('fivethirtyeight')
fig = plt.figure(figsize=(12,6))
ax = fig.add_subplot(111)

# Plot shaded Bollinger Band for Microsoft
x_axis = data['timestamp'].values
ax.fill_between(x_axis, data['upper band'], data['lower band'], color='grey')

# Plot Adjust Closing Price and Moving Averages
ax.plot(x_axis, data['adjusted_close'], color='blue', lw=2)
ax.plot(x_axis, data['20 day moving average'], color='black', lw=2)

# Set Title & Show the Image
ax.set_title('Bollinger Band For Microsoft')
ax.set_xlabel('Date (Year/Month)')
ax.set_ylabel('Price(USD)')
ax.legend()
plt.show()
```

Finally the shaded Bollinger Band is ready. Let’s take a look:

![alt text](https://i.imgur.com/acaAUMk.png "Bollinger bands Figure 2")

That look better and more readable doesn’t it? The grey area shows the full squeeze effect of upper and lower bands. The adjusted closing prices and its 20 trading days moving average can be seen and read better.

## Validating your trading strategy

Typically, traders use price movements below and above the upper and lower bands as buy/sell signals. For example, a price movement above the upper band seemingly indicates an unjustifiably high price of a security and a trader could profit by shorting at this level and buying back when the price moves back in the band. Similarly, a dip below the lower band is a strong buy signal. 

Sounds like we have a trading strategy: buy the dips (below the lower band) and sell the spikes (above the upper band). Or do we? 

Inspect your own graph of Microsoft stock prices with the following questions in mind:
+ Are spikes above the upper band *always* a sell signal?
+ Are dips below the lower band *always* a buy signal?
+ Overall, can we make profits by trading on the Bollinger Band?

## Bonus points: Bollinger Bands for Bitcoin

Using the same toolkit, can you generate the Bollinger Band for Bitcoin and generate your own trading strategy based on the chart patterns? 
+ Tip 1: Take a look at the cryptocurrency documentation on Alpha Vantage and try to download a sample CSV output: https://www.alphavantage.co/documentation/#currency-daily
+ Tip 2: We would like to look up Bitcoin (symbol = BTC) price movement in the US market (market = USD)
+ Tip 3: Unlike stocks, cryptocurrencies tend to be much more volatile (and unpredictable). Is Bollinger Band still a good signal for cryptocurrencies? 

#### Sample code ####

```python
# import needed libraries
import pandas as pd
import matplotlib.pyplot as plt

raw = pd.read_csv('https://www.alphavantage.co/query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=USD&apikey=Q7B8REQMCEOPI1CZ&datatype=csv')
data = pd.DataFrame(raw)
data['timestamp'] = data['timestamp'].astype('datetime64[ns]')
data = data.sort_values(by='timestamp',ascending=True)
data = data.tail(500)

data['20 day moving average'] = data['close (USD)'].rolling(window=20).mean()
data['20 day moving std'] = data['close (USD)'].rolling(window=20).std()
data['upper band'] = data['20 day moving average'] + (data['20 day moving std'] * 2)
data['lower band'] = data['20 day moving average'] - (data['20 day moving std'] * 2)

# set style, empty figure and axes
plt.style.use('fivethirtyeight')
fig = plt.figure(figsize=(12,6))
ax = fig.add_subplot(111)

# Plot shaded Bollinger Band for Bitcoin/USD
x_axis = data['timestamp'].values
ax.fill_between(x_axis, data['upper band'], data['lower band'], color='grey')

# Plot Adjust Closing Price and Moving Averages
ax.plot(x_axis, data['close (USD)'], color='blue', lw=2)
ax.plot(x_axis, data['20 day moving average'], color='black', lw=2)

# Set Title & Show the Image
ax.set_title('Bollinger Band For Bitcoin (BTC)')
ax.set_xlabel('Date (Year/Month)')
ax.set_ylabel('Price(USD)')
ax.legend()
plt.show()
```

![alt text](https://imgur.com/a/V5Mp0Mq "Bollinger bands Bitcoin")








