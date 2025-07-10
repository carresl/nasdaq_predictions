# Selecting Stocks from the Nasdaq 100 using Machine Learning

## Overview
In this project, I aim to outperform the Nasdaq 100 (QQQ) by selecting stocks from the index which have the most favorable risk-adjusted returns. In pursuit of this goal, I run two separate tests:
1) The first aims to establish that the model I created has predictive power by comparing the average gain of stocks that were given low and high ratings by the model.
2) The second aims to establish that the model is capable of outperforming the QQQ ETF by selecting stocks with favorable risk-adjusted returns.

Difference_in_Ratings.ipynb carries out the first test stated above by running a trading simulation from January 2, 2024 through May 6, 2025. At the end of the notebook, it outputs the results of t-test for the hypothesis that the average gain in stocks which were given ratings greater than one is greater than the average gain in stocks which were given a rating of less than negative one. The t-statistic is 1.691 with a p-value of 0.0462, making it significant at the 5% level. This establishes that the model has predictive power in forecasting the returns of a stock.

Trading_Simulations.ipynb carries out the second test stated above by running ten trading simulations starting on January 2, 2024 through June 9, 2025. At the end of the notebook, it outputs the results of a t-test for the hypothesis that average gain across the ten simulations outperforms the gain of the QQQ ETF over the same time period. The average gain across the ten simulations using my strategy was 38.14% which is greater than the 32.80% gain seen in QQQ. The t-test produced a t-statistic of 0.6722 with a p-value of 0.259. This means that although my strategy outperformed QQQ on average, there is insufficient evidence to suggest that this result is not due to random chance. This is an understandable result and aligns with the high variance between the returns in the simulations (see results for more).

In the rest of this document I go into further details about the methodology and results and include a brief discussion of the limitations of the simulation which can be improved in the future.

## Methodology

### Data Splitting
The input data is a time series of the previous 200 days of open, high, low, close, and volume (OHLCV) data. Training data is cutoff at 550 calendar days before the current prediction day to ensure that in the worst case (when there are many market holidays) there will be no overlap between training, validation, and testing data. Training data is also restricted to only the four previous years to ensure relevance of the data. Validation data is cutoff at the day before the current prediction day and 200 days before the last training day to ensure no leakage from the training data.

### Model
The model that creates the trade ratings is a neural network that contains two LSTM layers (the first returns sequences), then two fully-connected (Dense) layers with multiple units, and finally one Dense layer for the output. This model outputs a predicted gain for each stock over the next 45 trading days

### Risk-adjustment
Given the signifiacnt amount of model risk, predictions are adjust to account for this risk. To do this, predictions are made on the validation data, and the RMSE is calculated for each stock in the data set. Then, in the simulation all predictions are adjusted for the stock-specific model risk by dividing the prediction by the RMSE. Then only stocks which have a risk-adjusted rating larger than one (representing an attractive risk to return profile) are selected.

### Making Trades/Predictions
Predictions are made for the gain of a given stock over the next 45 trading days (calculated as the change in price from the open on the day after the prediction is made through the open 45 trading days later).

In simulations, trades are made every 45 trading days. They are simulated as if the stock was purchased at the open price on the next day and sold at the open price 45 trading days later. Only stocks which have a risk-adjusted rating greater than one are selected. Predictions are also adjusted to account for the "risk-free rate" which is the average return of the underlying ETF (QQQ) every 45 trading days during the training period.

### Evaluating Performance
Performance is evaluated in two ways:
  1) t-test for difference in mean gain for stocks with a rating below negative one and stocks with a rating greater than one
  2) t-test for difference in performance across 10 simulations versus baseline performance of the QQQ ETF over the same time period
The first method will indicate whether the ratings are actually effective at differentiating between overperforming and underperforming stocks, and the second method will indicate whether this strategy is capable of consistently generating alpha.

## Results
### Test #1
Across 206 predictions, the mean gain over 45 trading days for was 2.11% for stocks with a rating less than negative one, and 5.93% for stocks with a rating greater than one. The t-statistic for the hypothesis that stocks with a rating greater than one have higher average gains than stocks with ratings less than negative one was 1.691 with a p-value of 0.0462. This is significant at the 5% level, and therefore, it can be concluded that stocks with a rating greater than one have average higher gains than stocks with scores less than negative one.

### Test #2
The mean return over ten simulations using my strategy was 38.14% (median: 47.19%) compared with a 32.80% return in QQQ over the same time period. Although this result seems to imply that my strategy is effective at outperforming the ETF, a closer look at the results casts doubt on such a conclusion. The returns of all ten simulations are as follows:
1) 47.32%
2) 47.19%
3) 62.73%
4) 61.24%
5) 7.93%
6) 18.93%
7) 42.97%
8) 15.03%
9) 74.49%
10) 3.53%

Clearly, there is high variance between the performance of each simulation which raises concerns about the reliability of the strategy and makes it impossible to rule out the possibility that the outperformance is due to random chance. This is reflected in the t-statistic of 0.6722 and p-value of 0.259 for the hypothesis that the mean return using my strategy is greater than the performance of QQQ over the same testing period.

## Discussion
The result of the first test is highly promising as it shows that the model is capable of identifying stocks that are likely to have greater returns. This result is promising for optimizing an ETF to remove underperformers and only trade those expected to outperform.

The result of the second test is somewhat disappointing. Although the result from the first test suggests that it is possible to select outperforming stocks from an ETF and only own those, the results of the simulation suggest that in reality this task is more difficult than it seems. A closer look at the simulation reports shows that the reason for this is likely a lack of diversity. When my strategy is wrong, it is often wrong in an position of large size. In an ETF, diversity among a much larger number of positions mitigates such a risk.

### Concerns
There are many limitations that harm the results of this project. Some of the major ones are listed below.
1) **Short testing period:** The simulation was only run on a testing period of around 18 months. This is far too short to generalize that my strategy will outperform the ETF over a longer time period in different market conditions.
2) **Slippage:** In the simulations, orders were assumed to be executed at exactly the daily open price. In reality, this is unlikely to be feasible, and there is likely to slippage that was not accounted for in this simulation.
3) **Taxes:** My strategy requires making trades every 45 trading days, whereas an ETF can be bought and held. This means that the trades made using my strategy would be subject to short-term capital gains taxes which would reduce the returns of my strategy compared to the ETF which would not accrue these costs.

### Limitations
There were several limitations that affected the results of these project. Some are listed below.
1) **Computing Resources:** The model used to make predictions was a neural net with several layers. This takes a substantial amount of time to train for each simulation. To run these simulations on a CPU would take so long that it would be impossible to tune model parameters as I did, so I had to pay for GPU access on Google Colab in order to get results in a reasonable amount of time. I was averse to spending a signifiacnt amount of money on this project, so I tried to run as infrequently as possible in order to save money. This had some effect on the results I was able to produce as I was unable to run the simulations as frequently as I would have liked to (and even when I did it took over four hours to run ten simulations which means that that there was a significant time delay before I could see my results).
2) **Data:** I only used publicly available data for this project (specifically just daily OHLCV data). Obviously, this is not nearly enough data to make accurate predictions about future prices. As an individual, it is difficult to access high quality data especially given my restriction on spending money, so this added another major obstacle in achieving decent results.

### Final Thoughts
Although I do not believe there is any reason to believe that my strategy is capable of outperforming the ETF in the long run, I think the result of the first test does offer significant promise for future work. The model was able to effectively differentiate between stocks which underperformed and stocks that outperformed even given the aforementioned limitations. This is an exciting result and something that I believe can be built upon to create a profitable strategy.

*Thanks for taking the time to read, feel free to contact me via LinkedIn (linkedin.com/in/carres) or email (clagro@andrew.cmu.edu) if you have any questions or thoughts*
