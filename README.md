# options_trading

Delta Neutral Options Trading Management System


This is my first attempt to automate Options Trading. It is written in Python using Object Oriented Paradigm. The data is pulled from yfinance which is a 3rd party library that grabs data from Yahoo Finance. The data is stored in a PostgreSQL database every week. Yahoo Finance provides free options data but it has 15 minutes delays, so it can not be used for live trading. The trades are stored here but they are not actually executed. For that, one can use Interactive Brokers, TD Ameritrade etc. There are only 3 options strategies implemented. It's not backtested and I don't recommend using these trading strategies.

## What is options?
https://www.investopedia.com/articles/active-trading/040915/guide-option-trading-strategies-beginners.asp

### What does it do?

1. When options market is open (Mon-Fri 9:30AM EST to 4:00PM EST), it grabs options chain for preselected underlyings once per hour. It computes option greeks and then checks the portfolio.
   
2. It checks current trades and determines whether there are any trades that need to be closed or managed. If the profit is 50% of the initial credit, it closes the trade. If trade is at a loss and needs to be managed, it manages the trade according to the rules described as below.
   
4. After checking the portfolio, the next step is to determine whether a new trade should be instantiated. If portfolio SPY-weighted delta is far enough from zero and portfolio has enough buying power, it neutralizes the portfolio by shorting options. If portfolio delta is negative, it shorts Puts. If portfolio delta is positive, it shorts Calls. Lastly, if portfolio delta is close to zero it shorts Strangles. This check happens 7 times a day most of the days.
   
5. It saves hourly options data to csv files during the week.
   
6. On Saturdays, it grabs minute data for stock prices for that particular week for selected underlyings. After that, it stores options chains along with stock prices data to PostgreSQL database.

7. It computes some statistics about the portfolio. e.g., P/L, losing trades, winning trades etc.

#### How does it manage a losing trade?
Short Put Management:
    a. If the strike price is hit and days to expiration is greater than 21 days, then just short a .30 Delta Call
    b. If the strike price is hit and days to expiration is less than 21 days, close the put and short 45 days .30 Delta Strangle.

Short Call Management:
    a. If the strike price is hit and days to expiration is greater than 21 days, then just short a .30 Delta Put
    b. If the strike price is hit and days to expiration is less than 21 days, close the call and short 45 days .30 Delta Strangle.

Short Strangle Management:
    a. If the Strangle Delta is too big (like .40 Delta) and days to expiration is greater than 21 days, close the trade and open new .30 Delta Strangle for the           same expiration date.
    b. If the Strangle Delta is too big and days to expiration is less than 21 days, close the trade and open new 45 days .30 Delta Strangle.
    
For all these three strategies, tally up credits and debits and close the trade as soon as it breaks even.

##### Some 3rd Party Libraries Used:
1. pytest for Unit testing
2. yfinance for options and stock data
3. numpy, pandas to manipulate for computations and data manipulations
4. py_vollib_vectorized for computing options greeks
