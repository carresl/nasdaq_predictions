# Selecting Stocks from the Nasdaq 100 using Machine Learning

## Overview

In this project I aim to select stocks based on expected risk-adjusted returns. The goal is to select stocks from the Nasdaq 100 (QQQ) which are expected to have 

The Google Colab notebook contains several parameters which were tuned over many simulations but can still be adjusted for future testing.


## Methodology

### Data Splitting
The input data is a time series of the previous 200 days of open, high, low, close, and volume (OHLCV) data. Training data is cutoff at 550 calendar days before the current prediction day to ensure that in the worst case (when there are many market holidays) there will be no overlap between training, validation, and testing data. Training data is also restricted to only the four previous years to ensure relevance of the data. Validation data is cutoff at the day before the current prediction day and 200 days before the last training day to ensure no leakage from the training data.

### Model
The model that creates the trade ratings is a neural network that contains two LSTM layers (the first returns sequences), then two fully-connected (Dense) layers with multiple units, and finally one Dense layer for the output. This model outputs a predicted gain for each stock over the next 45 trading days

### Risk-adjustment
Given the signifiacnt amount a model risk, predictions are adjust to account for this risk. To do this, predictions are made on the validation data, and the RMSE is calculated for each stock in the data set. Then, in the simulation all predictions are adjusted for the stock-specific model risk by dividing the prediction by the RMSE. Then only stocks which have a risk-adjusted rating larger than 1 (representing an attractive risk to return profile) are selected.

### Making Trades/Predictions
Predictions are made for the gain of a given stock over the next 45 trading days (calculated as the change in price from the open on the day after the prediction is made through the open 45 trading days later).

In simulations, trades are made every 45 trading days. They are simulated as if the stock was purchased at the open price on the next day and sold at the open price 45 trading days later. Only stocks which have a risk-adjusted rating greater than one are selected.

### Evaluating Performance
Performance is evaluated in two ways:
  1) T-Test for difference in mean gain for stocks with a rating below negative one and stocks with a rating larger than one
  2) T-test for difference in performance across 10 simulations versus baseline performance of the QQQ ETF over the same time period
The first method will indicate whether the ratings are actually effective at differentiating between overperforming and underperforming stocks, and the second method will indicate whether this strategy is capable of consistently generating alpha.

## Results

// state results

## Discussion & Future Work

// discussions


*Thanks for taking the time. Feel free to contact me if you have any questions or thoughts.*
