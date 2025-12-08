## 0. 개요
- 목표: MCP 서버를 위한 Python 기반 표준 스캐폴드 템플릿(copier-mcp-py) 설계 및 구현  
  - core / mcp / cli 3계층 아키텍처 정립
  - uv + python-sdk `mcp[cli]` 조합을 기본 스택으로 사용하는 템플릿 제공
- 마감:
  - 1차 버전(v0.0.1) 사용 가능 수준: 완료
  - 2차 DX 강화(자동 패키지/툴 생성 스크립트 등): 2026-01-31 (임시)

## 1. 로드맵
- [x] 아키텍처 설계
  - [x] core / mcp / cli 3계층 구조 정의
  - [x] tools / resources / prompts 책임 분리 방식 결정
- [x] 기본 템플릿 구현 (copier-mcp-py v0.0.1)
  - [x] uv 기반 프로젝트 구조 (`src/{{package_name}}`)
  - [x] `pyproject.toml.jinja`에 `mcp[cli]` / scripts 설정
  - [x] 한국어/영어 README 분리 + `README.md.jinja`로 생성 프로젝트용 README 제공
  - [x] `.jinja` 템플릿 suffix 및 `_exclude` 구성
  - [x] git 태그 추가로 copier 경고 제거 (`No git tags found in template; using HEAD as ref`)
- [ ] 실제 MCP 서버 적용 및 피드백 수집
  - [ ] mcp-git-analyzer 차세대 버전 템플릿 적용
  - [ ] exe-analyzer-mcp 템플릿 적용
  - [ ] knowledge-ingester(mcp_knowledge_ingester 계열) 템플릿 적용
- [ ] DX 개선: 개발자용 스캐폴드 기능
  - [ ] Typer 기반 CLI에 `dev` 서브커맨드 추가
    - [ ] `dev new-package <name>`: 새 파이썬 패키지(폴더 + `__init__.py`) 생성
    - [ ] `dev new-tool <name>`: `mcp/tools/<name>.py` 스켈레톤 생성
    - [ ] (옵션) `dev new-resource`, `dev new-prompt` 등 확장
  - [ ] 또는 `tools/` 디렉토리 안에 독립 스크립트로 제공하는 방식 비교 후 결정
- [ ] 문서 및 배포 전략 정리
  - [ ] 템플릿 사용 가이드(예: `copier copy gh:crlotwhite/copier-mcp-py my-mcp-server`)
  - [ ] 템플릿 버전업/변경 시 `copier update` 활용 전략 정리
  - [ ] PyPI/uvx 배포 여부 및 이름 정책 검토

## 2. 회의/AI 대화 로그

### 2025-12-08
- ChatGPT 요약:
  - MCP 서버 개발 시 “mcptool에만 의존하지 않고, 함수 호출/라이브러리로도 활용 가능한 구조”가 필요하다는 문제의식에서 출발.
  - mcp 서버를 단일 스크립트가 아니라 **core / mcp / cli** 세 레이어로 나누는 아키텍처를 설계:
    - `core`: 도메인 로직, 데이터 모델, 스토리지/검색 등 순수 함수 집합
    - `mcp`: FastMCP 기반의 tools, resources, prompts 어댑터 계층
    - `cli`: Typer 기반의 사람/배치용 인터페이스
  - uv를 사용하여 프로젝트를 구성하되, 
    - `mcp-knowledge-ingester` 같은 MCP 실행 명령,
    - `knowledge-ingester` 같은 CLI 명령,
    - `knowledge_ingester` 같은 패키지 이름을 일관성 있게 생성하는 네이밍 규칙을 도출.
  - MCP 공식 python-sdk(`mcp[cli]`) 자체가 FastMCP + CLI를 모두 포함하므로,
    - SDK는 “엔진/프레임워크”
    - copier 템플릿은 “Noel 표 표준 프로젝트 구조/레이아웃” 역할로 포지셔닝.

### 2025-12-09
- ChatGPT 요약:
  - uv, copier, Windows 환경이 섞이면서:
    - uv 프로젝트 루트를 `{{project_slug}}` 디렉토리 아래에 두면
    - `_tasks`에서 `cd {{project_slug}} && uv sync` 같은 꼼수가 필요하고,
    - 셸/플랫폼 차이 때문에 동작이 불안정해지는 문제를 확인.
  - 해결책으로 템플릿 구조를 단순화:
    - 루트 디렉토리 바로 아래를 uv 프로젝트 루트로 사용
    - `src/{{package_name}}`만 패키지 경로로 두고, 상위 `{{project_slug}}` 디렉토리는 제거
    - `project_slug`는 **배포 이름 / 스크립트 이름**, `package_name`은 **import 경로**, 디렉토리 이름은 **copier 대상 경로(dst_path)** 로 역할 분리.
  - README 관련 정리:
    - `_templates_suffix: ".jinja"`를 활용해 `README.md`(템플릿 리포용)와 `README.md.jinja`(생성 프로젝트용)를 공존시키고,
    - 실제 생성 시에는 `.jinja` 버전만 렌더되어 프로젝트에 포함되도록 설계.
  - `.venv`, `uv.lock` 등 환경/캐시 파일은 템플릿에서 제거하고 `_exclude`/`.gitignore`로 관리하는 방향으로 정리.
  - copier의 역할을 재정의:
    - 파일/디렉토리/구조 템플릿 + 변수/네이밍 관리 + 추후 `copier update`용 기반으로 활용
    - uv sync, git init 등 환경 세팅은 최대한 템플릿 외부(README 안내, 별도 스크립트, 혹은 최소한의 `_tasks`)로 분리.
  - 최종적으로 `copier-mcp-py` v0.0.1 템플릿을 완성:
    - git 태그를 추가해 `No git tags found in template; using HEAD as ref` 경고 제거
    - “지금 상태로 바로 MCP 서버를 찍어내어 실전에서 사용할 수 있는” 기본 템플릿 완성.
  - 차기 단계 아이디어:
    - Typer CLI에 `dev` 서브커맨드를 추가해
      - 새 패키지/툴/리소스/프롬프트 스캐폴드 자동 생성
    - 다만 이는 “2단계 DX 향상 기능”으로, 우선은 템플릿을 실제 MCP 서버들에 적용해보며 필요성을 검증한 뒤 도입하기로 결정.
