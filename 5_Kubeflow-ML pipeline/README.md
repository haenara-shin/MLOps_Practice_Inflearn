# Kubeflow 개요
- 머신러닝 파이프라인을 추상화하고 관리할 수 있게끔 하는 오픈소스.
- 조합가능성(composability, 각 단계를 파이프 라인으로 묶어서 관리) + 이식성(portability, 컨테이너 기반이고 쿠버네티스 위 어느 랩탑/시스템에도 배포) + 확장성(scalability, 쿠버네티스 기반이라 컨테이너 개수 조절 가능)
- ML tools, libraries 모두 갖고 있음
# Kubeflow 기본 개념
- 센트럴 대시보드 존재
  - 파이프라인 등록, 실행, 각 컴포넌트 실행 체크 등.
  - Jupyterhub: 프로토타이핑과 실험
  - kubeflow(Argo): 모든 것이 컨테이너 기반. 파이프라인 오케스트레이션을 수행.
- 파이썬 함수 위에 데코레이터(`@kfp.dsl.python_component`)를 붙여서 파이프라인 컴포넌트로 만든다. 
- DSL decorator가 적힌 파이썬 함수를 도커 이미지로 패키징 한다. (`@kfp.compiler.build_python_component`)
- 마지막으로 해당 kubeflow 컴포넌트를 파이프라인으로 패키징 한다. (`@kfp.dsl.pipeline`)
- 파이프라인 구성을 완료하면 해당 파이프라인을 yaml 형태로 패키징 할 수 있고("파이프라인을 컴파일" 한다), 그리고 이 yaml 파일을 kubeflow 대시보드에 업로드하거나 `kfctl` 명령으로 업로드 할 수 있다.
- Katlib
  - 모델튜닝(AutoML)을 담당하는 모듈.
  - 기본적인 HPO 도구들 추가 중.
- TFJobs: 비동기로 학습하거나 오프라인 추론 할 때 사용.
- KFServing: 온라인 인퍼런스 서버를 KFServing으로 배포할 수 있음. (or TFServing)
- MinIO: 파이프라인 간의 저장소 기능을 MinIO 솔루션으로 구성함. MinIO에는 파이프라인 중간에 생기는 부산물들을 저장할 수 있음.

# Kubeflow 설치
- 도커, 쿠버네티스 설치/설정 완료 후,
- 터미널에 붙여넣고 실행 \
`export PLATFORM=$(uname) # Either Linux or Darwin
export KUBEFLOW_TAG=1.0.0 KUBEFLOW_BASE="https://api.github.com/repos/kubeflow/kfctl/releases" # Or just go to https://github.com/kubeflow/kfctl/releases
wget https://github.com/kubeflow/kfctl/releases/download/v1.0/kfctl_v1.0-0-g94c35cf_darwin.tar.gz KFCTL_FILE=${KFCTL_URL##*/}
tar -xvf "${KFCTL_FILE}"
sudo mv ./kfctl /usr/local/bin/
rm "${KFCTL_FILE}"`
- 완료되면 다시 아래 문구 실행, \
`export MANIFEST_BRANCH=${MANIFEST_BRANCH:-v1.0-branch} export MANIFEST_VERSION=${MANIFEST_VERSION:-v1.0.0}
export KF_PROJECT_NAME=${KF_PROJECT_NAME:-hello-kf} mkdir "${KF_PROJECT_NAME}"
pushd "${KF_PROJECT_NAME}"
manifest_root=https://raw.githubusercontent.com/kubeflow/manifests/ FILE_NAME=kfctl_k8s_istio.${MANIFEST_VERSION}.yaml KFDEF=${manifest_root}${MANIFEST_BRANCH}/kfdef/${FILE_NAME} kfctl apply -f $KFDEF -V
echo $? popd`
- 잘 되는지 보려면 `kubectl -n kubeflow get pods` 를 다른 터미널에서 실행
- Kubeflow 포트 포워딩: `kubectl port-forward` 명령으로 istio-ingressgateway를 8080 포트에 연결 (`kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80`)
- 설치 버전 문제로 중단함.