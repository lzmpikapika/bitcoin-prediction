# bitcoin-prediction
# Prolem statement
In order to study the price trend of the bitcoin market and how to get the most benefit in the near future, a bitcoin price prediction was made.
# Abstract
Use data from Yahoo Finance to build two models: arima and svr for predict bitcoin price. The results show that the svr model has a high accuracy rate, can predict the price of Bitcoin in the future, and solves the problem of how to get most benefit in the Bitcoin market.
# Code 
## ARIMA
load the data from Yahoo Finance,and use quantmod package
```
daily_BTC_USD<- read.csv("daily_BTC.csv")
weekly_BTC_USD <- read.csv("weekly_BTC_USD.csv")
monthly_BTC_USD <- read.csv("monthly_BTC_USD.csv")
```
```
library(quantmod)
```
determine the price of each day,and diterming the price fluctuation.
```
daily_BTC_USD$Price <- (daily_BTC_USD$Open +daily_BTC_USD$Close)/2
daily_BTC_USD$Diff_price <- (daily_BTC_USD$Close -daily_BTC_USD$Open)
```
```
library(MASS)
library(tseries)
library(forecast)
```
devided the trainging data
```
training <- log(new_bit$price[1:365])
training
```
determing the p and q values,then make plots.diff=1
```
acf(training,lag.max = 50)
pacf(training,lag.max = 50)
```
![Image](http://github.com/lzmpikapika/bitcoin-prediction/raw/master/acf.jpg)
when the value=1 there has a peak, so the diff=1
```
difftraining <- diff(training,1)
difftraining
```
use adf test the stationarity.
```
adf.test(training)
adf.test(difftraining)
```
training one year's data and get plot
```
pricepredict <- ts(training,start = c(2019,01),frequency = 365)
fittraining <- auto.arima(pricepredict)
fittraining
```
There are 93 test data,so run th 93 time period.
```
forecasrvalues <- forecast(fittraining,h=93)
forecasrvalues
plot(forecasrvalues)

forecasrvaluesex <- as.numeric(forecasrvalues$mean)
finalforecast <- exp(forecasrvaluesex)
finalforecast
```
use validation data set to test and calculate percentage error
```
validation <- data.frame(new_bit$price[366:458],finalforecast)
dataname <- c("actual","forecasted")
names(validation) <- dataname
attach(validation)
percentage_error <-((validation$"actual"-validation$"forecasted")/(validation$"actual"))
percentage_error
mean(percentage_error)
```
use test set (2020 April data) to test the model
```
forecasrvalues1 <- forecast(fittraining,h=26)
forecasrvalues1
plot(forecasrvalues1)

forecasrvaluet <- as.numeric(forecasrvalues1$mean)
testforecast <- exp(forecasrvaluet)
testforecast
```
![Image](http://github.com/lzmpikapika/bitcoin-prediction/raw/master/ARIMA.jpg)
# svr
load package
```
library(tidyverse)
library(e1071)
```
divided dataset into training data(365days),validation data(93days),testing data(26days)
```
new_bit$volume <- new_bit$volume/1000000
bittrain <- new_bit[1:365,]
bitvalidation <- new_bit[365:458,]
bittest <- new_bit[458:484,]
```
make regression between price(price for one day opening)and close,volume
```{r}
regressor <- svm(formula=price~close+volume,data=new_bit,
                 type="eps-regression",
                 kernel="radial",cost=1,sigma=)
summary(regressor)
```
predicted the value use validation data.
![Image](http://github.com/lzmpikapika/bitcoin-prediction/raw/master/svrval.jpg)
usd plot show the changes
```
ggplot()+
  geom_point(aes(x=bitvalidation$close,y=bitvalidation$price),colour="red")+
  geom_line(aes(x=bitvalidation$close,y=predict(regressor,newdata = bitvalidation)),colour="blue")+
  ggtitle("bitcoin prediction")+
  xlab('close')+
  ylab('volume traded')
```
![Image](http://github.com/lzmpikapika/bitcoin-prediction/raw/master/svrprediction.jpg)
No.484 row test.price is 2020.04.27 close price,and price predict result is forecast the 2020.04.28 opening price.It is decline, can consider selling.
```{r}
pricecomparetest <- data.frame(bittest$price,pricepredict1)
pricecomparetest
```
![Image](http://github.com/lzmpikapika/bitcoin-prediction/raw/master/svrtestprice.jpg)
