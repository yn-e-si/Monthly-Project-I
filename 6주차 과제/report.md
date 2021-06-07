
## &#128161; Total Process

### &#128204; Data Preprocessing
데이터에 Outlier 들과 NaN value 들이 다수 존재하는 것을 확인했기에 이를 preprocssing 하는 작업부터 진행하였습니다.

<br>

Outlier 들이 존재하기에 전체 데이터프레임에서 mean() 값을 대체하여 NaN value 를
채워줄 경우 이상치의 영향으로 값이 불안정하여 우선적으로 NaN 값을 채워주기 위하여
Outlier 들을 제거하는 과정을 진행하였습니다.

<br>

1. 데이터에 숫자가 아닌 문자열이 들어간 경우들이 존재하기에 이를 없애주는 작업을 진행했습니다.
2. 다음으로 숫자들로만 이루어진 새로운 데이터프레임을 통해서 IQR 값을 계산하여 float 타입으로 이루어진 5 개의 column 들 중에서
outlier 범위에 2개 이상 속하는 인덱스를 추출한 후 해당 인덱스들을 제거하였습니다.

<br>

위의 두 가지 과정을 통해 1. 문자열이 들어간 raw 2. outlier raw 들을 제거한 새로운 데이터프레임을 생성하였습니다.

<br>

이 데이터프레임을 통해 ``Location``, ``Cuisines`` 을 기준으로 ``groupby().mean()`` 을 진행하여
``original dataframe`` 에서 1 번으로 제거시킨 문자열로 이루어진 raw 들에 대하여 해당 column이 
문자열이면서, ``Location, Cuisines`` 가 같은 경우가 있을 경우 해당되는 ``grouby().mean()`` 값으로 대체하였습니다.

<br>

만약 위의 값이 존재하지 않을 경우 해당 column 의 mean() 값으로 대체하였습니다.

<br>

1 번에서 제거시킨 raw 들의 값을 다 채워준 뒤 2 번에서 제거시킨 NaN 값들을 위와 똑같은 과정으로 진행하였습니다.

<br>

다만 원래는 ``Location, Cusisines`` 뿐 아니라 NaN이 존재하는 column 기준으로 해당 column 과의 ``corr`` 을 계산하여
관계가 높은 column 을 추가하여 해당 column 값이 같은 경우도 추가하여 ``mean`` 값을 세분화하여 구하려 했으나
데이터가 ``float`` 이다 보니 ``Location`` 과 ``Cusisines`` 이외의 column 들을 조건으로 추가할 경우 상당수가 부합하지 않아
위의 두 가지 column 에 대해서만 조건을 걸었습니다.

<br>

다음으로 ``Location`` 과 ``Cuisines`` 에 대해서는 ``One-hot encoding`` 을 진행하였습니다.

<br>

마지막으로 columns 과의 값의 범위가 다르기에 ``scaler`` 를 진행하였습니다.

<br>

이 때 총 4가지의 경우로 진행하였습니다.

``One-hot encoding`` 된 모든 ``columns`` 들을 살렸을 때,
-  Robust scaler 사용
-  Minmax scaler 사용

``One-hot encoding`` 된 ``colmns`` 중, ``target value`` 와의 ``corr`` 이 ``0.05`` 이상되는 일부 column 만 선택하여 진행,
-  Robust scaler 사용
-  Minmax scaler 사용

<br>
<br>

### &#128204; Modeling

위의 4 가지의 ``train set`` 을 기준으로 ``LinearRegression, RandomFoestRegressor, DecisionTreeRegressor`` 를 진행하여
``Fine-Tuning`` 을 위한 모델들을 선정하였습니다.


### &#128204; 사용한 손실 함수
- MAE
- MPE
- Underperformance Ratio
- RMSLE

### &#128204; 성능 확인

- LinearRegression 을 제외한 두 가지 모델에선 ``MAE`` 를 보았을 땐 큰 차이가 나지 않았습니다.
- 이에 또 다른 지표로 ``Under_prediction Ratio`` 와 ``MPE`` 를 사용하여 성능을 비교하였습니다.

<br>
<br>

그 결과, 모든 학습데이터 상에서 ``Decision Tree Regressor`` 모델이 성능이 가장 좋음을 알 수있었습니다.

<br>

- 학습 데이터 상에선 ``mae`` 같은 경우는 큰 차이가 나지 않았습니다.
- 하지만 ``mpe`` 를 비교하였을 땐 일부만 사용한 학습데이터가 더 크게 빗나감을 확인하였습니다.
- 마지막으로 ``ratio`` 를 보았을 땐 일부만 사용한 학습데이터가 더 좋음을 확인하였습니다.

<br>

따라서 결론은 다음과 같습니다.

<br>

<b>
MPE 를 보았을 때 모든 학습데이터에서 전반적으로 overperformance 를 보임을 알 수 있습니다.
</b>

<br>
<br>

또한 ``One-hot encoding`` 의 전체 데이터를 학습하는 것은 ``MPE`` 관점에선 일부 데이터를 학습하는 경우보다 0 에 더 가까우므로 예측의 정확성은 더 높다라고 할 수 있습니다.
하지만 ``RATIO`` 관점에서 보면 일부 데이터를 학습하는 경우가 Under-prediction 의 비율이 더 낮음을 확인할 수 있습니다.

<br>

정리하자면 예측 정확도는 전체 데이터를 학습하는 경우가 유리하나 이는 3~4 정도의 작은 차이가 존재합니다. 일부 데이터를 학습하는 경우는 ratio 비율이 1.5 % 정도 더 낮게 관측이 됩니다.

<br>

두 경우 모두 작은 차이지만, 예측의 정확도 보다 ``under-prediction`` 의 비율을 줄이는 것이
조금 더 중요하다 하였으므로 최종적인 학습데이터는 일부 데이터를 학습하는 경우로 선정하였습니다.
또한 그 중에서도 편차의 값이 더 작은 ``minmax -scaler`` 를 사용한 경우를 채택하였습니다.

<br>
<br>

- 최종 모델 : DecisionTreeRegressor
- 최종 학습데이터: One-hot encoding 중 일부 데이터 및 minmax-scaler 사용



<br>
<br>

### &#128204; Fine-Tune

최종적인 모델에 대하여 ``Rmsle`` 값과 ``Under-prediction Ratio`` 을 최적화하는 파라미터를 ``gridsearchcv`` 을 통하여 선정하였습니다.

``gridsearchcv`` 를 ``rmsle`` 와 ``underperformance_ratio`` 를 사용하여 최적화를 시켜보았을 때, ``underperfomance_ratio`` 를 사용했을 때가 전체적으로 성능이 더 좋았습니다.

### &#128204; Result

따라서 최종적인 모델과 파라미터에 따른 결과는 다음과 같습니다.


- ``MAE``: $5.050480769230769$
- ``MPE``: $3.3260909763313604$
- ``RMSLE``: $0.2415444755985762$
- ``Under-prediction의 비율``: $19.231$




