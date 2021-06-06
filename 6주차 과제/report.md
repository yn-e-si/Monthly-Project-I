
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

총 12 가지 모델 중 가장 성능이 좋은 모델들을 다음과 같습니다.

``One-hot encoding`` 된 모든 ``columns`` 들을 살렸을 때,
-  Robust scaler - RandomForestRegressor
-  Minmax scaler - RandomForestRegressor

위의 두 가지 모델이 ``MAE`` 수치가 가장 낮음으로 성능이 가장 좋습니다.

단 ``DecisionTreeRegressor`` 모델 과 성능면에서 큰 차이가 나지 않았습니다.

이에 또 다른 성능 지표인 ``Under-prediction 의 비율`` 을 확인해 본 결과

``One-hot encoding`` 된 모든 ``columns`` 들을 살렸을 때,
-  Robust scaler - DecisionTreeRegressor
-  Minmax scaler - DecisionTreeRegressor

위의 두 모델이 ``RandomForestRegressor`` 모델보다 비율이 $11\%$ 가량 더 낮았습니다.

<br>

따라서 최종적으로 ``MAE`` 와 ``Under-prediction의 비율`` 모두 고려하였을 때,

가장 성능이 좋은 모델은

``One-hot encoding`` 된 모든 ``columns`` 들을 살렸을 때,
-  Robust scaler - DecisionTreeRegressor

입니다.


<br>
<br>

### &#128204; Fine-Tune

최종적인 모델에 대하여 ``MAE`` 값과 ``Under-prediction의 비율`` 을 최적화하는 파라미터를 ``gridsearchcv`` 을 통하여 선정하였습니다.


### &#128204; Result

최종적인 모델과 파라미터에 따른 ``MAE`` 와 ``Under-prediction의 비율`` 은 다음과 같습니다.

- ``MAE``: $6.76436235749662$
- ``Under-prediction의 비율``: $25.673000000000002$

