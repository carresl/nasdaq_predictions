# Beating the Nasdaq-100 Index by Predicting Top Performers

## Overview
In this project, I aim to outperform the Nasdaq-100 (QQQ) by selecting stocks from the index which have the most favorable risk-adjusted returns. In pursuit of this goal, I run two separate tests:
1) The first aims to establish that the model I created has predictive power by comparing the average gain of stocks that were given low and high ratings by the model.
2) The second aims to establish that the model is capable of outperforming the QQQ ETF by selecting stocks with favorable risk-adjusted returns.

### Test #1
In the Test 1 folder there are two files. Difference_in_Ratings.ipynb carries out the first test by running a trading simulation from January 1, 2023 through May 6, 2025. The Test1_results.csv file contains the ratings and percentage gain for each trade that is made during this simulation.
At the end of the Jupyter Notebook, a t-test is performed for the hypothesis that the average gain in stocks which were given ratings greater than one is greater than the average gain in stocks which were given a rating of less than negative one. The t-statistic is 1.691 with a p-value of 0.0462, making it significant at the 5% level. This establishes that the model has predictive power in forecasting the returns of a stock.

### Test #2
In the Test 2 folder there is a file called Trading_Simulations.ipynb. This file runs ten trading simulations that start on January 1, 2024 and run through June 9, 2025. Each simulation starts with a portfolio of size $1,000,000. There is another folder within Test 2 called Simulation Reports which contains the simulation reports for all ten simulations. These simulation reports show all the trades which were made in each simulation. This includes the date of the trade, the ticker of the stock being traded, whether the trade is long or short (note: there is no short selling in these simulations but the Jupyter Notebook is set up to accomodate short selling), the rating given by my strategy (see Methodology: Ratings and Risk-adjustment for more on how the rating is determined), the initial position size when it is opened, the final position size when it is closed, the P&L in dollars, the P&L in percentage, the total P&L for the 45 trading day period over which the given position was held, and the total capital which represents the total amount of capital in the portfolio after the end of the 45 day trading period in which the given trade was made.
At the end of the Jupyter Notebook, it outputs the results of a t-test for the hypothesis that average gain across the ten simulations outperforms the gain of the QQQ ETF over the same time period. The average gain across the ten simulations using my strategy was 38.14% which is greater than the 32.80% gain seen in QQQ. The t-test produced a t-statistic of 0.6722 with a p-value of 0.259. This means that although my strategy outperformed QQQ on average, there is insufficient evidence to suggest that this result is not due to random chance. This is an understandable result and aligns with the high variance between the returns in the simulations (see Results for more).

In the rest of this document I go into further details about the methodology and results and include a brief discussion about the results of the project as well as a few concerns and limitations which could be improved upon in the future.

## Methodology

### Training, Validation, and Testing Data
The data used to make predictions is a time series of the previous 200 trading days of open, high, low, close, and volume (OHLCV) data. The response variable is the percentage change in price from the open on the following day to the open 45 trading days later. The training data is used to train the model, the validation data is used to calculate the model risk (see Ratings and Risk-adjustment for more on the model risk), and the testing data is used to evaluate the ratings in Test 1 and run the simulation in Test 2. Training data is cutoff at 550 calendar days before the current prediction day to ensure that in the worst case (when there are many market holidays) there will be no overlap between training, validation, and testing data. Training data is also restricted to only the four previous years to ensure relevance of the data. Validation data is cutoff at the day before the current prediction day and 200 calendar days before the last training day to ensure no leakage from the training data.

### Model
The model that creates the trade ratings is a neural network that contains two LSTM layers (the first returns sequences), then two fully-connected (Dense) layers with multiple units, and finally one Dense layer for the output. This model outputs a predicted gain for each stock over the next 45 trading days

### Ratings and Risk-adjustment
There are three factors that go into the rating given to each stock. First is the prediction for the expected return of the stock over the next 45 trading days generated by the model. Then, the average return of QQQ for each 45-trading-day period over the training period is subtracted from this number. This is done to favor stocks which are expected to outperform the ETF. The third and final factor is the model risk. Model risk is defined here as risk arising from inaccurate model predictions. It is calculated by making predictions on the validation data, and calculating the RMSE for each stock in the data set. Then, in the simulation all predictions are adjusted for the stock-specific model risk by dividing the difference between the prediction and the average QQQ return by this RMSE (note: the RMSE is calculated individually for each stock rather than for the model as a whole to account for the fact that the model may be worse at predicting certain volatile stocks in the ETF).

### Making Trades
In the simulations in Test 2, trades are made every 45 trading days. They are simulated as if the stock was purchased at the open price on the next day and sold at the open price 45 trading days later. Only stocks which have a risk-adjusted rating greater than one are bought at each interval because these represent stocks which have attractive risk-return profiles.

### Evaluating Performance
Performance is evaluated differently for each test:
  1) In Test 1, performance is evaluated using a t-test for difference in mean gain for stocks with a rating below negative one and stocks with a rating greater than one
  2) In Test 2, performance is evaluated using a t-test for difference in performance across ten simulations versus the performance of the QQQ ETF over the same time period

The first test will indicate whether the ratings are effective at differentiating between overperforming and underperforming stocks, and the second test will indicate whether this is a viable strategy to outperform the Nasdaq-100.

## Results
### Test #1
Across 206 predictions, the mean gain over 45 trading days was 2.11% for the 126 stocks with a rating less than negative one, and 5.93% for the 80 stocks with a rating greater than one (note: the average return for QQQ every 45 trading days during the training period was 4.43%, this number is both greater than the mean of the stocks which were given ratings less than negative one and less than the mean of the stocks which were given ratings greater than one). The t-statistic for the hypothesis that stocks with a rating greater than one have higher average gains than stocks with ratings less than negative one was 1.691 with a p-value of 0.0462. This is significant at the 5% level, and therefore, it can be concluded that stocks with a rating greater than one higher gains on average than stocks with scores less than negative one.

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

Clearly, there is high variance in the returns of the simulations which raises concerns about the reliability of the strategy and makes it impossible to rule out the possibility that the outperformance is due to random chance. This is reflected in the t-statistic of 0.6722 and p-value of 0.259 for the hypothesis that the mean return using my strategy is greater than the performance of QQQ over the same testing period.

## Discussion
The result of the first test is promising as it shows that the model has some capacity to differentiate between stocks which overperform and underperform the index. This suggests that the model may have the capability to identify stocks that will outperform the index and generate alpha.

The result of the second test is somewhat disappointing. Although the result from the first test suggests that it is possible to select outperforming stocks from an ETF and only buy those, the results of the simulation suggest that in reality this task is more difficult than it seems. A closer look at the simulation reports shows that the reason for this is likely a lack of diversity. When my strategy is wrong, it is often wrong in an position of large size. In an ETF, diversity among a much larger number of positions mitigates such a risk.

### Concerns
There are several concerns with the results of this project due to factors/risks which were unaccounted for. Some of the major ones are listed below.
1) **Short testing period:** The simulation was only run on a testing period of around 18 months. This is far too short to generalize that my strategy will outperform the ETF over a longer time period in different market conditions.
2) **Slippage:** In the simulations, orders were assumed to be executed at exactly the daily open price. In reality, this is unlikely to be feasible, and there is likely to slippage that was not accounted for in this simulation.
3) **Taxes:** My strategy requires making trades every 45 trading days, whereas an ETF can be bought and held. This means that the trades made using my strategy would be subject to short-term capital gains taxes which would reduce the returns of my strategy compared to the ETF which would not accrue these costs.

### Limitations
There were several limitations that affected the results of this project. A couple are listed below.
1) **Computing Resources:** The model used to make predictions was a neural network with several layers, and it took a substantial amount of time to train in each simulation. Training on a CPU would take far too long to produce results to tune the parameters, so I had to pay for GPU access through Google Colab. I wanted to limit my spending, so I was not able to run as many simulations as I would have liked to in order to find optimal parameters.
2) **Data:** I only used publicly available data for this project (specifically just daily OHLCV data). Obviously, there are many more factors that affect stock prices that were not accounted for. As an individual, it is difficult to access high quality data especially given my spending restrictions, and this impacted the results of this project.

### Final Thoughts
Although I do not believe there is any reason to believe that my strategy is capable of outperforming the Nasdaq-100 in the long run, I do believe that the result of the first test does offer significant promise for future work. The model was able to effectively differentiate between stocks which outperformed and underperformed the index even given the aforementioned limitations. This is an exciting result and something that I believe can be built upon to create a profitable strategy.



*Thanks for taking the time to read, feel free to contact me at clagro@andrew.cmu.edu or via [LinkedIn](https://linkedin.com/in/carres) if you have any questions or thoughts*
