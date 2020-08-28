
## Arbitrage Pricing Model


`Arbitrage Pricing Theorem` (APT) revisits the CAPM approach and extends it in a way to include various systematic risk factors, which is likely to affect the excess return of the related stock. So, the main goal of APT is to improve portfolio performance by better understanding portfolio creation and evaluation. 

Following variables are employed as the explanatory macroeconomic factors:

* VIX index (CBOE Volatility Index)
* Gold price (U.S. Dollars per Troy Ounce)
* Exchange rate (U.S. $ to  1 Euro)

We begin with data retrieving using `FRED` database by which VIX index, gold price, and exchange rate variables are gathered.

`
from fredapi import Fred
fred = Fred(api_key='78b14ec6ba46f484b94db43694468bb1')
`
{{execute}}

`fred.search('VIX').iloc[0]`{{execute}}

`fred.search('exchange').iloc[0]` {{execute}}

Let's restrict the time period to 2010/01/01-2020/01/01.


`
vix=fred.get_series('VIXCLS')
vix=vix['2010-01-01':'2020-01-10']
`
{{execute}}

`
gold=fred.get_series('GOLDAMGBD228NLBM')
gold=gold['2010-01-01':'2020-01-10']
`
{{execute}}

`
exc=fred.get_series('DEXUSEU')
exc=exc['2010-01-01':'2020-01-10']
`
{{execute}}

Now, we merge all these three variables using `concat` function and have a data frame named `factors`. Then logarithmic returns of these variables are computed.

`
factors=pd.DataFrame()
factors=pd.concat([vix,gold,exc], axis=1)
`
{{execute}}

`return_factors = np.log(factors / factors.shift(1)).dropna()`{{execute}}

Similar to CAPM analysis, `BAC`,`BK`,`BLK`,`^GSPC` are employed with `Adjusted Closing Price`.

`stocks = ['BAC','BK','BLK','^GSPC']`{{execute}}

`
start = datetime.datetime(2010,1,1)
end = datetime.datetime(2020,1,1)
`
{{execute}}

`daily_stock_prices = yf.download(stocks,start=start,end = end, interval='1d')['Adj Close']`{{execute}}

After retrieving the stocks from `yahoo finance`, we are ready to calculate the logarithmic returns as shown.

`return_stocks = np.log(daily_stock_prices / daily_stock_prices.shift(1)).dropna()`{{execute}}

Let's now combine all these variable into single data frame so that we are able to do all the analysis using onle one data frame.

`
df_apt=pd.DataFrame()
df_apt=pd.concat([return_stocks,return_factors], axis=1)
`
{{execute}}

Factor names appears as 0, 1, and 2. So, it is better to rename the factors name as `vix`, `gold`, `exc`.

`df_apt=df_apt.rename(columns={0: 'vix',1:'gold',2:'exc'})`{{execute}}


`df_apt.dropna(inplace=True)`{{execute}}
`df_apt.head()`{{execute}}

In the next step, we turn daily variables into monthly ones by taking average. To do that we use `resample` function first and,  to take the monthly average, we compute the mean.

`
df_apt=df_apt.resample('M').mean()
df_apt
`
{{execute}}

We are all set to run APT analysis. This time we will not separately run the APT analysis instead all three regression analysis will be conducted at once using `for loop`, which is useful when we have many regression to run. Meanwhile, risk free rate is assumed to be zero.

`import statsmodels.api as sm`{{execute}}

`
APT_result=[]
for i in range(3):
    df_apt.iloc[:,4:]= sm.add_constant(df_apt.iloc[:,4:], prepend=False)
    APT= sm.OLS(df_apt.iloc[:,i], df_apt.iloc[:,4:])
    APT_result.append(APT.fit())
`
{{execute}}

Output relates to `BAC`, `BK`, and `BLK`, respectively. APT analysis reveals that only `vix` variable are statistically significant in all three models. Besides, the estimated coefficient of `vix` is negative indicating that volatility badly affects the finance-related stock returns.

In addition to that, in the third regression, which is about `BLK` company, `exc` is also statistically significant with positive coefficient meaning that an increase in exchange rate triggers an increase in `BLK` stock price.

`APT_result[0].summary()`{{execute}}

`APT_result[1].summary()`{{execute}}

`APT_result[2].summary()` {{execute}}



















































