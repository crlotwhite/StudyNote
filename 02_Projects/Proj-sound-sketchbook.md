# Proj-sound-sketchbook

## 0. 개요
- 목표: 브라우저 캔버스에 그린 낙서를 기반으로, Gemini 3 Pro가 그림을 이해하고 장면 설명/스토리/사운드스케이프(+옵션 영상 콘셉트)를 생성해주는 인터랙티브 멀티모달 창의성 도구 POC
- 마감: Kaggle Gemini 3 해커톤 제출 마감 (2025-12-12 기준)

## 1. 로드맵
- [x] 아이디어/컨셉 확정
  - [x] 타겟 정의: 어린이·일반 사용자·크리에이터 모두가 쉽게 쓸 수 있는 “낙서 → 장면/소리” 도구
  - [x] 심사 기준(영향/기술적 깊이/창의성/프레젠테이션)에 맞춘 메시지 정리
- [x] AI Studio Build 스캐폴딩
  - [x] Build 프롬프트로 React + TS 단일 페이지 앱 생성
  - [x] genAI 클라이언트 초기 설정 및 Gemini 3 Pro 호출 테스트
- [x] 캔버스 기능 구현
  - [x] `<canvas>` 기반 스케치 캔버스 구현 (마우스/터치 드로잉)
  - [x] 색상 팔레트(6~8색), 브러시 크기 조절, Clear 버튼
  - [x] `toDataURL()`로 PNG Base64 추출 및 상위 컴포넌트로 전달
- [x] Gemini 3 장면 분석 파이프라인
  - [x] 시스템 프롬프트 작성 (SceneAnalysis JSON 스키마 포함)
  - [x] 이미지(+옵션 mood/length 힌트) → Gemini 3 Pro 멀티모달 호출
  - [x] JSON 파싱 로직 및 타입 정의(SceneAnalysis, SoundEvent)
  - [x] Scene summary / mood tags / story를 UI에 표시하는 결과 패널 구현
- [x] 사운드 생성 기능
  - [x] `audioService.ts` 설계: `generateSoundscape(scene: SceneAnalysis)` 인터페이스 정의
  - [x] scene.music_prompt / mood_tags / environment / suggested_length_sec 조합해 오디오 모델용 프롬프트 생성
  - [x] AI Studio 내 오디오/뮤직 모델(Lyria 등) 호출 또는 샘플 믹싱(새소리/바람/BGM) 기반 MVP 구현
  - [x] “Generate Soundscape” 버튼 + `<audio controls>` 플레이어 UI 연결
- [ ] (옵션) 이미지/키프레임 또는 짧은 영상 생성
  - [ ] scene.image_prompt 기반으로 이미지 생성 모델(Imagen/Nano Banana) 호출
  - [ ] 생성된 이미지 썸네일을 장면 카드에 표시
  - [ ] 시간 되면 Veo 기반 짧은 영상(3~5초) 생성까지 확장
- [ ] UX/안정화
  - [ ] 로딩 상태(분석/이미지/사운드 단계별) 표시
  - [ ] 에러 처리 및 재시도 UX
  - [ ] 세션 내 장면 카드 갤러리(여러 그림 결과를 아래로 누적)
- [x] 데모 영상 준비
  - [x] 2분 내외 시나리오/스크립트 작성
  - [x] 실제 앱 사용 화면 녹화(서로 다른 스타일의 낙서 2~3개)
  - [x] 자막/간단 설명 텍스트 오버레이
- [x] Kaggle 해커톤 제출
  - [x] 노트북/앱 설명 작성(Overview, How it works, Gemini 3 usage, Future work)
  - [x] 데모 영상 업로드 및 제출 폼 작성

## 2. 회의/AI 대화 로그

### 2025-12-08
- ChatGPT 요약:
  - 해커톤 심사 기준(영향, 기술적 깊이, 창의성, 프레젠테이션)을 검토한 결과, 소수 연구자를 위한 TTS 디버거보다는 일반 대중·크리에이터도 직관적으로 이해할 수 있는 “창의성 도구” 방향이 더 적합하다는 결론.
  - 초기 후보로 AI 보컬 & 스피치 코치, 사운드 스케치북(환경 소리 → 스토리/이미지), 프레젠테이션 스피치 코치 등을 비교했으며, Gemini 3의 멀티모달·오케스트레이션을 가장 잘 보여줄 수 있는 방향으로 “그림 기반 멀티모달 창작 도구”를 선택.
  - 최종 컨셉: 브라우저 캔버스에서 낙서를 그리면, Gemini 3 Pro가 그림을 이해하여 장면 요약, 감정/분위기 태그, 짧은 스토리, 이미지/영상/오디오 모델용 프롬프트를 담은 JSON(SceneAnalysis)으로 반환하고, 프론트엔드가 이를 기반으로 장면 카드와 사운드스케이프를 생성·재생하는 구조.
  - 구현 전략:
    - AI Studio의 Build 기능을 사용해 React + TypeScript 단일 페이지 앱 스캐폴딩.
    - `<SketchCanvas>` 컴포넌트로 색상/브러시/지우개를 가진 캔버스 구현 후 `toDataURL()`로 이미지 추출.
    - `geminiService.analyzeSketch()`에서 Gemini 3 Pro 멀티모달 호출, SceneAnalysis JSON 파싱 및 UI 반영.
    - 이후 `audioService.generateSoundscape(SceneAnalysis)`를 작성하여 music_prompt / mood_tags / suggested_length_sec로 오디오 모델을 호출하거나, 최소한 샘플 믹싱을 통해 짧은 사운드 클립 생성.
  - 추가 계획:
    - 이미지/영상 생성 모델(Imagen/Nano Banana/Veo)을 호출하는 프롬프트를 SceneAnalysis에 포함해, 키프레임 이미지 또는 짧은 영상으로 확장 가능한 비전 제시.
    - 결과적으로 “그림 → 장면 이해 → 프롬프트 설계 → 사운드/이미지/영상까지 이어지는 멀티모달 오케스트레이션”을 통해 Gemini 3 Pro의 고급 기능(다중 모달리티, 추론, 맥락 인식)을 해커톤 데모로 명확하게 드러내는 것을 목표로 함.