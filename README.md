// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © voided

//@version=5
strategy("ry_strat_", overlay=true)

// roll yield

vix = request.security("VIX", "D", close)
vx1 = request.security("VX1!", "D", close)
vx2 = request.security("VX2!", "D", close)

one_day = 24 * 60 * 60 * 1000
month_start = time - dayofmonth * one_day

var int this_exp = na
for i = 0 to 8
    d = month_start + i * one_day
    if dayofweek(d) == 3
        this_exp := d + 14 * one_day
        break
        
var int next_exp = na
for i = 1 to 39
    d = time + i * one_day
    if month(d) != month and dayofweek(d) == 3
        next_exp := d + 14 * one_day
        break

var int period = na
var int dte = na
if time <= this_exp
    dte := (this_exp - time) / one_day
    period := (this_exp - this_exp[32]) / one_day
else
    dte := (next_exp - time) / one_day
    period := (next_exp - this_exp) / one_day
    
vx30 = (vx1 * dte / period)  + (vx2 * (1 - dte / period))
ry = (vx30 - vix) / vix

// strategy

qty         = input(title = "amount", defval = 100)
buy_thresh  = input(title = "buy", defval = 0)
sell_thresh = input(title = "sell", defval = 0)
length      = input(title = "length", defval = 111)

filter      = ta.sma(open, length)
buy_signal  = ta.crossover(ry, buy_thresh) and open > filter
sell_signal = ta.crossunder(ry, sell_thresh) or ta.crossunder(open , filter)


strategy.entry("long", strategy.long, qty, when = buy_signal)
strategy.entry("short", strategy.short, qty, when = sell_signal)
