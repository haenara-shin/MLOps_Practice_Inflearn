# MLOps Lecture Summary
- Inflearn + Coursera lecture

## ML Pipeline 
- 머신러닝 파이프라인은 새로운 학습 데이터를 수집하는 것으로 시작, 새로 학습된 모델이 어떻게 작동하고 있는지에 대한 피드백(성능 메트릭, 사용자 피드백 등)을 받는 것이 하나의 사이클임.
- 여러 deployment 방법이 있는데, 
  - MLEP 에서는 `fastAPI` 사용해서 간단하게 처리함. (`Deployment`-`server.ipynb`)
  - Inflearn 에서는 `bentoML` 사용
- ![스크린샷 2021-06-22 오전 10 26 24](https://user-images.githubusercontent.com/58493928/122972217-c1f18580-d344-11eb-895c-63e718030805.png)
- ![스크린샷 2021-06-22 오전 10 40 07](https://user-images.githubusercontent.com/58493928/122973594-42fd4c80-d346-11eb-8416-5677b95bda9e.png)


## 단계별 특징
1. `Scoping`: Define project. 해결해야할 문제, 프로젝트 등을 명확히 한다.
2. `Data`
   1. `Data ingestion/versioning`: 데이터 수집 및 버전 관리. 다음 구성 요소가 처리할 수 있는 형식으로 데이터를 처리함. (feature engineering 아님) `Define data`. 들어오는 데이터를 버전 관리하여 데이터 스냅샷을 파이프라인의 끝에 있는 학습된 모델과 연결함.
   2. `Data validation`: 데이터 유효성 검사. 새 모델 버전을 학습 하기 전에 새 데이터의 유효성을 검증함. 새 데이터의 통계(범주의 범위, 범주 수 및 분포)가 예상대로인지 확인함. 모형이 불균형한 훈련 데이터로 훈련하면 지배적인 범주에 치우치는 결과를 얻음. 학습 및 검증 집합으로 분할하는 경우에도 두 데이터 집합 간에 레이블 분할이 거의 동일한지 확인해야함.
   3. `Data preprocessing`: 데이터 전처리. `Label and organize data`
3. `Model`
   1. `Model train`: 모델 학습. 
   2. `Model tuning`: 다수의 모델을 병렬 또는 순차적으로 학습시킬 수 있음. 이를 통해 최종 모델에 적합한 모델 하이퍼파라미터를 선택할 수 있음.
   3. `Model analysis`: 모델의 성능 측정 및 분석. 단순한 성능 측정을 넘어서 학습에 사용되는 검증 집합 보다 더 큰 데이터 집합에서 성능을 계산하는 작업 등을 함. 모델의 예측이 공정함을 확인함. 학습에 사용되는 feature에 대한 모델의 의존도를 조사하고 단일 학습의 feature를 변경할 경우 모델의 예측이 어떻게 변화하는지 살펴볼 수 있음. 데이터 집합을 여러 그룹으로 나눠서 성능을 계산해서 모델이 여러 그룹에 대해 어떻게 작동하는지 조사함. `Perform error analysis`
   4. `Model validation`: 모델 버전 관리 및 검증. 모델/하이퍼파라미터 세트/데이터 세트 선택에 따른 모델의 버전을 관리함.
4. `Deployment`
   1. `Model deployment` (`deploy in production`): 모델 서버를 사용해서 애플리케이션을 재배포 하지 않고도 모델 버전을 업데이트 함. 여러 버전을 동시에 호스팅 하면서 모델의 A/B testing을 한다면 모델 개선 사항에 대한 피드백을 얻을 수 있음.
      - [`A/B testing`](https://brunch.co.kr/@bumgeunsong/17): [`Split testing` or `bucket testing`](https://www.optimizely.com/optimization-glossary/ab-testing/). A버전과 B버전을 무작위로 유저들에게 보여주고 어떤 것이 나은지 실험하는 방법. 각 버전을 본 유저의 행동 데이터를 통계적으로 분석할 수 있음. `가설을 직관이 아니라 데이터로 증명`할 수 있다. (ex: 디자인에 대한 사람들의 선호도) 하지만, 정답을 유추해내는 하나의 단서일 뿐임. `"산을 잘 올라가고 있는지는 말해주지만, 어느 산에 올라가야 하는지 말해주지 않음."`
  2. `Model feedback`: 새로 배포된 모델의 효과와 성능을 측정. 피드백 루프를 반복한다. `Monitor & Maintain system`

## Deployment (`Monitor & Maintain system`) 에서 생길 수 있는 이슈: Drift & Skew
### Drift
- `Data drift`: 모델 성능 저하를 초래하는 입력 데이터의 변경(`x`) (갑작스런 변화. `Suddenly changed`)
- `Concept drift`: 측정하는 가치가 현저하게 변화 (`x -> y`) (`Gradually change`)
### Skew (편향, 왜곡)
- `스키마 편향` (`Schema skew`)
  - 학습 및 서빙 데이터가 동일한 스키마를 따르지 않을때 발생. 
- `특성 편향` (`Feature skew`)
  - 모델이 학습하는 feature 값이 서빙 할때 표시되는 feature 값과 다를때 발생. 
  - Ex: 일부 feature 값이 학습 및 서빙 중간에 수정되거나, 학습과 서빙 시에 feature를 전처리하는 로직이 다르거나, 학습 혹은 서빙 중 하나에만 일부 전처리를 적용하는 경우
- `분포 편향` (`Distribution skew`)
  - 학습 데이터 세트의 분포가 제공 데이터 세트의 분포와 크게 다를때 발생.
  - Ex: 다른 코드 혹은 다른 데이터 소스를 사용하여 학습 데이터 세트를 생성하는 경우, 혹은 학습할 제공 데이터의 '대표적이지 않은' 하위 샘플을 선택하여 샘플링 하는 경우