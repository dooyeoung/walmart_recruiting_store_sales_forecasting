# Walmart Recruiting - Store Sales Forecasting


## Description
One challenge of modeling retail data is the need to make decisions based on limited history. If Christmas comes but once a year, so does the chance to see how strategic decisions impacted the bottom line. 

In this recruiting competition, job-seekers are provided with historical sales data for 45 Walmart stores located in different regions. Each store contains many departments, and participants must project the sales for each department in each store. To add to the challenge, selected holiday markdown events are included in the dataset. These markdowns are known to affect sales, but it is challenging to predict which departments are affected and the extent of the impact.

Want to work in a great environment with some of the world's largest data sets? This is a chance to display your modeling mettle to the Walmart hiring teams.

This competition counts towards rankings & achievements.  If you wish to be considered for an interview at Walmart, check the box "Allow host to contact me" when you make your first entry. 

You must compete as an individual in recruiting competitions. You may only use the provided data to make your predictions.

## Evaluation
This competition is evaluated on the weighted mean absolute error (WMAE):
$$
WMAE = \frac{1}{\sum W_i}\sum^n_{i=1}W_i|y_i-\hat y_i|
$$
where

- n is the number of rows
- ŷ i is the predicted sales
- yi is the actual sales
- wi are weights. w = 5 if the week is a holiday week, 1 otherwise

## Data
You are provided with historical sales data for 45 Walmart stores located in different regions. Each store contains a number of departments, and you are tasked with predicting the department-wide sales for each store.

In addition, Walmart runs several promotional markdown events throughout the year. These markdowns precede prominent holidays, the four largest of which are the Super Bowl, Labor Day, Thanksgiving, and Christmas. The weeks including these holidays are weighted five times higher in the evaluation than non-holiday weeks. Part of the challenge presented by this competition is modeling the effects of markdowns on these holiday weeks in the absence of complete/ideal historical data.

**stores.csv**

This file contains anonymized information about the 45 stores, indicating the type and size of store.

**train.csv**

This is the historical training data, which covers to 2010-02-05 to 2012-11-01. Within this file you will find the following fields:

- Store - the store number
- Dept - the department number
- Date - the week
- Weekly_Sales -  sales for the given department in the given store
- IsHoliday - whether the week is a special holiday week

**test.csv**

This file is identical to train.csv, except we have withheld the weekly sales. You must predict the sales for each triplet of store, department, and date in this file.

**features.csv**

This file contains additional data related to the store, department, and regional activity for the given dates. It contains the following fields:

- Store - the store number
- Date - the week
- Temperature - average temperature in the region
- Fuel_Price - cost of fuel in the region
- MarkDown1-5 - anonymized data related to promotional markdowns that Walmart is running. - MarkDown data is only available after Nov 2011, and is not available for all stores all the time. Any missing value is marked with an NA.
- CPI - the consumer price index
- Unemployment - the unemployment rate
- IsHoliday - whether the week is a special holiday week

For convenience, the four holidays fall within the following weeks in the dataset (not all holidays are in the data):

Super Bowl: 12-Feb-10, 11-Feb-11, 10-Feb-12, 8-Feb-13<br>
Labor Day: 10-Sep-10, 9-Sep-11, 7-Sep-12, 6-Sep-13<br>
Thanksgiving: 26-Nov-10, 25-Nov-11, 23-Nov-12, 29-Nov-13<br>
Christmas: 31-Dec-10, 30-Dec-11, 28-Dec-12, 27-Dec-13

## Result

- 각기다른 지역의 45개 점포의 과거 자료를 제공한다
- 2010-02-05 ~ 2012-10-26일까지 일주일 간격으로 143주의 데이터가 있다
- 2012-11-02 ~ 2013-07-26일까지 39주간의 판매량을 예측 하여야 한다

### EDA

- 판매량과 다른 변수들 간의 상관관계
  - 다른 변수들과 관련성을 찾기 힘들다
- 휴일에 영향을 받는 Department 파악
- 결측치 파악
  - 특정 Department에 결측값이 집중되어있다
  - 데이터 불균형을 찾을 수 있었으며 모델링에 큰 어려움이 있었다
- 과거 판매량에 얼마나 영향을 받는지 파악
  - 계절성을 파악하기위해 연도별 판매량을 분석
  - 대부분의 자료가 1년전 판매량과 비슷한 수치를 갖고 있다
- 45개 Store 클러스터링
  - 연도별 판매량 상위 10개 품목, 판매량 상위 3일, 판매량 하위 3개 품목으로 군집화
  - Store별 Department데이터가 조금씩 다르기 때문에 store 군집화는 큰 의미는 없었다

### Analysis

xgboost로 회귀분석을 fbprophet을 사용하여 시계열 분석을 진행

#### XGboost regressor

- 결측값이 많은 Markdown1~5 제외 
- IsHolyday, Type, Week, Month, Store, Dept변수를 onehot vector로 변환
  - score = 23451.53163  rate = 680 / 691 
- lag4를 추가로사용하여 분석 진행
  - score = 20818.13186  rate = 631 / 691 

> 너무 좋지 않은 성적이다. 더 나은 변수를 찾기위해 feature engineering작업이 필요하다
>
> 데이터 불균형으로 특정 데이터의 예측이 불안하다

#### FBpophet

- train데이터에서 143주 판매량이 있는 데이터와 결측값이 존재하는 데이터를 분류하여 분석 진행
- 2/3이상 결측값이 있는 데이터는 결측값 없는 데이터의 예측값들의 평균을 예측값으로 사용하였다
- 특별한 데이터 전처리 없이 시계열 분석만으로 50%의 성적을 얻을 수 있었다
  - score = 4100.51057  rate = 369 / 691 

> 결측치 처리가 큰 문제였다. 데이터 자체가 없어 시계열 모델을 만들 수 없는 Departments가 존재하였다
>
> departments와 store를 clustering과 classification을 시도하여 결측치 처리에 사용해봐야겠다

### conclusion

45개 Store에 최대 81개 Department에 대해서 2012-11-02 ~ 2013-07-26 39주간 주간 판매량을 예측하는 프로젝트 였다

회귀분석으로 예측값을 구하기 위해서는 feature engineering을 통해 더 많은 상관관계를 구해야 한다

시계열 분석으로 특별한 작업없이 50% 성적을 받을 수 있었다

결측치 처리가 성적을 높일 수 있는 큰 요인이라 생각한다

비록 성적은 만족스럽지 않지만 시계열 분석을 시도해보았으며 나아가 LSTM을 적용하여 분석을 시도해 보려한다



앞으로 시도해볼 작업은 결측치 처리 작업으로 다음과 같다

- Deptments 데이터를 코사인 유사도를 구하여 군집화
- 2010 ~ 2012데이터간 유사도를 파악하여 계절성이 있는지 파악하며 패턴이 있는 데이터와 없는 데이터 구분
- 휴일에 영향을 받는 데이터와 그렇지 않은 데이터 구분
- LSTM을 사용하여 시계열 분석



