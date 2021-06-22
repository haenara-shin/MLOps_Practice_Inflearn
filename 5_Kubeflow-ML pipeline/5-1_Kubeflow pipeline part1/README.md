# 개요
- 모든 컴포넌트는 Docker로 가상화 되어 있음. (component-docker-pod에 1:1 매칭)
- 그래프 형태로 파이프라인을 표현함. 분기화 할 수 있음. 
- `import kfp.dsl as dsl`

# 실습 (kubeflow 8080 포트에 이미 띄운 상태 가정)
- (1) Lesson2_hello_world 이동
- (2) Add: 컴포넌트1 + 컴포넌트2
- (3) Parallel: 동시에 2개를 띄운 다음에 붙임(합침)
- (4) Control: 동전 던지기(앞면 나올때의 행동과 뒷면 나올때의 행동을 다르게 함.)
