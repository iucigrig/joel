//@version=5
indicator(title="Joel on Crypto - MACD Scalping", shorttitle="Joel on Crypto - MACD Scalping", timeframe="", timeframe_gaps=true)
// Getting inputs
slow_length1 = input(title="EMA Trend 1", defval=50)
slow_length2 = input(title="EMA Trend 2 ", defval=200)
fast_length = input(title="MACD Fast Length", defval=12)
slow_length = input(title="MACD Slow Length", defval=26)
signal_length = input.int(title="MACD Signal Smoothing",  minval = 1, maxval = 50, defval = 9)
src = input(title="MACD Source", defval=close)

i_switch = input.string(title="Tick Highlight", defval="Moving average" ,options=["Moving average","Fixed value" ])
i_switch2 = input.string(title="Tick Source", defval="Highest bar" ,options=["Highest bar","Average","Last bar"])

signal_lengthup = input.int(title="Upticks Avg. Length",  minval = 1, maxval = 5000, defval = 72)
signal_lengthdown = input.int(title="Downticks Avg. Length",  minval = 1, maxval = 5000, defval = 72)

signal_lengthMA = input.int(title="Ticks Avg. Multiplier",  minval = 1, maxval = 5000, defval = 2)

sma_source = "EMA"
sma_signal = "EMA"
// Plot colors

col_grow_above = #26A69A
col_fall_above =#B2DFDB
col_grow_below = #FFCDD2
col_fall_below = #FF5252
// Calculating

fast_ma = sma_source == "SMA" ? ta.sma(src, fast_length) : ta.ema(src, fast_length)
slow_ma = sma_source == "SMA" ? ta.sma(src, slow_length) : ta.ema(src, slow_length)

time_macd=timeframe.period=="1"?"1": timeframe.period=="3"?"1": timeframe.period=="5"?"1": timeframe.period=="15"?"3":timeframe.period=="30"?"5":timeframe.period=="60"?"15":timeframe.period=="120"?"30":timeframe.period=="240"?"60":timeframe.period=="D"?"240":timeframe.period=="W"?"D":timeframe.period=="M"?"W":timeframe.period=="12M"?"M":timeframe.period



macd = fast_ma - slow_ma
macd1=request.security(syminfo.tickerid, time_macd, macd)
signal = sma_signal == "SMA" ? ta.sma(macd1, signal_length) : ta.ema(macd1, signal_length)

ema50=ta.ema(close,slow_length1)
ema200=ta.ema(close ,slow_length2)



hist = request.security(syminfo.tickerid, time_macd, macd1 - signal)


f() => [hist[4],hist[3],hist[2],hist[1], hist]
ss=request.security(syminfo.tickerid, time_macd, hist, barmerge.gaps_on,barmerge.lookahead_off)



[ss5,ss4,ss3,ss2,ss1]=request.security(syminfo.tickerid, time_macd, f(), barmerge.gaps_on,barmerge.lookahead_off)



a = array.from(ss5,ss4,ss3,ss2,ss1)

s3=i_switch2=="Highest bar"?(ss>0? array.max(a, 0) : array.min(a, 0)):i_switch2=="Average"?array.avg(a):i_switch2=="Last bar"?ss1:0

saa=timeframe.period == '1'? ss:s3

saa2=timeframe.period == '1'? ss:s3*signal_lengthMA


colorss=(s3>=0 ? (s3[1] < s3 ? col_grow_above : col_fall_above) : (s3[1] < s3 ? col_grow_below : col_fall_below))


saadown = saa2
saaup = saa2

saadown:=saa>=0? saa2:saadown[1]

saaup:=saa<0? saa2:saaup[1]



verr=ta.ema(saadown,signal_lengthup)
dowww=ta.ema(saaup,signal_lengthdown)

ss22=plot(verr, title="Avg. Cloud Upper 1", color=color.new(color.white, 100))
ss33=plot(dowww, title="Avg. Cloud Lower 1", color=color.new(color.white, 100))

fill(ss22, ss33, color.new(color.white, 93), title="Avg. Cloud Background")

fixeduptick = input(title="Fixed Uptick Value", defval=30)
fixeddowntick = input(title="Fixed Downtick Value", defval=-30)
minl = i_switch=="Fixed value"? fixeduptick  :  verr
maxl = i_switch=="Fixed value"? fixeddowntick : dowww 

plot(minl, title="Avg. Cloud Upper 2", color=color.new(color.white, 81))
plot(maxl, title="Avg. Cloud Lower 2", color=color.new(color.white, 81))


colors2= s3<=minl and s3>=maxl ? #2a2e39 : colorss

coro2=s3>0? ema50>ema200 ? #2a2e39 :  colors2 : ema50<ema200 ? #2a2e39: colors2
plot(saa, title="Histogram", style=plot.style_columns, color=coro2)



alertcondition(#2a2e39 != coro2 , title='MACD Tick Alert', message='Joel on Crypto - MACD Tick Alert')

