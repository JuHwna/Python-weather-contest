# 빅데이터 분석을 통한 GS편의점 발주량 추천 시스템
## 1. 공모전명 
 - 2019 날씨 빅데이터 콘테스트
## 2. 주최 및 후원 
 - 기상청, 환경부, 한국기상산업기술원, 한국정보통신진흥협회, Daumsoft, GS25, lalavla
## 3. 기간 
 - 2019.5.20 ~ 7.22
## 4. 공모내용
 - 날씨 빅데이터를 활용한 데이터 분석(유통분야, 자유분야)
 - 제가 속한 팀은 유통분야에 참여하여 gs25와 lalavla 데이터, 날씨 데이터를 활용했습니다.
 
## 5. 사용툴
 - Python, Excel

## 6. 프로젝트에서 맡은 역할
 - EDA 전담, 모델링 일부분

## 7. 공모전 데이터
 - GS25 : 지역구별, 날짜에 따른 상품 판매 갯수, 구매한 사람의 성별, 나이
 
|region01|region02|date|gender|age|category|quantity|
|--------|--------|----|------|---|--------|--------|
|서울|종로구|2016-01-01|F|00~19|라면|7|

- Lalavla : 지역구별, 날짜에 따른 상품 판매 갯수, 구매한 사람의 성별, 나이

|region01|region02|date|gender|age|category|quantity|
|--------|--------|----|------|---|--------|--------|
|서울|종로구|2016-01-01|F|00~19|립스틱|5|

- 기상 데이터: 날짜, 지역구, 기상 관련 데이터

|날짜|지역번호|지역|지역구|최대기온|최대풍속|최소기온|평균기온|평균상대습도|평균풍속|총강수량|
|-----------|-------|--------|--------|--------|---------|-------|-------|-----------|---------|--------|
|2016-01-01|98|경기도|동두천시|6.5|3.8|-6.4|-0.2|74|0.9|0.0|

- 지역별 미세먼지 데이터 : 지점명, 일자, 미세먼지 농도

|지점번호|지점명|일자|미세먼지농도|
|--------|-----|----|-----------|
|90|속초|2016-01-01|32|

- 소셜 데이터 : 음식, 취미, 건강, 맛집 단어로 sns에 검색된 횟수

|번호|날짜|블로그|트위터|기사|전체|
|----|----|-----|-----|----|----|
|1|2016-01-01|36|57|45|28|120


## 8. 프로젝트 진행 상황
### 1) 데이터 EDA
(1) 날씨 데이터 EDA

 - 제공 받은 기상 데이터가 원래 columns 명이 영어였기 때문에 눈으로 보기 쉽게 하기 위해 칼럼별 이름부터 변경한 후 시작했습니다.
~~~
weather_seoul.rename(columns={weather_seoul.columns[0] : '날짜',
                        weather_seoul.columns[1] : '지역번호',
                        weather_seoul.columns[2] : '지역',
                        weather_seoul.columns[3] : '지역구',
                        weather_seoul.columns[4] : '최대기온',
                        weather_seoul.columns[5] : '최대풍속',
                        weather_seoul.columns[6] : '최소기온',
                        weather_seoul.columns[7] : '평균기온',
                        weather_seoul.columns[8] : '평균상대습도',
                        weather_seoul.columns[9] : '평균풍속',
                        weather_seoul.columns[10] : '총강수량'}, inplace=True)
~~~

- 필요없는 지점번호와 지점명을 없앤 후, 결측치 확인 작업에 들어갔습니다.
  
  (msno.bar 사용)
  
![weather outlier](https://user-images.githubusercontent.com/49123169/72800709-5147c480-3c8b-11ea-8c93-a48482b40ceb.PNG)

- 상대습도쪽에 결측치가 꽤 많았고 기상청에서 저 부분을 대체하려고 했습니다. 하지만 기상청에서 다운 받은 자료에서도 저 부분이
  다 결측치인 것을 확인했습니다. 결측치를 삭제해도 되었지만 저희 팀은 살리는 방향으로 잡아서 결측치를 대체할 방법을 찾았습니다.
- 결측치를 바꿀 방법으로 mean 값이 있긴 했지만 선형으로 비례하는 결측값 보간 방법을 사용했습니다.
  결측치 보간 방법으로는 DataFrame 값에 선형으로 비례하는 방식으로 했습니다.
  
 
 
  **선형보간법**
 1. 시계열 자료에서 결측치를 채울 때 사용할 수 있는 방법(이미지에서도 사용한다고 합니다.)
 2. python은 df.interpolate으로 사용할 수 있습니다.
 3. 이론적으로는 알려진 지점의 값 사이(중간)에 위치한 값을 알려진 값으로부터 추정하는 것이라고 합니다.
    >https://darkpgmr.tistory.com/117 이론 참고하세요!

 - 그 후 날짜별로 평균기온, 강수량, 일교차 등을 선 그래프로 그렸습니다.
 
![날짜별평균기온](https://user-images.githubusercontent.com/49123169/72806607-ddacb400-3c98-11ea-8b32-d3a7e0d9a04f.PNG)


(2) GS25 EDA

 - 먼저 전체적인 면을 보는 작업을 진행했습니다.(전체 판매량, 전체 상품판매량 비중 등)
 
 * Python의 경우, 한글을 인식시키려면 글씨체를 한글로 바꿔야 합니다.
 
 ~~~
 import matplotlib as mpl
 import matplotlib.font_manager as fm
 from matplotlib.font_manager import fontManager
 for font in fontManager.ttflist:
    if '원하는 폰트이름' in font.name: #폰트가 없으면 깔으셔야지 적용됩니다. 보통 Mal치면 맑음고딕체는 다 있어서 그걸로 연습해보세요!
        print(font.name)
 plt.rc('font', family='원하는 폰트이름')
 ~~~
 
![전체 판매량](https://user-images.githubusercontent.com/49123169/72809832-c8875380-3c9f-11ea-82e9-806656f7f287.PNG)
![편의점 여성남성비율](https://user-images.githubusercontent.com/49123169/72809833-c8875380-3c9f-11ea-8062-8090a48cd87c.PNG)

- 이후, 좀 세부적으로 들어갔습니다.
  
  (각 상품별 날짜에 따른 매출액, 요일별 매출 등)
  
  ![상품별 매출액](https://user-images.githubusercontent.com/49123169/72810164-5c591f80-3ca0-11ea-8105-62fce9dc6b54.PNG)
![연령별 상품판매량](https://user-images.githubusercontent.com/49123169/72810166-5c591f80-3ca0-11ea-82f1-2a159d76b6a2.PNG)

- 이를 통해 GS25의 데이터의 형태에 대해서 알 수 있었습니다.
- lalavla도 이와 비슷하게 분석했습니다. 하지만 랄라블라 자체 데이터가 너무 별로라서 랄라블라 분석이 좀 난항을 겪었습니다.

  (물론 GS25 데이터도 별로라서...... 분석을 어찌할지 애매하긴 했지만요)
  
### 2) 데이터 전처리
(1) 파생변수 생성
 
 - 파생변수 생성을 위해 강수량을 강수 여부로 바꿔서 YES/NO로 범주화시켰습니다.
 - 일교차라는 변수도 만들었습니다. 일교차는 최대기온-최소기온으로 만들었습니다.

(2) 지역과 품목 군집화
 
 - 지역에 따른 상품별 수요 예측과 같은 식으로 분석을 진행하려고 했습니다. 그런데 막상 지역과 품목을 보게 되니 꽤 많다는 사실을 알 수 있었습니다.
 - 그래서 지역은 일단 서울로 줄인 후, 구 단위도 4개의 군집으로 나누었습니다.(hclusting이나 k-mean 방법이 아니라 매출 추세선을 통해 나눴습니다.)
 - 품목 또한 4개의 군집으로 나눴습니다. 그 중에 우산의 경우 0인 경우가 너무 많아 삭제하고 분석을 진행했습니다.
 
### 3) 모델링

**GS25 매출 극대화를 위한 날씨를 활용한 지역구에 따른 일별 판매량 예측**
 
 - 이런 식으로 목표를 정하고 분석에 들어갔습니다.
 - 이런 목표를 정한 이유는 대한민국 편의점 시장의 전만이 밝지 않고 gs25의 편의점 매출 성장률 역시 계속 감소하고 있다는 것을 볼 수 있었습니다. 그래서 물품 수요 예측을 통해 재고 감소 및 판매를 증가시키고 성장률 또한 높이기 위해 기존의 방식에서 나아가 날씨, 기온, sns 언급량에 따른 상품별 세분화된 판매량 예측이 필요하기 때문입니다.
 - lalavla의 경우, 이야기 끝에 패턴이 행사 기간에만 폭발적으로 증가해서 분석하기 어렵다고 판단하여 포기하고 gs25에만 집중해서 분석을 진행했습니다.

(1) 예측에 사용할 분석 기법

![예측 사용 모형](https://user-images.githubusercontent.com/49123169/72888285-a26dbc00-3d50-11ea-835d-8e82d820707c.PNG)

(2) 첫 번째 모델 - 날씨 Feature만을 사용하여 예측

![날씨 변수만 사용](https://user-images.githubusercontent.com/49123169/72888637-448da400-3d51-11ea-81f8-0560e222b5f3.PNG)


*첫번째 모델 pipeline 입니다.*

- 날씨 변수만을 사용하여 분석을 진행한 결과, 밑에 그림처럼 생각보다 좋지 않는 결과를 나타냈습니다.
- 각 지역 군집별 마스크 판매량 예측 그래프뿐만 아니라 다른 그래프도 좋지 않게 나와 변수 추가하는 방법을 생각해냈습니다.

![첫 모델 그래프](https://user-images.githubusercontent.com/49123169/72888638-45263a80-3d51-11ea-8229-10e2c94c1d22.PNG)

(3) 두 번째 모델 - 날씨 Feature와 전날 판매량을 사용하여 예측

![두번째 모델 파이프라인](https://user-images.githubusercontent.com/49123169/72888639-45263a80-3d51-11ea-8ec2-c123b8b7f246.PNG)

*두번째 모델 pipeline입니다.*

![두번째 모델별 성능](https://user-images.githubusercontent.com/49123169/72888640-45263a80-3d51-11ea-86a3-fca7fc795ea4.PNG)

- 그래프도 예측과 비슷하게 나와서 각 지역별 상품에 따른 모델을 돌려본 결과, 꽤 낮은 rmse 값을 얻을 수 있었습니다.
- 나중에 알고 보니 이게 시계열 분석 방법이라는 것도 있긴 했는데 당시에는 잘 몰라서 해당 방법의 모델을 채택했습니다.
  (자기 상관성을 보긴 했어야 했는데 그 점이 부족했습니다.)

(4) 응용 모델 - 미세먼지 농도, sns 언급량을 사용하여 분위수별 마스크 판매량 
 
 - EDA 때 미세먼지 농도와 SNS 미세먼지 언급량이 밀접한 연관이 있어서 두 변수만으로 간단한 모델을 구현했습니다.
 - 해당 작업을 제가 하지 않아서 잘 모르겠지만 딥러닝 방법을 사용했고 기존보다 더 좋은 결과를 얻었습니다.
 

## 9. 느낀 점
 1. 부족했던 파이썬 능력을 좀 더 올렸습니다. 아직 판다스가 익숙치 않았는데 이 때 꽤 실력이 늘면서 편해졌습니다.
 2. EDA 능력을 올리기 위해 matplotlib과 seaborn을 따로 공부해서 파이썬으로 그래프 그리는데 꽤 익숙해졌습니다.
    (요즘은 plotly도 있어서 이것도 공부해보려고 합니다.)
 3. scikit-learn을 예제만 보고 따라쳤었는데 직접 데이터로 돌려서 DecisionRegression과 randomforest를 해본 뜻 깊은 경험이었습니다.
 4. 아직 딥러닝에 대해서 잘 몰랐기 때문에 공부를 해봐야 되겠다는 계기로 작용했습니다.
    (이 후 화장품 프로젝트를 통해 딥러닝 코드를 짤 수 있게 되었습니다.)
