# Daegu-Traffic-Accident-Damage-Prediction-AI-Competition  
# [DACON] 대구 교통사고 피해 예측 AI 경진대회 [(Link)]([https://dacon.io/competitions/official/236176/overview/description) 
팀 : 국민대 AI빅데이터 (이상준, 이상우, 최준용)  

주최 : 산업통상자원부, 대구광역시

주관 : 한국자동차연구원, 대구디지털혁신진흥원

운영 : 데이콘

규모 : 약 2000여명 참가

===========================================================================

✏️
**대회 후기**
 - ```추론시점에서 알 수 있는 정보가 한정적이라 학습 데이터를 보충하는 과정에서 외부데이터를 많이 사용```
 - 타겟값을 결정짓는 핵심 정보가 마땅히 없어 외부데이터를 다양하게 수집 및 활용하는 것이 대회의 핵심이었다고 생각한다
   
===========================================================================

**핵심 전략**
 - 공공데이터 포털에서 구할 수 있는 train 데이터와 비슷한 형태의 대구 사고 데이터를 수집하여 전처리 후 모델에 학습데이터로 활용 > 모델의 예측 안정성 및 일반화 성능 향상
 - 데이콘 측에서 제공받은 보안등, 어린이 보호구역, 주차장, cctv 외부데이터를 활용하여 여러가지 피처들을 생성 > 모델의 예측 정확도 향상
 - 사망자 수, 중상자 수, 경상자 수, 사고발생 건수를 활용하여 대구 광역시 구 및 동별로 사고 위험도를 학습하게끔 피처 생성
 - 날짜 정보를 활용해 연,월,일,요일,시간피처 및 Cycling 피처등 시간적인 정보를 담은 피처를 생성해 XGBOOST로 학습 > 시계열 정보를 학습할 수 있는 XGBOOST를 사용함으로써 예측 정확도를 향상
 - Validation set을 StratifiedKfold 방식을 사용하여 타겟값이 골고루 분포하게끔 구성 > 모델의 예측 정확도 향상
 - StratifiedKfold 방식을 사용할 때 K 값 즉, 나누는 폴드 수를 다양하게 구성 > 예측의 일반화 성능 향상
 - 모델별로 Seed를 다양하게 구성 > 예측의 일반화 성능 향상

===========================================================================

## 외부데이터

**1. 보안등 데이터**
  * 대구광역시 동별 보안등의 설치개수 sum,mean
  * 대구광역시 동별 보안등의 위도 min,mean,max
  * 대구광역시 동별 보안등의 경도 min,mean,max
  * 대구광역시 동별 보안등의 설치연도 min,mean,max
  * 대구광역시 동별 보안등의 설치형태 nunique

**2. 어린이 보호구역 데이터**
  * 대구광역시 동별 어린이 보호구역의 cctv 설치대수 min,max,mean,sum
  * 대구광역시 동별 어린이 보호구역의 위도 min,mean,max
  * 대구광역시 동별 어린이 보호구역의 경도 min,mean,max
  * 대구광역시 동별 어린이 보호구역의 최소 도로폭 mean
  * 대구광역시 동별 어린이 보호구역의 평균 도로폭 mean
  * 대구광역시 동별 어린이 보호구역의 최대 도로폭 mean
  * 대구광역시 동별 어린이 보호구역 sum

**3. 주차장 데이터**
  * 대구광역시 동별 주차장의 주차구획수 min,mean,max,sum
  * 대구광역시 동별 주차장의 위도 min,mean,max
  * 대구광역시 동별 주차장의 경도 min,mean,max
  * 대구광역시 동별 주차장의 급지구분 별 sum
  * 대구광역시 동별 주차장 sum

**4. CCTV 데이터**
  * 대구광역시 동별 CCTV 단속구분 별 sum
  * 대구광역시 동별 CCTV 제한속도 min,mean,max,median
  * 대구광역시 동별 CCTV 위도 min,mean,max
  * 대구광역시 동별 CCTV 경도 min,mean,max
  * 대구광역시 동별 CCTV 설치연도 min,mean,max
  * 대구광역시 동별 CCTV sum

**5. 공공데이터 포털**
  * 대구광역시의 동별 교통사고 GIS 정보 데이터(2007년~2018년)

===========================================================================
## Process

**1. EDA** 
  * 새벽이외의 시간대(6시~)에는 ECLO값이 낮다가 새벽시간대(0시~5시)에는 높아지는 주기성을 확인함
  * 평일에는 대체적으로 ECLO값이 낮고, 토요일부터 높아져서 일요일에 가장 높은 값을 보이는 것을 확인함
  * 구별로 ECLO값의 평균을 비교해본 결과 특정 구(달성군,동구)에서 다른 구에 비해 높은 사고위험도를 보이는 것을 확인함
  * 동별로 ECLO값의 평균을 비교해본 결과 동별로 편차가 크고 사고위험도가 높은 동들을 분석해본 결과 대부분 근처에 고속도로가 있거나 산간지역에 위치해 있다는 특징을 지님
  * 설치된 CCTV 최대제한속도 및 최소제한속도별로 ECLO값을 비교해보았을 때 대체적으로 제한속도가 높아질수록 사고위험도가 높아지는 것을 확인함
    
**2. Feature Engineering** 
  * 0시부터 23시까지로 구성된 시간 feature를 시간대별로 그룹핑
  * 평일,토요일,일요일을 오름차순으로 label encoding
  * 시간 및 날짜 feature들의 inherently cyclical 하다는 특징을 활용하기 위해 sin/cos 변환
  * 구별 ECLO값의 평균이 5보다 큰 구를 뽑아 그 구에 속하는 지 여부에 대한 파생변수 생성
  * 동별 ECLO값의 평균이 5보다 큰 동을 뽑아 그 동에 속하는 지 여부에 대한 파생변수 생성
  * 동별 사망자,중상자,경상자 및 부상자 수를 sum하는 파생변수 생성
    
**3. Modeling**
  * 범주형 변수로 구성된 공간정보를 활용하기 위해 CatBoost 사용
  * 날짜 및 시간정보를 시계열 정보로 활용하기 위해 XGBoost 사용
  * Overfitting을 방지하기 위해 별도의 파라미터 튜닝은 하지 않음
  * 여러 개의 교차 검증 활용(Multi-StratifiedKFold cross validation)
  * 여러 개의 Seed를 활용
    
**추가 시도 사항** (최종 결과에는 사용하지 않음)
  * 데이콘 측에서 제공해준 대구 빅데이터 마트 데이터(gpkg형식)를 csv형식으로 활용하기 위해 멀티 폴리곤 중심점의 위도, 경도를 추출해 파생변수를 생성 -> 동별 위경도를 CCTV들의 평균 위경도로 대체하여 사용하다보니 부정확한 mapping으로 인해 정확도가 오히려 낮아짐
  * 사망자,중상자,경상자 및 부상자들에 대해 각각 모델링 및 예측하여 ECLO를 도출하는 방식을 사용 -> 주어진 시공간 정보로는 정확한 사망자,중상자,경상자 및 부상자들의 숫자를 예측하지 못하는 한계때문에 정확도가 오히려 낮아짐
  * ECLO 값은 정수형으로 이루어져 있다는 사실에 기반하여 추론값을 반올림 -> 정확도가 오히려 낮아짐
