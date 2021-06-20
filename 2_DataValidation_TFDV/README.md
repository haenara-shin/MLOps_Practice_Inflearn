# 데이터 검증

## 데이터 검증이 필요한 이유
- ML 시스템에서 데이터로 인한 장애 파악이 어려움.
  - 왜냐하면, 데이터가 잘못 들어와도 예측은 정상적으로 수행되기 때문 --> 잘못된 예측값을 늦게 인지하게 됨.
- TFDV (TensorFlow Data Validation)
  - 기술 통계 보기, 스키마 추론, 이상 항목 확인 및 수정, 데이터 세트의 drift 및 왜곡(스큐, skew) 확인

## 실습을 위한 데이터 세트 다운로드 및 로드 (Google Cloud Storage)
- `import os`
- `import tempfile, urllib, zipfile`
- `BASE_DIR = tempfile.mkdtemp()`
- `DATA_DIR = os.path.join(BASE_DIR, 'data')`
- `OUTPUT_DIR = os.path.join(BASE_DIR, 'chicago_taxi_output')`
- `TRAIN_DATA = os.path.join(DATA_DIR, 'train', 'data.csv')`
- `EVAL_DATA = os.path.join(DATA_DIR, 'eval', 'data.csv')`
- `SERVING_DATA = os.path.join(DATA_DIR, 'serving', 'data.csv')`
- `zip, headers = urllib.request.urlretrieve('https://storage.googleapis.com/artifacts. tfx-oss-public.appspot.com/datasets/chicago_data.zip’)`
- `zipfile.ZipFile(zip).extractall(BASE_DIR)`
- `zipfile.ZipFile(zip).close()`
- `!ls -R {os.path.join(BASE_DIR, 'data')}`

## TFDV 설치 및 로드
- `import tensorflow as tf`
- `print('Installing TensorFlow Data Validation')`
- `!pip install -q tensorflow_data_validation[visualization]`
- `import tensorflow_data_validation as tdfv`
- `print('TFDV version: {}'.format(tdfv.version.__version__))` 아마도 0.27.0 이상

## TFDV - 통계 계산 및 시각화
- 훈련 데이터에 대한 통계 계산
- (내부적으로 TFDV는) Apache Beam의 데이터 병렬 처리 프레임 워크를 사용하여 대규모 데이터 세트에 대한 통계 계산. (잘 모르는 내용인데) TFDV와 더 깊이 통합하려는 애플리케이션의 경우(ex: 데이터 생성 파이프 라인 끝에 통계 생성 연결) API는 통계 생성을 위해 Beam PTransform 도 노출한다.
- `tfdv.generate_statistics_from_csv` 사용
  - `train_stats = tfdv.generate_statistics_from_csv(data_location=TRAIN_DATA)`
  - 계산 및 시각화: `tfdv.visualize_statistics(train_stats)`
- Coursera 강의 내용 추가/업데이트 필요!!

## 스키마 추론과 스키마 환경
- 스키마: ML과 관련된 데이터에 대한 제약 조건을 정의 (SQL의 그것과 같음)
  - Ex: 각 feature의 데이터 유형(숫자형, 범주형), 데이터에 존재하는 빈도, 범주형 feature의 경우 허용되는 값 목록 포함(도메인)
- 스키마 작성은 feature가 많은 데이터 세트의 경우에 많은 시간이 필요하지만, TFDV는 기술 통계를 기반으로 스키마의 초기 버전을 생성하는 방법 제공.
  - `tfdv.infer_schema` : 데이터에 대한 스키마 생성
  - `schema = tfdv.infer_schema(statistics=train_stats)`
  - `tfdv.display_schema` : 추론된 스키마를 표시하여 검토할 수 있게함.
  - `tfdv.display_schema(schema=schema)`
- 평가 데이터의 오류 확인
  - Compute stats for evaluation data
    - `eval_stats = tfdv.generate_statistics_from_csv(data_location=EVAL_DATA`
  - Compare evaluation data with training data
    - `tfdv.visualize_statistics(lhs_statistics=eval_stats, rhs_statistics=train_stats, lhs_name='EVAL_DATASET, rhs_name='TRAIN_DATASET)`
- 평가 데이터의 이상 데이터(anomaly) 확인
  - Check eval data for errors by validating the eval data stats using the previously inferred schema
  - `anomalies = tfdv.validate_statistics(statistics=eval_stats, schema=schema)`
  - `tfdv.display_anomalies(anomalies)`
- 이상 데이터 확인 후 스키마 수정 
    - 평가 데이터에는 company에 대한 새로운 값이 있지만 학습 데이터에는 없는 상황 가정.
    - Payment_Type에 대한 새로운 값이 있는 상황 가정.
  - Relax the minimum fraction of values that must come from the domain for feature `company`
    - `company = tfdv.get_feature(schema, 'company')`
    - `company.distribution_constraints.min_domain_mass = 0.9`
  - Add new value to the domain of feature `payment_type`
    - `payment_type_domain = tfdv.get_domain(schema, 'payment_type')`
    - `payment_type_domain.value.append('Prcard')`
  - Validate eval stats after updating the schema
    - `updated_anomalies = tfdv.validate_statistics(eval_stats, schema)`
    - `tfdv.display_anomalies(updated_anomalies)` --> No amomalies found 출력 되야 함.
- 스키마 환경
  - 본 예제에서는 'SERVING' 데이터 세트를 분리 했으므로 체크 해야함. 
  - `default_environment`, `in_environment`, `not_in_environment` 사용해서 환경 세트와 연관될 수 있음.
    - 예제 데이터 세트에는 `tips` feature는 학습용 라벨로 포함됐지만, serving data 에는 없음. 환경을 지정하지 않으면 예외로 표시됨.
    - `serving_stats = tfdv.generate_statistics_from_csv(SERVING_DATA)`
    - `serving_anomalies = tfdv.validate_statistics(serving_stats, schema)`
    - `tfdv.display_anomalies(serving_anomalies)` --- 이렇게 해서 이상치 확인
    - `options = tfdv.StatsOptions(schema=schema, infer_type_from_schema=True)`
    - `serving_stats = tfdv.generate_statistics_from_csv(SERVNING_DATA, stats_options=options)`
    - `serving_anomalies = tfdv.validate_statistics(serving_stats, schema)`
    - `tfdv.display_anomalies(serving_anomalies)` --- 옵션 지정
    - --- TFDV에 무시하도록 지시 ---
    - All features are by default in both TRAINING and SERVING environments
    - `schema.default_environment.append('TRAINING')`
    - `schema.default_environment.append('SERVING')`
    - Specify that 'tips' feature is not in SERVING environment
    - `tfdv.get_feature(schema, 'tips').not_in_environment.append('SERVING')`
    - `serving_anomalies_with_env = tfdv.validate_statistics(serving_stats, schema, environment='SERVING')`
    - `tfdv.display_anomalies(serving_anomalies_with_env)` --- "No anomalies found" 출력

## 데이터 드리프트 및 스큐
### Drift
- Data drift: 모델 성능 저하를 초래하는 입력 데이터의 변경 (갑작스런 변화. 코로나로 인한 구매 패턴 변홛 등)
- Concept drift: 측정하는 가치가 현저하게 변화 (기술의 진화)
- Drift 감지는 범주형 특성 및 데이터의 연속 범위에 대해 지원
- L-Infinity distance로 drift를 표현하며, drift가 허용 가능한 것보다 높을 때 경고를 받을 수 있도록 임계 거리를 설정할 수 있음. (올바른 범위 설정 노력이 필요)
### Skew (편향, 왜곡)
- TFDV는 데이터에서 (1) 스키마 편향, (2) 특성 편향, (3) 분포 편향 등 3가지 다른 종류의 편향을 감지할 수 있음.
- (1) 스키마 편향 (Schema skew)
  - 학습 및 서빙 데이터가 동일한 스키마를 따르지 않을때 발생. 
  - 앞서 살펴본 환경 필드(environment) 통해 지정해야함.
- (2) 특성 편향 (Feature skew)
  - 모델이 학습하는 feature 값이 서빙 할때 표시되는 feature 값과 다를때 발생. 
  - Ex: 일부 feature 값이 학습 및 서빙 중간에 수정되거나, 학습과 서빙 시에 feature를 전처리하는 로직이 다르거나, 학습 혹은 서빙 중 하나에만 일부 전처리를 적용하는 경우
- (3) 분포 편향 (Distribution skew)
  - 학습 데이터 세트의 분포가 제공 데이터 세트의 분포와 크게 다를때 발생.
  - Ex: 다른 코드 혹은 다른 데이터 소스를 사용하여 학습 데이터 세트를 생성하는 경우, 혹은 학습할 제공 데이터의 '대표적이지 않은' 하위 샘플을 선택하여 샘플링 하는 경우
  - Add skew comparator for 'payment_type' feature.
  - `payment_type = tfdv.get_feature(schema, 'payment_type')`
  - `payment_type.skew_comparator.infinity_norm.threshold = 0.01`
  - Add drift comparator for 'company' feature.
  - `company=tfdv.get_feature(schema, 'company')`
  - `company.drift_comparator.infinity_norm.threshold = 0.001`
  - `skew_anomalies = tfdv.validate_statistics(train_stats, schema,previous_statistics=eval_stats, serving_statistics=serving_stats)`
  - `tfdv.display_anomalies(skew_anomalies)`

### 스키마가 검토되고 선별 된 후 '고정' 상태를 반영하여 파일에 저장
- `from tensorflow.python.lib.io import file_io`
- `from google.protobuf import text_format`
- `file_io.recursive_create_dir(OUTPUT_DIR)`
- `schema_file = os.path.join(OUTPUT_DIR, 'schema.pbxt')`
- `tfdv.write_schema_text(schema, schema_file)`
- `!cat {schema_file}`

[실습 코드 link to Colab](https://colab.research.google.com/drive/192REpT6ygmlcvJZagTFqA8rqfpGxgNZD?usp=sharing)