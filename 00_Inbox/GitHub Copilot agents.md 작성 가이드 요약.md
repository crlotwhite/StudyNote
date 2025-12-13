# GitHub Copilot `agents.md` 작성 가이드 요약 (2,500+ 레포 분석 기반) + CMake C++용 템플릿 세트

이 문서는 GitHub Blog 글(“How to write a great agents.md…”)의 핵심 내용을 이해하기 쉽게 정리하고, 실제로 바로 적용 가능한 **CMake 기반 C++ 레포용 에이전트 3종 세트**(`docs-agent`, `test-agent`, `lint-agent`)를 포함합니다.

---

## 1. `agents.md`란 무엇인가?

`agents.md`(또는 `.github/agents/*.md`)는 GitHub Copilot에서 사용할 **전문화된 에이전트(Agents)**를 정의하는 문서입니다.  
목표는 “범용 조수”가 아니라, 레포의 규칙과 워크플로우에 맞춰 **역할·권한·지식·명령·금지사항이 명확한 전문가 에이전트 팀**을 만드는 것입니다.

예:
- `@docs-agent`: 문서 작성/정리 담당
- `@test-agent`: 테스트 추가/보강 담당
- `@lint-agent`: 포맷/린트만 담당
- (확장) `@security-agent`, `@api-agent`, `@dev-deploy-agent` 등

---

## 2. 잘 되는 `agents.md`의 공통 특징(핵심 원리)

### 2.1 명령어(Commands)를 문서 앞쪽에 두기
- 빌드/테스트/린트 커맨드를 정확한 형태로 제공
- 옵션/플래그를 포함해 “복사-실행 가능”하게 작성  
예:
- `cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo`
- `cmake --build build -j`
- `ctest --test-dir build --output-on-failure`

### 2.2 설명보다 코드 예시(Examples) 우선
- 긴 설명보다 “좋은 예 / 나쁜 예” 코드 스니펫이 더 효과적
- 스타일, 테스트 구조, 네이밍 규칙은 예시로 고정하는 편이 안정적

### 2.3 경계(Boundaries)를 명확히: Always / Ask first / Never
- 에이전트가 “어디까지 할 수 있는지”를 강하게 제한할수록 결과 품질과 안전성이 올라감
- 특히 다음을 명확히:
  - **쓰기 가능한 경로(WRITE)**
  - **읽기 전용 경로(READ ONLY)**
  - **절대 금지(Never)**: secrets/토큰 커밋, third_party 수정, 테스트 삭제로 CI 통과 등

### 2.4 스택을 구체적으로 명시
- “C++ 프로젝트”가 아니라:
  - “CMake, C++ 표준(예: C++20), 테스트 프레임워크(googletest/catch2), 포맷(clang-format), 린트(clang-tidy)” 같이 구체화

### 2.5 여섯 영역을 커버하면 안정화된다
- Commands, Testing, Project Structure, Code Style, Git Workflow, Boundaries

---

## 3. 추천 운영 방식(실전 팁)

1) **작고 단일한 책임의 에이전트부터 시작**
- 문서, 테스트, 린트처럼 “안전하고 반복되는 업무”를 우선 분리

2) **실행 루프(검증 루프)를 에이전트에게 쥐여주기**
- 커맨드가 없으면 검증 없이 추측으로 끝날 확률이 증가

3) **실수/사고가 나면 규칙을 문서에 추가**
- 에이전트 문서는 “한 번 쓰고 끝”이 아니라 운영하면서 진화하는 룰북

---

## 4. 적용 권장 파일 구조

```text
agents.md
.github/
  agents/
    docs-agent.md
    test-agent.md
    lint-agent.md
