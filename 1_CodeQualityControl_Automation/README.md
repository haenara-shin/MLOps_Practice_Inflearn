# 리서치 코드 품질 관리 자동화

1. Indent/space check
   1. black
2. flake8
3. Type check
   1. mypy
   2. 다음과 같이 설치할 수 있다.
      1. $ pip install mypy
   4. 타입 힌트가 잘못 지정된 코드는 다음과 같이 오류가 발생 하므로 확인 후 코드를 수정할 수 있다.
      1. $ mypy solution.py
      2. solution.py:9: error: Incompatible return value type (got "str", expected "int")
4. Continuous Integration
   1. Github Actions 사용
   2. 실습
      1. Repos 하나 만든다.
      2. 