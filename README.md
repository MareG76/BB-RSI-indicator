### Indicator
```
//@version=5
indicator('BB + RSI indicator', shorttitle='BB+RSI', overlay=true)

//RSI
RSIsrc = input(close, title='RSI-Source')
len = input.int(14, minval=1, title='RSI-length')
up = ta.rma(math.max(ta.change(RSIsrc), 0), len)
down = ta.rma(-math.min(ta.change(RSIsrc), 0), len)
rsi = down == 0 ? 100 : up == 0 ? 0 : 100 - 100 / (1 + up / down)
overSold = input(30, title='RSI-Over Sold')
overBought = input(70, title='RSI-Over Bought')

//BB
BBsrc = input(close, title='BB-Source')
BBlength = input.int(20, minval=1, title='BB-length')
low_bb = input(0, title='Low-BB')
high_bb = input(0, title='High-BB')
mult = input.float(2.0, minval=0.001, maxval=50, title='Stdev mult')
BBbasis = ta.sma(close, BBlength)
BBdev = mult * ta.stdev(close, BBlength)
BBupper = BBbasis + BBdev
BBupper_high_bb = BBupper - (BBupper - BBbasis) * (high_bb / 50)
BBlower = BBbasis - BBdev
BBlower_low_bb = BBlower + (BBbasis - BBlower) * (low_bb / 50)

plot(BBbasis, color=color.new(color.yellow, 0), title='Basis')
p1 = plot(BBupper, color=color.new(color.blue, 0), title='Upper bound')
p1bb = plot(BBupper_high_bb, color=color.new(color.red, 0), title='High-BB')
p2 = plot(BBlower, color=color.new(color.blue, 0), title='Lower bound')
p2bb = plot(BBlower_low_bb, color=color.new(color.green, 0), title='Low-BB')
fill(p1, p2, color=color.new(color.navy, 90))
fill(p1, p1bb, color=color.new(color.red, 90))
fill(p2, p2bb, color=color.new(color.green, 90))

//Indikator
showIndicator = rsi > overBought and close > open and close[1] > open[1] or rsi < overSold and close < open and close[1] < open[1]
indicator_1 = rsi > overBought and close > BBupper_high_bb and showIndicator ? -1 : rsi < overSold and close < BBlower_low_bb and showIndicator ? 1 : 0

//Boja indikatora
bgcolor(indicator_1 == 0 ? na : color.new(color.maroon, 70))

//Strelica
plotarrow(indicator_1, colorup=color.new(color.green, 0), colordown=color.new(color.red, 0), minheight=10, maxheight=30)


// Alarm
alertcondition(indicator_1 != 0, title='BB & RSI signal', message='BB&RSI crossed')

//buy
alertcondition(indicator_1 == 1, title='Buy', message='BUY_BINANCE_USDT-ETH')

//sell
alertcondition(indicator_1 == -1, title='Sell', message='SELL_BINANCE_USDT-ETH')
```
### strategy
```
//@version=4
strategy("BB + RSI test", shorttitle="BB+RSI_T", overlay=true, pyramiding=999)
take_profit = input(3, title='Gain %', step=0.1, minval=0) / 100
//RSI
RSIsrc = input(close, title="RSI-Source")
len = input(14, minval=1, title="RSI-length")
up = rma(max(change(RSIsrc), 0), len)
down = rma(-min(change(RSIsrc), 0), len)
rsi = down == 0 ? 100 : up == 0 ? 0 : 100 - 100 / (1 + up / down)
overSold = input(30, title="RSI-Over Sold")
overBought = input(70, title="RSI-Over Bought")

//BB
BBsrc = input(close, title="BB-Source")
BBlength = input(20, minval=1, title="BB-length")
low_bb = input(0, title="Low-BB")
high_bb = input(0, title="High-BB")
mult = input(2.0, minval=0.001, maxval=50, title="Stdev mult")
BBbasis = sma(close, BBlength)
BBdev = mult * stdev(close, BBlength)
BBupper = BBbasis + BBdev
BBupper_high_bb = BBupper - (BBupper - BBbasis) * (high_bb / 50)
BBlower = BBbasis - BBdev
BBlower_low_bb = BBlower + (BBbasis - BBlower) * (low_bb / 50)

// === INPUT BACKTEST RANGE ===
FromMonth = input(defval=1, title="From Month", minval=1, maxval=12)
FromDay = input(defval=1, title="From Day", minval=1, maxval=31)
FromYear = input(defval=2019, title="From Year", minval=2013)
ToMonth = input(defval=1, title="To Month", minval=1, maxval=12)
ToDay = input(defval=1, title="To Day", minval=1, maxval=31)
ToYear = input(defval=9999, title="To Year", minval=2014)

start = timestamp(FromYear, FromMonth, FromDay, 00, 00)  // backtest start window
finish = timestamp(ToYear, ToMonth, ToDay, 23, 59)  // backtest finish window
window() =>  // create function "within window of time"
    time >= start and time <= finish ? true : false

plot(BBbasis, color=color.yellow, title="Basis")
p1 = plot(BBupper, color=color.blue, title="Upper bound")
p1bb = plot(BBupper_high_bb, color=color.red, title="High-BB")
p2 = plot(BBlower, color=color.blue, title="Lower bound")
p2bb = plot(BBlower_low_bb, color=color.green, title="Low-BB")
fill(p1, p2, color=color.navy)
fill(p1, p1bb, color=color.red)
fill(p2, p2bb, color=color.green)

//Indikator
showIndicator = rsi > overBought and close > open and close[1] > open[1] or 
   rsi < overSold and close < open and close[1] < open[1]
indicator = rsi > overBought and close > BBupper_high_bb and showIndicator ? -1 : 
   rsi < overSold and close < BBlower_low_bb and showIndicator ? 1 : 0

take_profit_level = strategy.position_avg_price * (1 + take_profit)


// Alarm
//alertcondition(indicator != 0, title = "BB & RSI signal", message = "BB&RSI crossed")

//buy
//alertcondition(indicator == 1, title = "Buy", message = "BUY_BINANCE_USDT-ETH")

//sell
//alertcondition(indicator == -1, title = "Sell", message = "SELL_BINANCE_USDT-ETH")

buy = indicator == 1 and window()
if buy
    strategy.entry("Buy", strategy.long)

// Sell
sell = indicator == -1 and window()
if sell
    strategy.exit("Sell", limit=take_profit_level)

plot(strategy.position_avg_price, style=plot.style_linebr, color=color.red, title='Average Price')
```
