# GitHub Actions CI Practice

[![Python CI](https://github.com/shbong06-sketch/ci_practice_repo/actions/workflows/ci.yml/badge.svg)](https://github.com/shbong06-sketch/ci_practice_repo/actions/workflows/ci.yml)

Python과 pytest를 이용해 GitHub Actions의 기본 CI 흐름을 연습한 저장소다.  
작업 브랜치 개발부터 Pull Request 자동 검사, 의도적인 테스트 실패와 수정, development에서 main으로의 릴리스까지 실습했다.

## 실습 목표

- pytest를 이용한 단위 테스트 작성
- Pull Request와 Push 이벤트에서 CI 자동 실행
- CI 실패 로그 확인 및 같은 PR에서 수정
- main과 development 보호 브랜치 운영
- 작업 브랜치에서 development로 병합
- development에서 main으로 릴리스
- 버전 태그를 이용한 릴리스 기록

## 구현 내용

calculator.py는 다음 기능을 제공한다.

- add(a, b): 두 값의 합 반환
- divide(a, b): 두 값의 나눗셈 결과 반환
- divide(a, 0): ValueError 발생

pytest로 다음 동작을 검증한다.

| 테스트 | 검증 내용 |
| --- | --- |
| test_add | 덧셈 결과 |
| test_divide | 나눗셈 실수 결과 |
| test_divide_by_zero | 0으로 나눌 때 예외 발생 |

## 프로젝트 구조

~~~text
.
├── .github/
│   └── workflows/
│       └── ci.yml
├── tests/
│   └── test_calculator.py
├── .gitignore
├── calculator.py
├── pytest.ini
├── requirements-dev.txt
└── README.md
~~~

## 로컬 실행

### 1. 저장소 복제

~~~bash
git clone https://github.com/shbong06-sketch/ci_practice_repo.git
cd ci_practice_repo
~~~

### 2. 가상환경 생성 및 활성화

~~~bash
python3 -m venv .venv
source .venv/bin/activate
~~~

### 3. 개발 의존성 설치

~~~bash
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
~~~

### 4. 테스트 실행

~~~bash
python -m pytest
~~~

정상 실행 시 단위 테스트 3건이 통과한다.

~~~text
3 passed
~~~

## GitHub Actions CI

Workflow 파일은 .github/workflows/ci.yml에서 관리한다.

### 실행 조건

- development 또는 main을 대상으로 하는 Pull Request
- development 또는 main에 반영된 Push

### 실행 과정

~~~text
저장소 Checkout
→ Python 3.12 설정
→ pip 의존성 Cache
→ 개발 의존성 설치
→ pytest 실행
→ 성공·실패 결과 반환
~~~

Workflow 권한은 저장소 내용을 읽을 수 있는 contents: read로 제한한다.

## CI 실패 및 복구 실습

test/ci-failure-demo 브랜치에서 덧셈의 예상값을 의도적으로 잘못 설정해 CI 실패를 발생시킨 뒤, 같은 Pull Request에 수정 Commit을 Push해 CI가 다시 통과하는 과정을 확인했다.

| 단계 | 결과 |
| --- | --- |
| 잘못된 테스트 Push | [CI 실패 확인](https://github.com/shbong06-sketch/ci_practice_repo/actions/runs/33288585194) |
| 기대값 수정 후 Push | [CI 성공 확인](https://github.com/shbong06-sketch/ci_practice_repo/actions/runs/33288645941) |
| development 병합 | [CI 성공 확인](https://github.com/shbong06-sketch/ci_practice_repo/actions/runs/33288660267) |
| main 릴리스 병합 | [CI 성공 확인](https://github.com/shbong06-sketch/ci_practice_repo/actions/runs/33289091687) |

## 브랜치 및 PR 흐름

~~~text
feature/* ─┐
fix/*     ─┼→ development → main
test/*    ─┤
docs/*    ─┘
~~~

- 작업은 development에서 분기한 작업 브랜치에서 진행한다.
- main과 development에는 직접 Push하지 않는다.
- 모든 변경은 Pull Request와 CI 검사를 거친다.
- CI가 실패한 Pull Request는 병합하지 않는다.
- 병합은 Create a merge commit 방식으로 진행한다.
- 병합된 작업 브랜치는 삭제한다.

이번 실습에서는 다음 Pull Request를 진행했다.

| PR | 내용 | 병합 대상 |
| --- | --- | --- |
| [#1](https://github.com/shbong06-sketch/ci_practice_repo/pull/1) | 계산기, pytest, CI Workflow 추가 | development |
| [#2](https://github.com/shbong06-sketch/ci_practice_repo/pull/2) | CI 실패 및 수정 과정 실습 | development |
| [#3](https://github.com/shbong06-sketch/ci_practice_repo/pull/3) | v0.1.0 릴리스 | main |

## 릴리스

- 현재 버전: v0.1.0
- 릴리스 흐름: development → main
- main 병합 후 Git Tag로 버전을 기록한다.

## ROS2 환경에서 실행할 때

ROS2 Jazzy가 활성화된 터미널에서는 pytest가 launch_testing 플러그인을 자동으로 불러올 수 있다. 일반 Python 테스트만 실행할 때 ROS2의 PYTHONPATH로 인해 충돌하면 다음 명령을 사용한다.

~~~bash
env -u PYTHONPATH python -m pytest
~~~

이 명령은 해당 실행에 한해 ROS2 Python 경로를 제외하며 현재 셸의 환경변수는 변경하지 않는다.

## 학습 결과

- 로컬 테스트와 CI 테스트의 차이를 확인했다.
- GitHub Actions의 Event, Job, Runner, Step 구조를 적용했다.
- Pull Request 생성 및 변경 시 pytest가 자동 실행되도록 구성했다.
- 실패한 CI 로그를 확인하고 수정 Commit으로 복구했다.
- 보호 브랜치와 Release PR을 이용한 Git 흐름을 실습했다.
- v0.1.0 태그로 릴리스 상태를 기록했다.

## License

이 프로젝트는 [Apache License 2.0](LICENSE)을 따른다.
