# 실험관리와 하이퍼 파라미터 최적화
## 실험 관리 - Weight and Bias (WandB)
- 실험 관리(스프레드 시트 바탕으로 grid search 하는것 보다 훨씬 깔끔. TensorBoard 보다 깔끔하고 접근성이 좋음) 및 보고서 작성
- `!pip install wandb -qqq`
- `import wandb`
- `!wandb login` --> 토큰 복붙
- `from wandb.keras import WandbCallback` --> TF callback 함수
- `experiment_name = wandb.util.generate_id()` --> 실험 이름 정하기
- `wandb.init(하이퍼파라미터들, 딕셔너리 타입)`
- `config = wandb.config`
- `wandb.log(하이퍼파라미터들, 딕셔너리 타입)` --> loop 문에 넣는다. 실습 코드 참조
- `wandb.finish()`
- ### [실습코드](https://github.com/wandb/examples/tree/master/examples/wandb-sweeps/sweeps-python)
## 하이퍼 파라미터 최적화(HPO) - Weight and Bias Sweep 
- AutoML 가능
- `sweep_config= {하이퍼파라미터}`
- `sweep_id = wandb.sweep(sweep_config)`
- `wandb.agent(sweep_id, function=train)`
- `Dashboard` 형태로 만들어서 보고를 할 수 있음.
- ### [실습코드](https://github.com/haenara-shin/MLOps_Practice_Inflearn/blob/feature/20210620-main/3_WandB_What-If-Tool/Intro_to_Weights_%26_Biases.ipynb)

# 모델 분석
## What-If-Tool (WIT)
- 훈련된 ML 모델의 동작을 분석하는 시각화 기반 도구
- 3개의 탭으로 구성 (Datapointer Editor, Performance, FEatures)
  - Datapointer Editor: 각각의 데이터 포인트를 시각화 화면에 적용함. 해당 feature를 변경했을때 예측 결과 실시간 확인
  - Performance and Fairness: 각 feature를 슬라이스(구간 별 정확도, 어떤 특정 범주 별 정확도 등) 한 모델의 성능을 확인할 수 있음. 이진 분류 모델의 경우, 각각의 feature에 대한 임계값을(threshold) 다르게 설정했을때 성능의 차이를 다차원으로 확인할 수 있음. Confusion matrix 확인하고 임계값 변경 결과에 따른 결과를 실시간으로 확인할 수 있음. 데이터 분포가 고른지도 확인(TFDV와 비슷).
  - Features: feature distribution 확인 (데이터셋의 특성을 확인, TFDV와 비슷)
- 노트북에서 WitWidget 생성
  - `from witwidget.notebook.visualization from WitConfigBuilder`
  - `from witwidget.notebook.visualization from WitWidget`
  - `config_builder = WitConfigBuilder(text_examples).set_estimator_and_feature_spec(classifier, feature_spec)`
  - `WitWidget(config_builder)`
- `Nearest counterfactual`: L1/L2 거리 기준에 따라서, 선택한 데이터 포인트에 가장 가까운 Counterfactual 을 찾는것. 가장 가까운 `counterfactual`은 다른 추론 결과 또는 다른 분류를 가진 가장 유사한 데이터 포인트임. 
- ### [실습코드 - 이진분류](https://github.com/haenara-shin/MLOps_Practice_Inflearn/blob/feature/20210620-main/3_WandB_What-If-Tool/WIT.ipynb)
- ### [실습코드 - 이미지](https://pair-code.github.io/what-if-tool/demos/image.html)
- ### [실습예제 - 노트북 저장소](https://pair-code.github.io/what-if-tool/explore/#notebook)
