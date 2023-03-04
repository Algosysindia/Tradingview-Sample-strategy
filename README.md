// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © TD_Algosys

//@version=5
strategy(title="2 EMA Crossover", overlay=true)

ema1 = ta.ema(close, 10)
ema2 = ta.ema(close, 20)

// Buy signal
buySignal = ta.crossover(ema1, ema2)

// Sell signal
sellSignal = ta.crossunder(ema1, ema2)

// Plot EMAs
plot(ema1, color=color.green, title="EMA 1")
plot(ema2, color=color.red, title="EMA 2")

// Plot buy/sell signals
plotshape(buySignal, style=shape.triangleup, color=color.green, location=location.belowbar, title="Buy Signal")
plotshape(sellSignal, style=shape.triangledown, color=color.red, location=location.abovebar, title="Sell Signal")

////// ALGOSYS API INTEGRATION //////////
exitType = input.string("Intraday", "Intraday or Positional?", ["Intraday", "Positional"], group="Strategy Type", inline="Exit Type")
s = input.session(title='INTRA DAY TRADE SESSION', defval='0916-1500')
st = time(timeframe.period, s)
e = input.session(title='END SESSION', defval='1500-1520')
et = time(timeframe.period, e)

//adding stoploss and target option 
type = input.string(defval="Percent",title ="Point or Percent",options=["Point","Percent"])
ut=input(defval=false,title="ENABLE TARGET") 
us=input(defval=false,title="ENABLE STOPLOSS") 
tar=input(defval=10.0,title="TARGET") 
stop=input(defval=7.0,title="STOPLOSS")
qty=input(defval=1,title="QUANTITY")
tarp=tar 
slp=stop 

BuyPrice = ta.valuewhen(buySignal, close, 0)
ShortPrice = ta.valuewhen(sellSignal, close, 0)

float bstop=na 
float sstop=na
float btar=na
float star=na


if type=="Point" 
    bstop :=  stop/syminfo.mintick  
    sstop :=  stop/syminfo.mintick 
    btar:=tar/syminfo.mintick 
    star := tar/syminfo.mintick 

else
    bstop := (BuyPrice-(BuyPrice * (1-(stop/100))))/syminfo.mintick  
    sstop := ((ShortPrice * (1+(stop/100))) - ShortPrice )/syminfo.mintick 
    btar  := ((BuyPrice * (1+(tar/100))) -BuyPrice)  /syminfo.mintick       //tar/syminfo.mintick 
    star  := (ShortPrice - (ShortPrice * (1-(tar/100)))) /syminfo.mintick      //tar/syminfo.mintick


//setting automated alerts required by ALGOSYS
stag = input(title='ALGOSYS TAG', defval='')

sxle = 'inSymbol=' + syminfo.ticker + '&price=' + str.tostring(BuyPrice) + '&stopLoss='+ str.tostring(bstop) +'&takeProfit='+ str.tostring(btar) +'&stopPrice='+ str.tostring(bstop) +'&type=SXLE' + '&strat_code=' + stag + '&quantityInAlert='+ str.tostring(qty)

lxse = 'inSymbol=' + syminfo.ticker + '&price=' + str.tostring(ShortPrice) + '&stopLoss='+ str.tostring(sstop) +'&takeProfit='+ str.tostring(star) +'&stopPrice='+ str.tostring(sstop) +'&type=LXSE' + '&strat_code=' + stag + '&quantityInAlert='+ str.tostring(qty)

le = 'inSymbol=' + syminfo.ticker + '&price=' + str.tostring(BuyPrice) + '&stopLoss='+ str.tostring(bstop*syminfo.mintick ) +'&takeProfit='+ str.tostring(btar*syminfo.mintick ) +'&stopPrice='+ str.tostring(bstop) +'&type=LE' + '&strat_code=' + stag + '&quantityInAlert='+ str.tostring(qty)

lx = 'inSymbol=' + syminfo.ticker + '&price=' + str.tostring(close) + '&stopLoss='+ str.tostring(bstop*syminfo.mintick ) +'&takeProfit='+ str.tostring(btar*syminfo.mintick ) +'&stopPrice=0&type=LX' + '&strat_code=' + stag

se = 'inSymbol=' + syminfo.ticker + '&price=' + str.tostring(ShortPrice) + '&stopLoss='+ str.tostring(sstop*syminfo.mintick ) +'&takeProfit='+ str.tostring(star*syminfo.mintick ) +'&stopPrice='+ str.tostring(sstop) +'&type=SE' + '&strat_code=' + stag + '&quantityInAlert='+ str.tostring(qty)

sx = 'inSymbol=' + syminfo.ticker + '&price=' + str.tostring(close) + '&stopLoss='+ str.tostring(sstop*syminfo.mintick ) +'&takeProfit='+ str.tostring(star*syminfo.mintick ) +'&stopPrice=0&type=SX' + '&strat_code=' + stag

if(buySignal and st and strategy.position_size==0 )
    strategy.entry("BUY",strategy.long,comment=le)

if(sellSignal and st and strategy.position_size==0)
    strategy.entry("SELL",strategy.short,comment=se)

if(buySignal and st and strategy.position_size!=0)
    strategy.entry("BUY",strategy.long,comment=sxle)
    
if(sellSignal and st and strategy.position_size!=0)
    strategy.entry("SELL",strategy.short,comment=lxse)


if(ut==true and us==false) 
    strategy.exit(id="LX",from_entry="BUY",profit=btar,comment=lx) 
    strategy.exit(id="SX",from_entry="SELL",profit=star,comment=sx)

if(us==true and ut==false) 
    strategy.exit(id="LX",from_entry="BUY",loss=bstop,comment=lx) 
    strategy.exit(id="SX",from_entry="SELL",loss=sstop,comment=sx) 


if(ut==true and us==true) 
    strategy.exit(id="LX",from_entry="BUY",profit=btar,loss=bstop,comment=lx) 
    strategy.exit(id="SX",from_entry="SELL",profit=star,loss=sstop,comment=sx) 


if (exitType=="Intraday")  
    strategy.close(id="BUY",when=et,comment=lx)
    strategy.close(id="SELL",when=et,comment=sx)



// Webhook URL //
// http://trade.algosys.co.in/tdalgoapi/api/ProcessTask/PostStockRateWithParams
