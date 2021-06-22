# mlflow
- 실험 기록, 프로젝트 관리, 모델 관리 --> 모델 관리에서 뛰어남
- (1) Tracking: Record and query experiments; code, data, config, results
- (2) Projects: Packaging format for reproducible runs on any platform
- (3) Models: General format for sending models to diverse deploy tools
- mlflow = `Model` (모델) + `Model_Registry` (모델 저장소)
- mlflow 설치
  - `pip3 install mlflow==1.15.0`
- `pkill -f gunicorn` 으로 mlflow 죽일 수 있음.
- (1) 실습(`track1`): mlflow Tracking API - 실험 로그 남기기
- (2) 실습(`iris`): Iris ML 모델 저장
  - `sk_iris.py` 모델 저장
  - `sk_iris2.py` 명시적 스키마
  - `sk_iris3_example.py` Input example 포함
  - localhost에 ML 모델 서빙: `mlflow models serve -m iris_rf -p 1234`
  - `ctrl+c` 눌러서 abort 시키고, 
  - 그 뒤 curl 추론(`curl.sh` 내용 붙여넣고 실행) 
- (3) 실습(`tensorflow-mnist model`)
- (4) 실습(`pyfunc`): Add N
  - 모델 여러개 쓰고 싶을 때.
- (5) 실습(`XGBoost Iris`)
  - 모델 서빙: `mlflow models serve -m xgb_mlflow_pyfunc -p 1234`
  - curl 추론까지!

# mlflow Registry
- (1) 실습(`Lesson6 - XGBoost Iris`)
  - 모델 등록: `mlflow server --backend-store-uri sqlite:///sqlite.db --default-artifact-root ~/mlflow`
  - `source mlflow_host.sh`
  - `export MLFLOW_TRACKING_URI=http://localhost:5000`