## **0. 개요**
- 목표: 자연어 처리용 텍스트 DSL runecaster의 Python 기반 EDSL 및 런타임 v0.1 구현
- 마감: 2026-03-31 v0.1 라이브러리 MVP

## **1. 로드맵**
- 핵심 개념/IR 설계
    - Rune / Spell / Caster 의미 모델 정의
    - Rune 카테고리(한글/공백/기호/숫자/이모지 등) 및 태그 구조 설계
    
- Python EDSL 런타임 구현
    - @spell 데코레이터 기반 Spell 정의 방식 구현
    - Caster 파이프라인(.use(...)) 실행 모델 구현
    - 텍스트 → Rune 스트림 변환 유틸 구현
    
- 기본 내장 Spell/Builtin 작성
    - 공백/제어문자 정규화 Spell
    - 간단한 문장 분리/토큰화용 Spell (ko/en 최소 기능)
    - URL/숫자/기호 등 기본 패턴 처리 Spell
    
- G2P/텍스트 유틸 PoC 통합
    - 한국어 G2P 전처리 파이프라인에서 runecaster 활용 PoC
    - 기존/예정 텍스트 유틸리티(정규화, 토큰화 모듈 등)와 연동
    
- 외부 DSL(Lark) 도입 검토
    - 실제 사용 경험 기반으로 .rune/.spell 최소 문법 요소 도출
    - Lark 기반 파서 프로토타입으로 외부 DSL → 내부 IR 변환 PoC
    
- 문서화 및 배포
    - 기본 개념/사용법 문서화 (README, 개념 문서)
    - 예제 스크립트 및 G2P 연동 예제 작성
    - PyPI 배포 및 버전 태그(v0.1.0) 정리

## **2. 회의/AI 대화 로그**

### **2025-12-06**

- ChatGPT 요약:
    - runecaster는 UTF-8 텍스트를 공통 단위인 Rune으로 다루고, Spell(규칙/변환)과 이를 실행하는 Caster(파이프라인 런타임)로 구성되는, NLP 지향 awk 스타일 DSL로 정의.
    - 초기 구현은 **Python 기반 EDSL(Internal DSL)** 로 진행하고, 필요해질 때 **Lark를 이용한 외부 DSL(.rune/.spell 파일) 파서를 추가하는 2단계 전략**으로 가는 것이 현실적이고 확장성도 좋다고 합의.
    - 핵심은 Rune / Spell / Caster의 의미 모델(IR)을 먼저 Python 클래스/타입으로 명확히 정의하고, 이후 어떤 문법이 오더라도 이 IR에 매핑되도록 설계하는 것.
    - G2P 및 텍스트 처리 유틸리티의 **공통 기반 런타임**으로 사용하기 위해, 공백/숫자/기호 정규화, 한글/영문 토큰화, 간단 전처리용 Spell들을 우선 제공하는 방향으로 계획.
    - 장기적으로는 Lark 기반 외부 DSL, MCP/에이전트와의 연동, 언어별(NLP 도메인별) Spell 패키지(runecaster.nlp.ko, runecaster.nlp.en 등)로 확장할 수 있는 구조를 목표로 함.