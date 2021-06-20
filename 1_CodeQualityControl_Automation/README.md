# 리서치 코드 품질 관리 자동화
## Github action, CI, Codeclimate

1. Indent/space check: `black`
   1. `pip install -r requirements.txt`
   2. ex) `black main.py`
2. flake8
3. Type check: `mypy`
   1. `pip install mypy`
   2. 타입 힌트가 잘못 지정된 코드는 다음과 같이 오류가 발생 하므로 확인 후 코드를 수정할 수 있다.
      1. `mypy solution.py`
      2. solution.py:9: error: Incompatible return value type (got "str", expected "int")
4. Continuous Integration (CI)
   1. Github Actions 사용
      1. 실습
         1. Repos. 생성 (init 등의 기본 설정 후, `pip install -r requirements.txt` 통해서 black 등의 패키지 설치)
         2. `.gitignore` 생성: `.coverage`, 그리고 `__pycache__` 넣어줌.
         3. black lint 적용하기 위해서 `.github/workflows`폴더 생성
            1. `lint.yml` 생성: 린트 체크(`Super Linter`) github action 용도
         4. `git add .`, `git commit -a -m "black implementation"`
         5. 브랜치를 하나 더 만들어 준다 (`git checkout -b <feature/20210620-main>`)
            1. `git push --set-upstream origin feature/20210620-main`
         6. 깃헙에서 `pull request`를 한다.
         7. 깃헙에서 `compare&pull request` 체크
            1. Lint Code Base 실행됨. 
            2. 과정 지켜보고 에러나는 곳 확인 
         8. pull request 막기 위해서는,
            1. `settings` - `Branches` - `Branch protection rules` - `*` in `Brach name pattern` - `Require status checks to pass before merging` - `Require branches to be up to date before merging` - 린트 체크 항목 선택 (Main merge 막음)
         9. unit test 및 `coverage report` 생성
            1. 테스트 할 `main_test.py` 생성
            2. `.github/workflows` 밑에 `coverage.yml` 생성
            3. `CODECLIMATE_REPO_TOKEN`을 수정하기 위해서 `codeclimate.com` 이동
               1. `opensource project` - `add repos.`
               2. `Repo Setting` - `Test coverage` - `Test reporter ID` 복사 후 `coverage.yml` 의 `CODECLIMATE_REPO_TOKEN`에 붙여넣기
               3. `codeclimate` 에서 계속 설정 업데이트 해줌
                  1. `Github` - `Summary comments` 활성화 - `Inline issue comments` 활성화 - `Pull requrest status updates` 인스톨 - `Webhook on Github` 인스톨
                  2. `Badges` - `Maintainability Badges` 와 `Test Coverage Badge` 의 마크다운 주소를 복사해서 README.md 에 붙여넣는다
         10. 깃헙의 settings 에 다시 가서 규칙 추가 다시 체크
         11. Squash-merge 한다.
         12. `git checkout main` 으로 메인으로 돌아감.
         13. 주의! `if __name__ == "__main__":` 이곳에 `# pragma: no cover` 삽입해줘야함.