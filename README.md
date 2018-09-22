# Walmart Recruiting - Store Sales Forecasting
https://www.kaggle.com/c/walmart-recruiting-store-sales-forecasting
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



