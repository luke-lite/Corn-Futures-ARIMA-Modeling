# Corn Futures ARIMA Modeling
With this project I am attempting to create an ARIMA model that can be used to accurately predict the price of corn at 1, 3, 6, 9, and 12-month intervals. This project is currently in the preliminary phase, but I plan to continue making improvements and incorporating additional data in the future.

Check the [notebook](https://github.com/luke-lite/Corn-Futures-ARIMA-Modeling/blob/main/corn-futures-arima-modeling.ipynb) for a breakdown of the code and description of the processes I am using.

## Data
I am using historical daily corn price data provided by [Trading Economics](https://tradingeconomics.com/). After verifying and cleaning the data, I created monthly averages to use for training a model, and split the data into training and test sets:

![train_test_data.png](https://github.com/luke-lite/Corn-Futures-ARIMA-Modeling/blob/5ed81799472f8dbe420210e1353b4555cc012e62/train_test_data.png)

## Modeling
I ran some tests on the monthly averages data and discovered it was seasonal and non-stationary. This was expected, but meant that I would need to be careful selecting my ARIMA parameters. I used [auto-arima](https://alkaline-ml.com/pmdarima/modules/generated/pmdarima.arima.auto_arima.html) to test a variety of parameters, with the best being a SARIMA(1,1,0)(0,0,[1,2],12) model:
```
                                        SARIMAX Results                                        
===============================================================================================
Dep. Variable:                                       y   No. Observations:                  217
Model:             SARIMAX(1, 1, 0)x(0, 0, [1, 2], 12)   Log Likelihood                 -40.149
Date:                                 Fri, 26 Jul 2024   AIC                             88.298
Time:                                         14:41:25   BIC                            101.800
Sample:                                     01-31-2000   HQIC                            93.753
                                          - 01-31-2018                                         
Covariance Type:                                   opg                                         
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
ar.L1          0.2456      0.062      3.950      0.000       0.124       0.367
ma.S.L12      -0.1452      0.039     -3.696      0.000      -0.222      -0.068
ma.S.L24      -0.1153      0.045     -2.534      0.011      -0.204      -0.026
sigma2         0.0846      0.004     19.672      0.000       0.076       0.093
===================================================================================
Ljung-Box (L1) (Q):                   0.00   Jarque-Bera (JB):               365.30
Prob(Q):                              0.97   Prob(JB):                         0.00
Heteroskedasticity (H):               5.78   Skew:                            -0.33
Prob(H) (two-sided):                  0.00   Kurtosis:                         9.34
===================================================================================

Warnings:
[1] Covariance matrix calculated using the outer product of gradients (complex-step).
```

For an overview of how to interpret some of these statistical measures, you can check my [blog post](https://luke-lite.github.io/blog/statistical-measures-in-statsmodels/). But for this particular model, the biggest issue is that the data appears to be non-normal. This is something that ideally needs to be addressed, but it will need to wait for a future iteration of the project.

## Results

Using these parameters, I fit a model using the training data. From there, I made 1, 3, 6, 9, and 12 month forecasts for each month of the test data, refitting the model with each new data point to ensure the model was up-to-date. The average root-mean square error for each interval was:

```
1-month RMSE = 0.3446
3-month RMSE = 0.7718
6-month RMSE = 1.1236
9-month RMSE = 1.3106
12-month RMSE = 1.4143
```

These values are in real dollars, so the average error for 1-month predictions was .34 cents, and the average error for 12-month predictions was $1.41.

Overall, the error is obviously too large for the model to be useful in anticipating corn futures, but given that this is a preliminary model, I think the results are promising.

One major issue is that ARIMA models rely on the autoregressive nature of the data, meaning that past price activity informs future price movements. However, my test set include the COVID pandemic, a period of huge price volatility that obviously can't be accounted for by a time-series model. In such instances, it is advisable to focus on more traditional supply-demand price analysis. This also means that my test results will be worse than if the model was tested on a more "normal" market window.

There are several more adjustments that could be made, such as factoring in supply-demand price predictions provided by the USDA in their regular [WASDE reports](https://www.usda.gov/oce/commodity/wasde). Additionally, creating a bi-variate model that include the price movements from a different exchange could also further improve the accuracy of the model. Another option is to include up-to-date movements in the futures market to influence the model predictions at different time intervals. All of these are methods I plan on exploring in future iterations of the project.
