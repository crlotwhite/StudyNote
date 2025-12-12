## 0. 개요
- 목표: JUCE 기반 폴리 서브트랙티브 신디사이저(VST3) MVP 구현 + (후속) 로컬 LLM(Gemma 등)으로 프리셋/패치 생성·변주 기능 연동
- 범위:
  - 1차(MVP): 플러그인 자체(오디오 엔진/파라미터/프리셋/기본 UI) 완성
  - 2차(확장): LM Studio/Ollama 또는 llama.cpp 기반 로컬 LLM을 통해 “텍스트 → 프리셋(JSON)” 적용
- 마감: (자체 스프린트 기준) 2~4주 내 MVP, 4~8주 내 LLM 연동 데모

## 1. 로드맵
- [ ] 요구사항 정리
  - [ ] 신스 컨셉 확정: “텍스트로 프리셋을 설계하고, Macro 6개로 연주/자동화하는 폴리 신스”
  - [ ] MVP 모듈 스펙 확정(osc/filter/env/lfo/fx/preset)
  - [ ] 파라미터 범위/정규화(0~1) 규약 정의 및 스키마 초안 작성(JSON Patch)
- [ ] JUCE/CMake 프로젝트 스캐폴딩
  - [ ] CMake + JUCE FetchContent 기반 플러그인(VST3) 템플릿 구성
  - [ ] APVTS(AudioProcessorValueTreeState) 파라미터 트리 구성
  - [ ] 기본 UI 레이아웃(Macro 상단, 상세 파라미터 그룹화)
- [ ] 최소 기능 구현(“소리 나는 악기” 우선)
  - [ ] Mono Voice: Osc1/2(+Noise) → Filter(SVF) → Amp ADSR
  - [ ] Poly 확장: voice allocator(기본 8~16 voice), note on/off 안정화
  - [ ] Filter ADSR + LFO1 및 최소 Mod routing(LFO→cutoff, Env→cutoff)
- [ ] Global FX 및 안정화
  - [ ] Delay(time/feedback/mix), Reverb(size/damp/mix) 추가
  - [ ] 클릭/팝 방지: 파라미터 스무딩(램핑), 프리셋 적용 시 안정화 전략
  - [ ] CPU/안정성 체크(오디오 스레드 I/O 금지, 동적 할당 최소화)
- [ ] Macro 6개 매핑(제품감 확보)
  - [ ] Brightness/Warmth/Punch/Motion/Width/Space 정의
  - [ ] Macro→내부 파라미터(3~6개) 매핑 + 클램프 + 스무딩 적용
- [ ] 프리셋 시스템
  - [ ] APVTS state 저장/복원(DAW 프리셋/세션 호환)
  - [ ] 기본 프리셋 10개 제공(Pad/Lead/Pluck/Bass 포함)
- [ ] (후속) LLM 연동 설계/구현
  - [ ] “PresetPatch” 구조 정의 + JSON 파싱/검증/적용 함수 분리
  - [ ] 로컬 LLM 호출 방식 확정
    - [ ] 우선안: LM Studio(OpenAI 호환 HTTP) 또는 Ollama(REST)로 C++ HTTP 호출
    - [ ] 대안: llama.cpp/gemma.cpp 직접 내장(추후)
    - [ ] ONNX는 필요 시 검토(초기 MVP에서는 보류)
  - [ ] 데모 흐름: 텍스트 입력 → JSON Patch 생성 → 검증/클램프 → Macro/내부 파라미터 적용

## 2. 회의/AI 대화 로그

### 2025-12-13
- ChatGPT 요약:
  - 해커톤 제출 이후 다음 프로젝트로 “JUCE 기반 LLM 신디사이저”가 충분히 가치 있으며, 오디오 엔진은 C++/JUCE가 가장 현실적이라고 정리.
  - TS/Py SDK 의존을 우려했으나, LM Studio/Ollama는 로컬 HTTP API 형태로 붙일 수 있어 C++에서도 문제없음. LLM은 “패치(JSON) 생성”, 오디오는 “실시간 DSP”로 역할을 분리하는 아키텍처를 권장.
  - 컨셉은 폴리 신스가 직관적이며 성공 확률이 높아, “서브트랙티브 폴리 신스 + Macro 6개”를 MVP로 제안.
  - 개발 시작을 위한 상세 프롬프트(요구사항/구조/수용 기준 포함)를 제공했고, 구현 순서는 “모노→폴리→LFO/Env→FX→Macro→프리셋→(후속)LLM”로 권장.
