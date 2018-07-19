# Stock visualization and trading strategies with Python 

First, please make sure that your pandas version is greater than 0.20. 

```
pip show pandas
```
And update if necessary: 

```
pip install --upgrade pandas
```

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

![alt text](https://i.imgur.com/69mzJww.png "Bollinger bands Bitcoin")

## Bonus Points 2: other trading signals

In the language of quantitative investing, Bollinger Band is part of a family of trading signals called **technical indicators**. A technical indicator is a mathematical calculation based on historic price and volume information that aims to forecast financial market direction. Technical indicators are fundamental part of technical analysis and are typically plotted as a chart pattern to try to predict the market trend. Indicators generally overlay on price chart data to indicate where the price is going, or whether the price is in an "overbought" condition or an "oversold" condition. 

We now introduce three popular technical indicators that are regularly used *in place of* or *in addition to* the Bollinger Band. 

### SMA/EMA

A simple moving average (SMA ) is an arithmetic moving average calculated by adding recent closing prices and then dividing that by the number of time periods in the calculation average. A simple, or arithmetic, moving average that is calculated by adding the closing price of the security for a number of time periods and then dividing this total by that same number of periods. 

An exponential moving average - EMA is a type of moving average that places a greater weight and significance on the most recent data points. The exponential moving average - EMA is also referred to as the exponentially weighted moving average. Exponentially weighted moving averages react more significantly to recent price changes than a simple moving average, which applies an equal weight to all observations in the period.

### RSI

The relative strength index (RSI) is calculated using the following formula:

```
RSI = 100 - 100 / (1 + RS)

Where RS = Average gain of up periods during the specified time frame / Average loss of down periods during the specified time frame.
```

Traditional interpretation and usage of the RSI is that RSI values of 70 or above indicate that a security is becoming overbought or overvalued, and therefore may be primed for a trend reversal or corrective pullback in price. On the other side of RSI values, an RSI reading of 30 or below is commonly interpreted as indicating an oversold or undervalued condition that may signal a trend change or corrective price reversal to the upside.

### MACD

The moving average convergence divergence (MACD) is a technical momentum indicator, calculated for use with a variety of exponential moving averages (EMAs) and used to assess the power of price movement in a market. Putting together the MACD requires simply doing all of the following EMA calculations for any given market instrument (a stock, future, currency pair, or market index):
+ Calculate a 12-period EMA of price for the chosen time period.
+ Calculate a 26-period EMA of price for the chosen time period.
+ Subtract the 26-period EMA from the 12-period EMA.
+ Calculate a nine-period EMA of the result obtained from step 3.

The MACD also has a zero line to indicate positive and negative values. The MACD has a positive value whenever the 12-period EMA is above the 26-period EMA and a negative value when the 12-period EMA is below the 26-period EMA.

Below is a Python snippet that plots SMA, RSI, and MACD on top of the raw price and volume of the Microsoft stock. 

Before we proceed, 

```
pip install mpl_finance
```

### Sample code

```python
# THIS VERSION IS FOR PYTHON 2 #
import urllib2
import time
import datetime
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import matplotlib.dates as mdates
from mpl_finance import candlestick_ohlc
import matplotlib
import pylab

matplotlib.rcParams.update({'font.size': 9})


def rsiFunc(prices, n=14):
    deltas = np.diff(prices)
    seed = deltas[:n + 1]
    up = seed[seed >= 0].sum() / n
    down = -seed[seed < 0].sum() / n
    rs = up / down
    rsi = np.zeros_like(prices)
    rsi[:n] = 100. - 100. / (1. + rs)

    for i in range(n, len(prices)):
        delta = deltas[i - 1]  # cause the diff is 1 shorter

        if delta > 0:
            upval = delta
            downval = 0.
        else:
            upval = 0.
            downval = -delta

        up = (up * (n - 1) + upval) / n
        down = (down * (n - 1) + downval) / n

        rs = up / down
        rsi[i] = 100. - 100. / (1. + rs)

    return rsi


def movingaverage(values, window):
    weigths = np.repeat(1.0, window) / window
    smas = np.convolve(values, weigths, 'valid')
    return smas  # as a numpy array


def ExpMovingAverage(values, window):
    weights = np.exp(np.linspace(-1., 0., window))
    weights /= weights.sum()
    a = np.convolve(values, weights, mode='full')[:len(values)]
    a[:window] = a[window]
    return a


def computeMACD(x, slow=26, fast=12):
    """
    compute the MACD (Moving Average Convergence/Divergence) using a fast and slow exponential moving avg'
    return value is emaslow, emafast, macd which are len(x) arrays
    """
    emaslow = ExpMovingAverage(x, slow)
    emafast = ExpMovingAverage(x, fast)
    return emaslow, emafast, emafast - emaslow


def graphData(stock, MA1, MA2):
    '''
        Use this to dynamically pull a stock:
    '''
    try:
        print 'Currently Pulling', stock
        print str(datetime.datetime.fromtimestamp(int(time.time())).strftime('%Y-%m-%d %H:%M:%S'))
        urlToVisit = 'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=MSFT&outputsize=full&apikey=Q7B8REQMCEOPI1CZ&datatype=csv'
        stockFile = []
        try:
            sourceCode = urllib2.urlopen(urlToVisit).read()
            splitSource = sourceCode.split('\n')
            for eachLine in splitSource[1:800]:
                splitLine = eachLine.split(',')
                if len(splitLine) == 6:
                    if 'values' not in eachLine:
                        stockFile.append(eachLine)
        except Exception, e:
            print str(e), 'failed to organize pulled data.'
    except Exception, e:
        print str(e), 'failed to pull pricing data'
    try:
        date, openp, highp, lowp, closep, volume = np.loadtxt(stockFile, delimiter=',', unpack=True,
                                                              converters={0: mdates.strpdate2num('%Y-%m-%d')})
        x = 0
        y = len(date)
        newAr = []
        while x < y:
            appendLine = date[x], openp[x], closep[x], highp[x], lowp[x], volume[x]
            newAr.append(appendLine)
            x += 1

        Av1 = movingaverage(closep, MA1)
        Av2 = movingaverage(closep, MA2)

        SP = len(date[MA2 - 1:])

        fig = plt.figure(facecolor='#07000d')

        ax1 = plt.subplot2grid((6, 4), (1, 0), rowspan=4, colspan=4, facecolor='#07000d')
        candlestick_ohlc(ax1, newAr[-SP:], width=.6, colorup='#53c156', colordown='#ff1717')

        Label1 = str(MA1) + ' SMA'
        Label2 = str(MA2) + ' SMA'

        ax1.plot(date[-SP:], Av1[-SP:], '#e1edf9', label=Label1, linewidth=1.5)
        ax1.plot(date[-SP:], Av2[-SP:], '#4ee6fd', label=Label2, linewidth=1.5)

        ax1.grid(True, color='w')
        ax1.xaxis.set_major_locator(mticker.MaxNLocator(10))
        ax1.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
        ax1.yaxis.label.set_color("w")
        ax1.spines['bottom'].set_color("#5998ff")
        ax1.spines['top'].set_color("#5998ff")
        ax1.spines['left'].set_color("#5998ff")
        ax1.spines['right'].set_color("#5998ff")
        ax1.tick_params(axis='y', colors='w')
        plt.gca().yaxis.set_major_locator(mticker.MaxNLocator(prune='upper'))
        ax1.tick_params(axis='x', colors='w')
        plt.ylabel('Stock price and Volume')

        maLeg = plt.legend(loc=9, ncol=2, prop={'size': 7},
                           fancybox=True, borderaxespad=0.)
        maLeg.get_frame().set_alpha(0.4)
        textEd = pylab.gca().get_legend().get_texts()
        pylab.setp(textEd[0:5], color='w')

        volumeMin = 0

        ax0 = plt.subplot2grid((6, 4), (0, 0), sharex=ax1, rowspan=1, colspan=4, facecolor='#07000d')
        rsi = rsiFunc(closep)
        rsiCol = '#c1f9f7'
        posCol = '#386d13'
        negCol = '#8f2020'

        ax0.plot(date[-SP:], rsi[-SP:], rsiCol, linewidth=1.5)
        ax0.axhline(70, color=negCol)
        ax0.axhline(30, color=posCol)
        ax0.fill_between(date[-SP:], rsi[-SP:], 70, where=(rsi[-SP:] >= 70), facecolor=negCol, edgecolor=negCol,
                         alpha=0.5)
        ax0.fill_between(date[-SP:], rsi[-SP:], 30, where=(rsi[-SP:] <= 30), facecolor=posCol, edgecolor=posCol,
                         alpha=0.5)
        ax0.set_yticks([30, 70])
        ax0.yaxis.label.set_color("w")
        ax0.spines['bottom'].set_color("#5998ff")
        ax0.spines['top'].set_color("#5998ff")
        ax0.spines['left'].set_color("#5998ff")
        ax0.spines['right'].set_color("#5998ff")
        ax0.tick_params(axis='y', colors='w')
        ax0.tick_params(axis='x', colors='w')
        plt.ylabel('RSI')

        ax1v = ax1.twinx()
        ax1v.fill_between(date[-SP:], volumeMin, volume[-SP:], facecolor='#00ffe8', alpha=.4)
        ax1v.axes.yaxis.set_ticklabels([])
        ax1v.grid(False)
        ###Edit this to 3, so it's a bit larger
        ax1v.set_ylim(0, 3 * volume.max())
        ax1v.spines['bottom'].set_color("#5998ff")
        ax1v.spines['top'].set_color("#5998ff")
        ax1v.spines['left'].set_color("#5998ff")
        ax1v.spines['right'].set_color("#5998ff")
        ax1v.tick_params(axis='x', colors='w')
        ax1v.tick_params(axis='y', colors='w')
        ax2 = plt.subplot2grid((6, 4), (5, 0), sharex=ax1, rowspan=1, colspan=4, facecolor='#07000d')
        fillcolor = '#00ffe8'
        nslow = 26
        nfast = 12
        nema = 9
        emaslow, emafast, macd = computeMACD(closep)
        ema9 = ExpMovingAverage(macd, nema)
        ax2.plot(date[-SP:], macd[-SP:], color='#4ee6fd', lw=2)
        ax2.plot(date[-SP:], ema9[-SP:], color='#e1edf9', lw=1)
        ax2.fill_between(date[-SP:], macd[-SP:] - ema9[-SP:], 0, alpha=0.5, facecolor=fillcolor, edgecolor=fillcolor)

        plt.gca().yaxis.set_major_locator(mticker.MaxNLocator(prune='upper'))
        ax2.spines['bottom'].set_color("#5998ff")
        ax2.spines['top'].set_color("#5998ff")
        ax2.spines['left'].set_color("#5998ff")
        ax2.spines['right'].set_color("#5998ff")
        ax2.tick_params(axis='x', colors='w')
        ax2.tick_params(axis='y', colors='w')
        plt.ylabel('MACD', color='w')
        ax2.yaxis.set_major_locator(mticker.MaxNLocator(nbins=5, prune='upper'))
        for label in ax2.xaxis.get_ticklabels():
            label.set_rotation(45)

        plt.suptitle(stock.upper(), color='w')

        plt.setp(ax0.get_xticklabels(), visible=False)
        plt.setp(ax1.get_xticklabels(), visible=False)



        plt.subplots_adjust(left=.09, bottom=.14, right=.94, top=.95, wspace=.20, hspace=0)
        plt.show()
        fig.savefig('example.png', facecolor=fig.get_facecolor())

    except Exception, e:
        print 'main loop', str(e)



graphData('MSFT', 10, 50)
```
### Output graph

![alt text](https://i.imgur.com/Ru0vlNg.png "TIs")

That's it! You are now well equipped to: 
+ develop stock charting and visualization features for your own iOS/Android/Web/IoT (you name it!) applications and dashboards
+ Formulate trading strategies and test your hypothesis against historical price/volume movements 





