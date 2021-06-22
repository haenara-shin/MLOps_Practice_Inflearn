# bentoML
- ML model serving을 구현한 오픈 소스 프로젝트
- `mlflow`와 달리 model serving 에만 집중함.
  - 테스트/배포/통합하기 쉬운 방식으로 모델을 제공하도록 함.

# 실습
- `pip install bentoml`
- (1) 모델 훈련
- (2) BentoService 인스턴스 만들기
- (3) BentoService#pack 으로 학습된 모델 artifact 패키징
- (4) BentoService#save 로 Bento 에 저장

## Lesson1_Iris classifier
- `iris.py`: BentoML을 사용하기 위해 학습된 모델 준비
- `iris_classifier.py`: iris 모델을 제공하기 위해 생성된 최소 예측 서비스
- `@artifact(...)`: 예측 서비스에 필요한 모델 정의
- `@env(infer_pip_packages=True)` 는 예측 서비스에 필요한 종속성 및 환경 설정을 지정. requirements.txt 지정해주는게 더 명확함.
- `@api(...)` 는 예측 서비스에 엑세스 하기 위한 endpoint. 추론 API를 정의
- `iris_service.py`: IrisClassifier 위에서 정의한 예측 서비스 클래스로 학습된 모델을 패키징한 다음, 배포 및 배포를 위한 IrisClassifier 인스턴스를 BentoML 형식으로 디스크에 저장함. (`saved_path = iris_classifier_service.save()`)
  - BentoML은 기본적으로 모든 패키지 모델 파일을 `~/bentoml/repository/{service_name}/{service_version}` 디렉토리에 저장함.
  - YataiService 라는 모델 관리 구성 요소와 함께 제공되며, team 이 웹 UI 및 API를 통해 패키지 모델을 관리하고 액세스 할 수 있는 중앙 허브를 제공함.
- (1) BentoML IrisClassifier 인퍼런스 서버 실행(launch)
  - `bentoml serve IrisClassifier:latest`
- (2) curl 로 IrisClassifier 인퍼런스 작업을 요청 (`curl -i \
--header "Content-Type: application/json" \ --request POST \
--data '[[5.1, 3.5, 1.4, 0.2]]' \ http://localhost:5000/predict`)
- (3) predict 요청 (`bentoml run IrisClassifier:latest predict --input '[[5.1, 3.5, 1.4, 0.2]]'`)
- (4) BentoML IrisClassifier Docker 컨테이너화
  - `bentoml containerize IrisClassifier:latest -t iris-classifier`
- (5) `http://localhost:5000` 으로 접속하면 Swagger로 추론 API의 spec.을 확인할 수 있음.

# 적응형 micro batch
- 마이크로 배치가 중요한 이유: 텐서플로 모델을 서빙하는 동안, 개별 모델 추론 요청을 모아서(배치 단위) 한 번에 처리하는 것이 성능에 좋음. 특히 GPU 같은 하드웨어 가속기가 제공하는 높은 처리량을 활용하려면 일괄 처리가 필요.
- `인바운드 요청` : 사용자 클라이언트의 요청
- `아웃바운드 요청`:업스트림 모델 서버에 대한 요청
- `mb_max_batch_size`: 배치의 최대 크기. 이 매개 변수는 처리량 / 대기 시간 상충 관계를 제어하고, 일부 리소스 제약을 초과하는 너무 큰 배치 (예 : 배치의 데이터를 보관 하기 위한 GPU 메모리)를 방지함. 기본값 : 1000.
- `mb_max_latency`: 서비스의 지연 시간 목표 (millisecond)입니다. 기본값 : 10000.
- `아웃바운드 세마포어`: 세마포어는 병렬 처리 정도, 즉 동시에 처리 되는 최대 배치수를 나타냄. 동일한 수의 모델 서버 작업자로 동시 락 서비스를 시작할 때 자동으로 설정됨.
- `예상시간`: 모델 서버가 배치를 실행하는데 걸리는 예상 시간. 대기열의 기록 데이터 및 현재 배치 크기에서 유추.
-  최적의 효율성을 달성하기 위해 `CORK 디스패처`는 인바운드 요청을 `cork/release` 하는 적응형 제어를 수행함. `릴리스`는 다음과 같은 경우에 발생
   -  (다음 조건 중 하나를 충족).
      - `waited time` + `estimated time`이 `mb_max_latency`를 초과할 때 
      - `다음 인바운드 요청`을 기다릴 필요가 없을 때
   - `아웃바운드 세마포어`가 잠겨 있지 않을 때.
- 처리량 및 지연 시간은 API 서버에 가장 큰 영향을 미침.
- BentoML은 다음과 같이 자동으로 배치를 미세 조정함 (우선 순위에 따라 정렬).
  - 1순위: 사용자 정의 제약 조건 `mb_max_batch_size` 및 `mb_max_latency` 을 보장.
  - 2순위: Throughput 최대화
  - 3순위: 평균 Latency Time 최소화
- 주요 디자인 결정 사항과 트레이드 오프
  -  Tensorflow Serving과는 달리, BentoML은 자동적으로 wait timeout, batch size을 조정하여 최대의 처리량과 최소의 응답시간을 찾음.
  -   만약 RAM, GPU가 처리할 수 있는 배치사이즈가 100이라면, mb_max_batch_size는 100으로 설정. 만약 API를 사용하는 client의 timeout이 200ms라면, mb_max_latency는 200을 설정. 만약 모델의 속도가 100ms 정도로 느리다면, mb_max_latency를 10*100ms 정도로 설정.
     -  `class MovieReviewService(bentoml.BentoService): @bentoml.api(
input=DataframeInput(), mb_max_latency=10000, mb_max_batch_size=1000, batch=True)
def predict(self, inputs): pass`

# TF2 이미지 분류 서빙 실습
- FashionMnistTensorflow 인퍼런스 서버를 실행(`bentoml serve FashionMnistTensorflow:latest`)
- curl로 FashionMnistTensorflow 인퍼런스 작업을 요청(`echo '{"instances":[{"b64":"'$(base64 test.png)'"}]}' > test.json`) (`curl -X POST http://localhost:5000/predict -d @test.json --header "Content-Type: application/json"`)