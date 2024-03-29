
# A Quantitative Research on Cross-Sectional Dispersion and Expected Returns
- Original Paper: *Verousis, Thanos and Voukelatos, Nikolaos, Cross-Sectional Dispersion and Expected Returns (December 12, 2015)*. Available at SSRN:(https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2734192))

- [Our Full Report](https://github.com/CoookieYou/Cross-Sectional-Dispersion-as-Risk-Factor/blob/main/csd_and_expected_returns.pdf)

## Introduction 

This paper is a replication of a research paper by Thanos Verousis and Nikolaos Voukelatos in 2016,  *Cross-Sectional Dispersion and Expected Returns.* The team followed the original paper’s concept with a wider timeline of market and stock data. The data analysis and calculation were executed in Python with Jupyter Notebook, along with packages including pandas, numpy, and statsmodels. The final result negligibly differs from the primitive paper, however the team determined that the factor is capable of explaining and predicting the market volatility and therefore, predicting the returns.


## Hypothesis Overview
1. CSD has negative correlation to individual stocks’ return. Stocks with low sensitivity to CSD generally have lower returns than the ones with higher sensitivity.
  a. Mimicking portfolio for CSD monthly (1-5, N-P)
  b. Monthly rolling regression on MKT, CSD
  3. Sort based on CSD beta
  
2. The risk premium of CSD is distinct from the premia generated by other systematic risk factors
  a. Risk adjusted returns, with MKT, SMB, HML, MOM
  b. Regression on monthly returns
  
3. After accounting for various stock characteristics, market conditions and industry groupings, the risk-premium of CSD is still statistically significant.
  a. Risk adjusted returns, after accounting for systematic factors, with size, momentum, std, skew, kurt, 
  b. Double sorted portfolios

4. After accounting for other measurements of aggregated risk, the CSD still has significant explanatory power over the cross-section of stocks’ returns.
  a. Mimicking portfolio FCSD
  b. Regression on double sorted portfolios, maybe add Liquidity factor
  c. Also do b. To VIX, Stock Variance SVAR, index of macro uncertainty, cross-sectional means of residual volatility

***Due to the lack of data for analysts’ prediction, we can not test the affect of FDISP factor


## Data Source

We provide the source of the data we used and the methods we used to manage them to provide direction for future research. The stock data were obtained from CRSP database. We obtained the Fama-French Three factors as well as Carhart momentum factors from the data library in Kenneth French's website, the VIX index from Bloomberg. The data of the index of the macroeconomic uncertainty was obtained from the website of Turan Bali. Here are the brief descriptive analysis about our main data.

<img src="https://user-images.githubusercontent.com/60916875/190038006-9247577d-f607-41d5-8072-592dd0ebed75.png" width = "750">

**Factors**

Fama-French three factors combined with Carhart momentum factors were seems as one of the standard source of systematic risk premia in many of the academic researches.

```{python echo = FALSE}
pd.read_csv("Factors.csv", header = 0, index_col = 0).div(100).plot(figsize = (16, 9),title = "Systematic Risk Factors")
```

**VIX**

VIX index is the weighted average implied volatility for at-the-money S&P500 index options. It usually serves as a good proxy for the short-term market uncertainty.

```{python echo = FALSE}
pd.read_excel("VIX_daily.xlsx", sheet_name = "Worksheet", header = 0, index_col = 0).PX_LAST.plot(figsize = (16, 9),title = "VIX Index")
```

**Index of Macroeconomic uncertainty**

This index is good measure for systematic risk in a macro level. It closely follows the fluctuation in business conditions and financial crisis period.

```{python echo = FALSE}
pd.read_csv("Economic Uncertainty Index.csv", header = 0, index_col = 0).plot(figsize = (16, 9))
```

### Data Management

The CRSP database we used contains all the stocks data since 1989 including fields like daily close price, daily trading volume, issue type of the asset etc. The original database has size of over 60 GB.In order to save our RAM and be able to extract required data efficiently, we read the original data by chunk using Python's pandas library, and stored the transformed data based on the fields in HDF format. Here are our main codes for storing the data.

```{python eval = FALSE}
import time

for date in ['2010-2019', '2000-2009', '1990-1999']:
  data = pd.read_csv(f"/content/drive/My Drive/FIN554/Data{date}.csv")
  data_new = data.loc[data.tic.isin(whole_list)]
  del data
  
  time.sleep(10)
  
  # Save the daily close price
  close_data = data_new.pivot_table(index = "datadate", columns = ["tic"], \
  values = ["prccd"], aggfunc = "mean")
  close_data.to_hdf(f"/content/drive/My Drive/FIN554/ClosePrice_2012.h5", \
  key = date, complevel = 9, complib = 'blosc')
  del close_data
  
  # Save the daily shares outstanding
  share_data = data_new.pivot_table(index = "datadate", columns = ["tic"], \
  values = ["cshoc"], aggfunc = "mean")
  share_data.to_hdf(f"/content/drive/My Drive/FIN554/ShareOutstanding_2012.h5", \
  key = date, complevel = 9, complib = 'blosc')
  del share_data
  
  # Save the price adjusting factor
  adj_data = data_new.pivot_table(index = "datadate", columns = ["tic"], \
  values = ["ajexdi"], aggfunc = "mean")
  adj_data.to_hdf(f"/content/drive/My Drive/FIN554/AdjustFactor_2012.h5", \
  key = date, complevel = 9, complib = 'blosc')
  del adj_data
  
  trfd_data = data_new.pivot_table(index = "datadate", columns = ["tic"], \
  values = ["trfd"], aggfunc = "mean")
  trfd_data.to_hdf(f"/content/drive/My Drive/FIN554/TrfdFactor_2012.h5", \
  key = date, complevel = 9, complib = 'blosc')
  del trfd_data
  
  # Delete original data frame and pause the program to save RAM
  del data_new

  time.sleep(10)
```

### Software Usage

We conduct all our data processing task and statistical analysis in Python. We mainly used Pandas and Numpy modules for data management. The statistical analysis were conducted using statsmodels module. Our data and test results visualization were applied with Matplotlib and Seaborn modules in Python as well as knitr library in R.

## Replication of Key Analytical Techniques

### Mimicing portfolio for CSD monthly

We conducted regression on stocks' return and $\Delta CSD$ as well as $MKT$ factor. Sort the $\beta \Delta CSD$ and form portfolios on different quintiles (1-5).

The performance of portfolios are as follows:

```{python}
### Portfolio Returns
sorting = pd.read_csv("Sorting_return.csv", header = 0, index_col = 0)
quintile = pd.read_csv("Quintile_return.csv", header = 0, index_col = 0)
```

```{python}
sorting.index = pd.DatetimeIndex(sorting.index)
sorting.add(1).cumprod().plot(figsize = (16,9), title = 'Sorting Portfolios')
```
<img src="https://user-images.githubusercontent.com/60916875/190038235-cf7a4d36-167c-4cb8-88fa-41690a486a5b.png" width = "750">

```{python}
quintile.index = pd.DatetimeIndex(quintile.index)
quintile.add(1).cumprod().plot(figsize = (16,9), title = 'Quintile Portfolios')
```
<img src="https://user-images.githubusercontent.com/60916875/190038254-f3e40803-7fbe-4960-9910-4b1fceab7c40.png" width = "750">


### Determine the risk premium of CSD

In order to show the relationship between dispersion loadings and mean returns is robust to other aggregate factors that have been commonly found to explain the cross-section of stock returns, such as market return MKT, the two additional [@Fama-French] factors SMB and HML, and the [@Carhart] momentum factor MOM. We first select entities that have survived from 1990 to 2019. The stock returns were adjusted by the adjusted factors in order to be used for time series analysis. We multiply each stocks’ daily return with their adjusted factor with the following code.

```{python, eval = FALSE}
price = pd.DataFrame(index = data_new.index)
 
for f in range(data_new.shape[1]):
  name = col_name[f]
  price[name] = data_new[name]*adj[name]
```

Then we only keep stocks that survive all the time. We drop stocks that contain N/A for more than 200 days.

Next, we generate the monthly return for each stock. 

```{python, eval = FALSE}
price_data['month'] = price_data['datadate'].astype(str).str[:6]
monthly_total = price_data.groupby('month')
first = price_data.groupby('month').first()
last = price_data.groupby('month').last()
month_return = last - first
month_return
month_return = month_return.drop(['datadate'], axis = 1)
month_return = month_return.reset_index()
month_return
```

Get monthly factors datas and combine with stock return.
```{python, eval = FALSE}
raw_data = month_return
raw_data['month'] = raw_data['month'].astype(int)
df = month_return.merge(factors, how = 'left', left_on = 'month', right_on='date')
```

Doing regression for each stock.

```{python, eval = FALSE}
import statsmodels.api as sm
X = df[my_list[900:904]]
X = sm.add_constant(X)
final = pd.DataFrame(index = ['const','Mkt-RF','SMB','HML','Mom'])
for g in range(1,899):
  Y = df[my_list[g]]
  result = sm.OLS(Y,X).fit()
  final[my_list[g]] = result.params.to_list()

name = list(final)
Diff = pd.DataFrame(index = df.index)
for g in range(final.shape[1]):
  con = final[name[g]][0]
  X1 = final[name[g]][1]
  X2 = final[name[g]][2]
  X3 = final[name[g]][3]
  X4 = final[name[g]][4] 
 
  predict = factors['Mkt-RF']*X1 + factors['SMB']*X2 + factors['HML']*X3+ \
  factors['Mom   ']*X4  + con
  Diff[name[g]] = predict
```

After estimate the risk adjusted return, we examine the impact of stock characteristics[@Chen]. By using The stock characteristics such as CSD and the standard deviation, skewness[@Harvey] and kurtosis of stock returns over the past six months.

```{python, eval = FALSE}
name = list(KKK)
res = pd.DataFrame( index = ['cons', 'CSD','adjR2'])
for i in range(1):
  sk1 = list()
  std1 = list()
  kuro1 = list()
  for j in range(300):
    skvalue = original[name[i]]
    skvalue = skvalue[(66+j):(7.2+j)]
    sk = skvalue.skew()
    std = skvalue.std()
    kuro = skvalue.kurtosis()
    sk1.append(sk)
    std1.append(std1)
    kuro1.append(kuro)
  
  X['sk'] = sk1
  X['std'] = std1
  X['kuro'] = kuro1
  Y = KKK[name[i]].to_list()
  result = sm.OLS(Y,X).fit()
  res[name[i]] = result.params.to_list()
```

### Test Risk Adjusted Returns

We run regression on the portfolio returns (1-5, N-P) with systematic factors, and use the significant of interseption terms to indentify the risk-adjusted returns of CSD.

```{python, eval = FALSE}
### Fetch the monthly returns for mimicing portfolios
def monthly_return(x):
  return x.add(1).cumprod().iloc[-1]-1

one_five = one_five.to_frame(name = '1-5')
N_P = N_P.to_frame(name = 'N-P')

one_five['month'] = one_five.index.strftime('%Y-%m')
N_P['month'] = N_P.index.strftime('%Y-%m')

return_1_5 = one_five.groupby(['month']).apply(lambda x: monthly_return(x))
return_N_P = N_P.groupby(['month']).apply(lambda x: monthly_return(x))
```

```{python, eval = FALSE}
return_1_5.index = return_1_5.index.str[:4] + return_1_5.index.str[-2:]
return_N_P.index = return_N_P.index.str[:4] + return_N_P.index.str[-2:]
```

```{python, eval = FALSE}
### montly factors data
factors_monthly = pd.read_csv('/content/drive/My Drive/FIN554/Factors.csv', \
index_col = 0)
factors_monthly.index = factors_monthly.index.astype(str)
factors_monthly = factors_monthly.loc[return_1_5.index]

### access returns
return_1_5 = return_1_5.sub(factors_monthly.RF, axis = 0)
return_N_P = return_N_P.sub(factors_monthly.RF, axis = 0)
```

```{python, eval = FALSE}
factors_monthly.rename(columns = {'Mom   ': 'Mom'}, inplace = True)
```

### Test the robustness of models

#### 1. formation window

Formation window equals to 3 months:

```{python, eval = FALSE}
### store the result in dictionary
def log_result_3month(result):
    beta_3month.append(result)

from multiprocessing import Pool
import os

pool = Pool(os.cpu_count())

# regression_data.fillna(method = 'ffill', inplace = True)
beta_3month = []

### 3month regression
months = regression_data.index.unique()
for i in tqdm(range(len(months)-2)):
  grouped_df = regression_data.loc[months[i]: months[i+2]]
  pool.apply_async(regression_stocks, args=(grouped_df,['Mkt-RF', 'CSD'], 'CSD'),\
  callback = log_result_3month)

pool.close()
pool.join()

beta_3month = pd.concat(beta_3month, axis=1).T

### save Beta of CSD factors
beta_3month.to_csv("/content/drive/My Drive/FIN554/CSD_Betas_3month.csv")

### read Beta of CSD factors
beta_3month = pd.read_csv("/content/drive/My Drive/FIN554/CSD_Betas_3month.csv",\
index_col = 0)
# beta_3month

### Reset the index of beta Dataframe into daily
beta_3month = beta_3month.shift()
beta_3month.index = pd.to_datetime(beta_3month.index)
beta_3month = beta_3month.resample('1d').mean()
beta_3month.pad(inplace = True)

### use data only after 1998-04-01
beta_3month = beta_3month.loc['19980401':]

### generate portfolio weights
No1, No2, No3, No4, No5, Neg, Pos, new_index = get_weights(beta_3month, return_daily.index)

N_P = long_short_return(Neg, Pos)
one_five = long_short_return(No1, No5)

X = sm.add_constant(factors.loc[N_P.index, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_3month = [one_five.mean(), sm.OLS(one_five, X).fit().params['const'], N_P.mean(),\
sm.OLS(N_P, X).fit().params['const']]
row_3month = [round(x, 5) for x in row_3month]
row_3month
```

Formation window equals to 6 months:

```{python, eval = FALSE}
### store the result in dictionary
def log_result_6month(result):
    beta_6month.append(result)

pool = Pool(os.cpu_count())

# regression_data.fillna(method = 'ffill', inplace = True)
beta_6month = []

### 6month regression
months = regression_data.index.unique()
for i in tqdm(range(len(months)-5)):
  grouped_df = regression_data.loc[months[i]: months[i+5]]
  pool.apply_async(regression_stocks, args=(grouped_df,['Mkt-RF', 'CSD'], 'CSD'),\
  callback = log_result_6month)

pool.close()
pool.join()

beta_6month = pd.concat(beta_6month, axis=1).T

### save Beta of CSD factors
beta_6month.to_csv("/content/drive/My Drive/FIN554/CSD_Betas_6month.csv")

### read Beta of CSD factors
beta_6month = pd.read_csv("/content/drive/My Drive/FIN554/CSD_Betas_6month.csv",\
index_col = 0)

### Reset the index of beta Dataframe into daily
beta_6month = beta_6month.shift()
beta_6month.index = pd.to_datetime(beta_6month.index)
beta_6month = beta_6month.resample('1d').mean()
beta_6month.pad(inplace = True)

### use data only after 1998-04-01
beta_6month = beta_6month.loc['19980401':]

### generate portfolio weights
No1, No2, No3, No4, No5, Neg, Pos, new_index = get_weights(beta_6month, return_daily.index)

N_P = long_short_return(Neg, Pos)
one_five = long_short_return(No1, No5)
X = sm.add_constant(factors.loc[N_P.index, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_6month = [one_five.mean(), sm.OLS(one_five, X).fit().params['const'], \
N_P.mean(), sm.OLS(N_P, X).fit().params['const']]
row_6month = [round(x, 5) for x in row_6month]
row_6month
```

#### 2. Sign of market

Negative MKT:

```{python, eval = FALSE}
mkt_neg = set(mkt.loc[mkt<0].index).intersection(set(one_five.index))

X = sm.add_constant(factors.loc[mkt_neg, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_mkt_neg = [one_five.loc[mkt_neg].mean(), sm.OLS(one_five.loc[mkt_neg], X)\
.fit().params['const'], N_P.loc[mkt_neg].mean(), sm.OLS(N_P.loc[mkt_neg], X).fit().params['const']]
row_mkt_neg = [round(x, 5) for x in row_mkt_neg]
row_mkt_neg
```

Positive MKT:

```{python, eval = FALSE}
mkt_pos = set(mkt.loc[mkt>0].index).intersection(set(one_five.index))

X = sm.add_constant(factors.loc[mkt_pos, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_mkt_pos = [one_five.loc[mkt_pos].mean(), sm.OLS(one_five.loc[mkt_pos], X)\
.fit().params['const'], N_P.loc[mkt_pos].mean(), sm.OLS(N_P.loc[mkt_pos], X).fit().params['const']]
row_mkt_pos = [round(x, 5) for x in row_mkt_pos]
row_mkt_pos
```

Negative CSD:

```{python, eval = FALSE}
csd_neg = set(csd.loc[csd<0].index).intersection(set(one_five.index))

X = sm.add_constant(factors.loc[csd_neg, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_csd_neg = [one_five.loc[csd_neg].mean(), sm.OLS(one_five.loc[csd_neg], X)\
.fit().params['const'], N_P.loc[csd_neg].mean(), sm.OLS(N_P.loc[csd_neg], X).fit().params['const']]
row_csd_neg = [round(x, 5) for x in row_csd_neg]
row_csd_neg
```

Positive CSD:

```{python, eval = FALSE}
csd_pos = set(csd.loc[csd>0].index).intersection(set(one_five.index))

X = sm.add_constant(factors.loc[csd_pos, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_csd_pos = [one_five.loc[csd_pos].mean(), sm.OLS(one_five.loc[csd_pos], X)\
.fit().params['const'], N_P.loc[csd_pos].mean(), sm.OLS(N_P.loc[csd_pos], X).fit().params['const']]
row_csd_pos = [round(x, 5) for x in row_csd_pos]
row_csd_pos
```

#### 3. CSD as AR(1) innovations

```{python, eval = FALSE}
from statsmodels.tsa.ar_model import AR

ar = AR(csd.dropna()).fit(1)
fit_data = ar.predict(start = csd.index[2], end = csd.index[-1])
csd_ar = csd.sub(fit_data)
csd_ar.name = 'CSD_AR'

mkt_copy = return_daily.factors['Mkt-RF'].copy(deep = True)
mkt_copy.index = pd.DatetimeIndex(mkt_copy.index.astype(str))

regression_ar = pd.concat([return_data, pd.concat([mkt_copy, csd_ar], axis = 1)],\
axis = 1, keys = ['stocks', 'factors'])

regression_ar.index = pd.DatetimeIndex(regression_ar.index).strftime('%Y-%m')
### store the result in dictionary
def log_result_ar(result):
    beta_ar.append(result)

pool = Pool(os.cpu_count())

# regression_data.fillna(method = 'ffill', inplace = True)
beta_ar = []

### regression
months = regression_ar.index.unique()
for i, grouped_df in regression_ar.groupby(level = 0):
  pool.apply_async(regression_stocks, args=(grouped_df,['Mkt-RF', 'CSD_AR'], 'CSD_AR'),\
  callback = log_result_ar)

pool.close()
pool.join()

beta_ar = pd.concat(beta_ar, axis=1).T

### save Beta of CSD factors
beta_ar.to_csv("/content/drive/My Drive/FIN554/CSD_Betas_AR1.csv") 

beta_ar = pd.read_csv("/content/drive/My Drive/FIN554/CSD_Betas_AR1.csv", index_col = 0)
### Reset the index of beta Dataframe into daily
beta_ar = beta_ar.shift()
beta_ar.index = pd.to_datetime(beta_ar.index)
beta_ar = beta_ar.resample('1d').mean()
beta_ar.pad(inplace = True)

### use data only after 1998-04-01
beta_ar = beta_ar.loc['19980401':]

### generate portfolio weights
No1, No2, No3, No4, No5, Neg, Pos, new_index = get_weights(beta_ar, return_daily.index)

N_P = long_short_return(Neg, Pos)
one_five = long_short_return(No1, No5)

X = sm.add_constant(factors.loc[N_P.index, ['Mkt-RF', 'SMB', 'HML', 'MOM']])

row_ar = [one_five.mean(), sm.OLS(one_five, X).fit().params['const'], N_P.mean(),\
sm.OLS(N_P, X).fit().params['const']]
row_ar = [round(x, 5) for x in row_ar]
row_ar
```

CSD's performance under different settings and conditions:

<img src="https://user-images.githubusercontent.com/60916875/190038325-0a7ff1d0-5ddb-47a1-8043-e8ac19c09bb9.png" width = "500">

### Price the Cross-Sectional Dispersion

We try to measure the price of CSD by showing that its price is immune of other cross-sectional risk measures[@Brennan]. Here we conduct the analysis of seven factors, $MKT$ market returns, $FVIX$ VIX volatilty index, $SVAR$ volatilty of returns of all stocks(arithmatic mean of squared returns), $UNC$ the index of uncertainty, $LIQ$ liquidity factor[@Pastor], and $IDVOL$ aggregated idiosyncratic risk, which were obtained as the residual of regression for each stocks' return on $MKT$, $SMB$, $HML$ and $MOM$ factors. Here for the convinence of calculation, we use the factor value as the mimic portfolio returns of that factor.

```{python, eval = FALSE}
# Read all additional data
### VIX
vix = pd.read_excel('/content/drive/My Drive/FIN554/VIX_daily.xlsx', sheet_name='Worksheet',\
index_col = 0).PX_LAST.diff()
vix.name = 'VIX'
### Macro Uncertainty Index
macro_risk = pd.read_csv('/content/drive/My Drive/FIN554/Economic Uncertainty Index.csv',\
index_col=0)

def format_index(data):
  if int(data.name[-2:])>=94:
    return '19' + data.name[-2:] + '-' +data.name[:3]
  else:
    return '20' + data.name[-2:] + '-' +data.name[:3]

macro_risk.index = pd.DatetimeIndex(macro_risk.apply(lambda x: format_index(x),\
axis =1).values)

### double sorting portfolios
def double_sorting(mkt_beta, target_beta, return_data, new_index, mv_df, factor_name):
  mkt_beta, target_beta = mkt_beta.align(target_beta, join = 'inner', axis = 0)
  portfolios = {}
  for i in range(5):
    No_mkt = ((mkt_beta.sub(mkt_beta.quantile(q=0.2*i, axis=1), axis = 0)>=0) & \
             (mkt_beta.sub(mkt_beta.quantile(q=0.2*(i+1), axis=1), axis = 0)<0)).astype(int)
    for j in tqdm(range(5), f'MKT quantile {i+1}'):
      No_target = ((target_beta.sub(target_beta.quantile(q=0.2*j, axis=1), axis = 0)>=0) & \
             (target_beta.sub(target_beta.quantile(q=0.2*(j+1), axis=1), axis = 0)<0) & No_mkt)\
             .astype(int)
      
      weights = No_target.loc[new_index].mul(mv_df.loc[new_index])
      weights = weights.div(weights.sum(axis = 1), axis = 0)
      returns = return_data.align(weights, join = 'right')[0].mul(weights).sum(axis=1)
      portfolios[f'factor_name{j+1}_mkt{i+1}'] = returns

  return pd.concat(portfolios, axis = 1)
```

MKT:

```{python, eval = FALSE}
mkt = return_daily.factors['Mkt-RF'].copy(deep = True)
mkt.index = pd.DatetimeIndex(mkt.index.astype(str))

### construct regression data
regression_mkt = pd.concat([return_data.align(mkt, join = 'right', axis =0)[0],\
                           mkt], axis = 1, keys = ['stocks', 'factors'])
regression_mkt.index = pd.DatetimeIndex(regression_mkt.index.astype(str)).strftime("%Y-%m")

### store the result in dictionary
def log_result_mkt(result):
    beta_mkt.append(result)

pool = Pool(os.cpu_count())

# regression_data.fillna(method = 'ffill', inplace = True)
beta_mkt = []

### monthly regression
for idx, grouped_df in regression_mkt.groupby(level = 0):
  pool.apply_async(regression_stocks, args=(grouped_df,['Mkt-RF'], 'Mkt-RF'),\
  callback = log_result_mkt)

pool.close()
pool.join()

beta_mkt = pd.concat(beta_mkt, axis=1).T
### save Beta of CSD factors
beta_mkt.to_csv("/content/drive/My Drive/FIN554/MKT_Betas.csv")

beta_mkt = pd.read_csv("/content/drive/My Drive/FIN554/MKT_Betas.csv", index_col = 0)

beta_mkt = beta_mkt.shift()
beta_mkt.index = pd.to_datetime(beta_mkt.index)
beta_mkt = beta_mkt.resample('1d').mean()
beta_mkt.pad(inplace = True)
```

FCSD:

```{python, eval = FALSE}
csd.index = pd.DatetimeIndex(csd.index.astype(str))

### Reset the index of beta Dataframe into daily
beta_df = beta_df.shift()
beta_df.index = pd.to_datetime(beta_df.index)
beta_df = beta_df.resample('1d').mean()
beta_df.pad(inplace = True)
### use data only after 1998-04-01
beta_df = beta_df.loc['19980401':]

fcsd = mimicing_portfolio(beta_df, return_data, mv_df, csd)
fcsd.name = 'FCSD'
fcsd.index = pd.DatetimeIndex(fcsd.index)
fcsd = fcsd.resample('1d').mean()
fcsd.pad(inplace = True)
```

VIX:

```{python, eval = FALSE}
vix = vix.sort_index()
vix = vix.loc['1996-01-01': '2012-12-31']
```

SVAR:

```{python, eval = FALSE}
svar = return_daily.stocks.pow(2).mean(axis = 1).diff()
svar.name = 'SVAR'
svar.index = pd.DatetimeIndex(svar.index.astype(str))
svar = svar.loc['1996-01-01': '2012-12-31']
svar = svar.pad()
```

UNC:

```{python, eval = FALSE}
macro_risk = macro_risk.resample('1D').pad()
macro_risk.columns = ['UNC']
unc = macro_risk.UNC
unc = unc.loc['1996-01-01': '2012-12-31']
```

LIQ:

```{python, eval = FALSE}
liq = pd.read_table('/content/drive/My Drive/FIN554/LIQ.txt', header = None, index_col = 0).iloc[:, 0]
liq.index = pd.to_datetime(liq.index.astype(str), format = "%Y%m")
liq.index.name = 'datatime'
liq.name = "LIQ"
liq = liq.resample('1D').mean()
liq.pad(inplace = True)
```

IDVOL:

```{python, eval = FALSE}
### Daily factor data
factors = pd.read_csv("/content/drive/My Drive/FIN554/Factors_daily.csv", header = 0,\
index_col = 0)
factors.index = pd.DatetimeIndex(factors.index.astype(str))

### concatenate all factors
factors = pd.concat([factors, liq], axis = 1, join = 'inner')
factors.drop(columns = ['RF'], inplace = True)

### construct regression data
regression_idvol = pd.concat([return_data.align(factors, join = 'right', axis =0)[0],\
                           factors], axis = 1, keys = ['stocks', 'factors'])
regression_idvol.index = pd.DatetimeIndex(regression_idvol.index.astype(str))\
.strftime("%Y-%m")
regression_idvol = regression_idvol.loc['1998-04-01':]

### store the result in dictionary
def log_result_idvol(result):
    beta_idvol.append(result)

pool = Pool(os.cpu_count())

# regression_data.fillna(method = 'ffill', inplace = True)
beta_idvol = []

### monthly regression
for idx, grouped_df in regression_idvol.groupby(level = 0):
  pool.apply_async(regression_stocks, args=(grouped_df,['Mkt-RF', 'SMB', 'HML',\
  'MOM', 'LIQ'],
  'residual'), callback = log_result_idvol)

pool.close()
pool.join()

beta_idvol = pd.concat(beta_idvol, axis=0)
### save Beta of CSD factors
beta_idvol.to_csv("/content/drive/My Drive/FIN554/IDVOL_Betas.csv")

beta_idvol = pd.read_csv("/content/drive/My Drive/FIN554/IDVOL_Betas.csv",\
index_col = 0)
beta_idvol.index = pd.DatetimeIndex(beta_idvol.index.astype(str))
beta_idvol = beta_idvol.groupby(level = 0).std().mean(axis = 1)
idvol = beta_idvol.resample('1D').mean()
idvol.pad(inplace = True)
idvol.name = 'IDVOL'
```

In order to show that the risk premium of CSD is distinct from the other factors, we conduct double sorting to obtain 25 quintile portfolios at the beginning of each month.

```{python, eval = FALSE}
double_sorted_csd = double_sorting(beta_mkt, beta_df, return_data, new_index, mv_df,\
'CSD')

### calculate the access returns
RF = return_daily.factors.RF.copy(deep = True)
RF.index = pd.DatetimeIndex(RF.index.astype(str))
RF = RF.align(double_sorted_csd, join = 'right', axis = 0)[0]
double_sorted_csd = double_sorted_csd.sub(RF, axis = 0)
```

Then we run the Fama-MacBeth two pass regression to test the significant of those factors' effect on CSD returns:

```{python, eval = FALSE}
### Fama-MacBeth two pass regression
def two_pass_regression(reg_data, portfolios):
  ### first pass: time-series regression
  time_series_beta = []
  for portfolio in portfolios.columns:
    Y = portfolios.loc[:, portfolio]
    X = reg_data.align(Y, join = 'right', axis = 0)[0]
    X = sm.add_constant(X)
    model = sm.OLS(Y, X).fit()
    time_series_beta.append(pd.Series(model.params, name = portfolio))

  time_series_beta = pd.concat(time_series_beta, axis = 1).T
  # print(portfolios.mean(), time_series_beta)
  ### second pass: cross-sectional regression
  X_new = time_series_beta
  Y_new = portfolios.mean()
  results = sm.OLS(Y_new, X_new).fit()
  return results
```

## Hypothesis Tests
### Test. CSD has negative correlation with expected returns

From the return of the sorting portfolo, we can conclude that CSD has a slight negative trend. However, from the t-test statistics of long-short portfolios 1-5 and N-P, the correlation is in fact not negative, and not significant. The difference might came from the different usage of data. In our analysis we use only the stocks that still are listing today, which intrduced survivor ship bias, and affect the significance. What's more, due to the missing data of market value in the late 90s, some of the stocks with significant negative exposure to CSD might not participate in the regression (we follow the rules which only use those stocks that has more than 15 observation in that month)

<img src="https://user-images.githubusercontent.com/60916875/190038364-bf1d6ead-9b5e-491e-ba78-ef9026f39019.png" width = "500">


We used the monthly returns of sorting portfolios 1-5 and N-P to do the regression on traditional Fama-French three factors $MKT$, $SMB$, $HML$ as well as Carhart's momentum factor $MOM$. From the regression results, we obtained a contradictory conclusion from Test1 which indicated that CSD has a significant negative risk-adjusted returns. The reason behind this might be that we use daily returns on Test1 but monthly returns on Test2, because monthly returns are more representative for factors like $SML$ and $HML$, which might reactly slowly to the market change, so we conclude that in long term investing, CSD can generate a significant risk-premium which are distinct from other established systematic risk premia.

<img src="https://user-images.githubusercontent.com/60916875/190038384-a60e3b83-6e65-4e37-b14d-10efa29b9365.png" width="350">

### Test. CSD can be price cross-sectionally

In order to prove that CSD can be priced cross-sectionally. We use Fama-MacBeth two-pass regression to test the returns of double-sorted portfolios (based on $MKT$ and $FCSD$) on different measurement of cross-secional risk. From the table below we can see that all factors are not significant in the regression, indicating that CSD can be priced cross-sectionally, and is a distinct measurement of cross sectional risk.

<img src="https://user-images.githubusercontent.com/60916875/190038401-93e207e9-cb72-459d-abcc-5ce460437e20.png" width = "500">

### Compare with Original Paper

By extending the testing period, our results are different from the original paper. This indicate that the stability of CSD is not as strong as it's stated in the original paper

* Overlap

  + Using the return of sorting portfolios, our paper showed that the risk premium      attained from CSD is in fact independent from other established systematic risk       factors (all factors' t-test are significant under 5% confidence level, with a     significant interception term)
  
  + The CSD is indeed a good measure of aggregated idiosyncratic risk of individual     stocks, when conducting Fama-MacBeth two-pass regression with other measurements, it is significant that CSD is comprised with different idiosyncratic risk.(With Adjusted R-squared closed to 1.0)

* Difference

  + Our experiment in section 5.2.1 showed that the sorting portfolio returns does not have a very strong increasing order with smaller CSD beta. We also showed that the negative relationship between CSD and expect returns is rejected in the t-test, which differs from the results in original paper. 
 

# 6 Future Work

Currently the team only analyzed the cross-sectional dispersion with Standard Poor 500 Composite Index data as our market return, while the original research paper used the CRSP value-weighted index as a proxy for aggregated market returns. It is possible that a different index or calculation for the market return could have delivered a different CSD factor, and thus a different regression result. Another important evaluation that needs to be implemented is the stock selection process in this replication. Currently the team only utilized stocks that were consistently traded. All delisted stocks were eliminated from our factor calculation. This definitely is a biased data handling approach and should be further scrutinized. It is almost certain that there is no way to predict if a stock will be delisted from the stock market or not, therefore it is inaccurate to eliminate all delisted stocks from the beginning, while conducting this data analysis.

However, it certainly opens up new opportunities for the team to look at the calculation of the Cross-Sectional Dispersion factor calculation. From the original formula: 

$$CSD_t= \frac{\sum_{i} |r_{i,t}-r_{mkt,t}|}{N-1}$$ 

It is shown that the CSD is an equally weighted factor, but since we mentioned that eliminating delisted stocks brings a different CSD value, it is easy to imply that CSD could be expressed as a weighted factor in such a way that:

$$CSD_t^{'}=\frac{\sum_{i} \lambda_{i,t}^* |r_{i,t}-r_{mkt,t}|}{N-1}$$ 

Where i,t represents a weight, regarding different stocks. This weight can be informally calculated by the market cap proportion of the corresponding stock, or can be further optimized using a reinforcement learning approach such as Q-learning to determine the optimal value. This approach would make CSD a more refined factor that can be deliberately tuned. 

#### Eliminate correlation

Another improvement that can be examined is the correlation between a certain stock return and the market return. By eliminating the correlation of returns we might be able to calculate an unbiased CSD that represents the neutral fluctuation and volatility of the market. In this case we have

$$CSD_t^{''}=\frac{\sum_{i} \lambda_{i,t}^* |\rho_{i,t}^* r_{i,t}-r_{mkt,t}|}{N-1}$$ 

Where i,t represents the correlation between stock i and market return, which can be calculated by a linear regression of stock returns.


## Conclusions

In this paper, we replicated the research of Cross-Sectional Dispersion and Expected Returns by Thanos Verousis and Nikolaos Voukelatos in 2016. We used data from January 1996 to December 2012 to test the relationship between Cross-sectional Dispersion and the expected stock returns. First, we computed the $\Delta CSD$ factor, regression on all the stocks to obtain a mimicking portfolio. We then show that the mimicking portfolio had mild negative returns, and further proved that CSD has negative relationship with stocks' expected returns. In order to show that CSD is a new factor, we test the risk premium of CSD with the risk premia of some established systematic risk factors, the results indicated that CSD is a distinct risk factor. And it has similar performance under different market conditions, demonstrating the robustness of this factor. We finally use different measurement of cross-sectional risk to prove that CSD is a priced factor, and can be a distinct measures of aggregated idiosyncratic risk of individual stocks cross-sectionally. 

We denote that the data we use is slightly different from the original paper, due to the missing data. So the results were less significant than the original paper. We suggest further analysis on more recent period. Because of the pandemic, the fundamental of market has experienced some changes, the factors that were once significant might generate less returns compared to the past. We also discovered that the alternative calculation methodology for CSD may generate different results, so further study on the robustness with calculation method is strongly suggested.

## Reference

Amit Goyal, Pedro Santa-Clara. 2003. “Idiosyncratic Risk Matters!” Journal of Finance 98: 975–1008.

Ang, Hodrick, A., and X. Zhang. 2006. “The Cross-Section Ofvolatility and Expected Returns.” Journal of Finance 61: 259–99.

Angelidis, Sakkas, T., and N. Tessaromatis. 2015. “Stock Market Dispersion, the Business Cycle and Expected Factor Returns.” Journal of Banking and Finance 59: 265–79.

Bali, Brown, T., and Y. Tang. 2015. “Macroeconomic Uncertainty and Expected Stock Returns.”

Bali, Turan. n.d. https://sites.google.com/a/georgetown.edu/turan-bali/data-working-papers.

Brennan, Chordia, M., and A. Subrahmanyam. 1998. “Alternative Factor Specifications, Security Characteristics, and the Cross-Section of Expected Returns.” The Journal of Financial Economic 49 (4): 345–73.

Carhart, M. 1997. “On Persistence in Mutual Fund Performance.” Journal of Finance 52: 57–82.

Chen, Z., and R. Petkova. 2012. “Does Idiosyncratic Risk Proxy for Risk Exposure?” Review of Financial Studies 25: 2745–87.

CRSP. n.d. https://www.crsp.org/.

Diether, C., K. Malloy, and A. Scherbina. 2002. “Differences of Opinion and the Cross-Section of Stock Returns.” Journal of Finance 57: 2113–41.

Fama, E., and K. French. 2002. “Differences of Opinion and the Cross-Section of Stock Returns.” Journal of Finance 33: 3–56.

Fama, E., and J. Macbeth. 1973. “Risk, Return, and Equilibrium: Empiricaltests.” Journal of Political Economy 71: 607–36.

French, Kenneth. n.d. https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html.

Garcia, Mantilla-Garcia, R., and L. Martellini. 2014. “A Model-Free Measure of Aggregate Idiosyncratic Volatility and the Prediction of Market Returns.” Journal of Financial and Quantitative Analysis. https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1973471.

Harvey, C., and A Siddique. 2000. “Conditional Skewness in Asset Pricing Tests.” Journal of Finance 55(4): 1263–95.
28

Pastor, L., and R. Stambaugh. 2003. “Liquidity Risk and Expected Stock Returns.” Liquidity Risk and Expected Stock Returns 111: 642–85.

Verousis, Thanos, and Nikolaos Voukelatos. 2015. “Cross-Sectional Dispersion and Expected Returns.”

