## 0. 개요
- 목표: Ghidra/IDA/JADX/ILSpy 출력과 ASM/IL/bytecode를 입력으로 받아, C/C#/Java pseudo-code와 자연어 설명을 생성하는 역공학 특화 LLM POC
- 마감: 2026-03-31 POC (초기 목표, 추후 조정 가능)

## 1. 로드맵
- [ ] 베이스 모델 선정 및 실험 환경 확정
  - 후보: Qwen2.5-Coder-7B-Instruct (1차), 이후 DeepSeek-Coder-V2-Lite·gpt-oss-20B 등 확장
  - Colab + QLoRA 기준으로 VRAM/세션 전략 정리
- [ ] 데이터셋 분석 및 전처리
  - [ ] `decompile-ghidra-100k` 구조 파악 (instruction=Ghidra pseudo-C, output=clean C)
  - [ ] Instruct 스타일 프롬프트 포맷 설계 (예: [Task] asm_to_c, [TOOL=ghidra][LANG=C] 등)
  - [ ] 향후 multi-tool 확장을 고려한 공통 스키마 설계
    - 필드 예시: tool, input_type(asm/pseudo_c/il/bytecode), lang, instruction, output, meta(arch/compiler/opt_level)
- [ ] 1차 파인튜닝 PoC (Ghidra → C 정제 모델)
  - [ ] Transformers + PEFT(QLoRA) 기반 학습 스크립트 작성
  - [ ] Colab에서 7B 모델 + decompile-ghidra-100k로 1~2 epoch 학습
  - [ ] 샘플 함수 단위로 Ghidra 출력 vs 모델 출력 비교·리뷰
- [ ] 평가 및 기준 수립
  - [ ] 최소 기준 정의: 컴파일 가능성, 가독성, Ghidra 대비 개선 포인트
  - [ ] 작은 벤치셋(함수 50~100개) 구성해서 정성 평가 템플릿 마련
- [ ] exe-analyzer-mcp 연동 설계
  - [ ] MCP 툴 I/O 스펙 정의
    - 입력: task/lang/arch/compiler/opt_level/module_path/function_name/disassembly/extra_context(JSON)
    - 출력: pseudo_code, natural_language_summary, quality_score 등
  - [ ] 학습 데이터 포맷과 MCP I/O 스펙을 최대한 동일하게 맞추는 방향으로 조정
- [ ] 최소 기능 구현
  - [ ] Colab/서버에 LLM 추론 엔드포인트 구성 (임시 HTTP 또는 로컬 CLI)
  - [ ] exe-analyzer-mcp에서 함수 디스어셈블 → LLM 호출 → 결과 표시까지 연결
- [ ] 데모 준비
  - [ ] 실제 SVS 관련 바이너리/플러그인 일부를 대상으로 데모 시나리오 작성
  - [ ] “전통 디컴파일러 출력 vs LLM 정제 코드 + 요약” 비교 리포트 준비
  - [ ] 향후 확장 계획(IDA/Hopper/JADX/ILSpy, IL→C#, bytecode→Java, SVS 도메인 지식 주입) 정리

## 2. 회의/AI 대화 로그

### 2025-12-05
- ChatGPT 요약:
  - Claude/GPT 같은 범용 LLM도 일정 수준의 리버스 엔지니어링(의사코드 → C 복원, EXE 분석 설명 등)이 가능하지만, Ghidra/IDA/JADX/ILSpy 등의 디컴파일 출력과 ASM/IL/bytecode를 대상으로 동작하는 **역공학 특화 LLM**을 만들면 정확도·일관성·툴 통합 측면에서 큰 이점이 있다고 정리함.
  - LLM4Decompile이 공개한 `decompile-ghidra-100k` 데이터셋을 활용하여, Ghidra pseudo-C → 사람이 읽기 좋은 C 코드로 정제하는 태스크를 1차 목표로 삼고, 학습 아이디어(ExeBench 기반 C↔ASM 병렬, 컴파일러/옵티마이저 메타 태그, 두 단계 학습 구조)는 참고하되 **베이스 모델은 최신 AR 코드 LLM(Qwen2.5-Coder-7B-Instruct 등)**으로 잡는 것이 좋다는 결론에 도달함.
  - dLLM(확산 기반 LLM)은 학습 루프와 하이퍼파라미터, 툴링, 계산 비용 측면에서 AR LLM보다 난이도가 높으므로, **현재 단계에서는 AR 기반 LLM 파이프라인(데이터 전처리 → QLoRA 파인튜닝 → 평가 → MCP 연동)을 먼저 확실히 다지고, dLLM은 2단계 연구 트랙으로 분리**하는 전략이 합리적이라고 정리함.
  - 최종적으로 이 역공학 LLM을 exe-analyzer-mcp, SVS 엔진 리버스, Obsidian/Git 기반 지식베이스와 연결해 **“SVS/리버스 연구용 내부 플랫폼”**으로 활용하는 방향성이 커리어·연구 양쪽에서 의미 있는 플래그십 프로젝트가 될 수 있다는 인식을 공유함.
