## 0. 개요
- 목표: openvpi/DiffSinger 코드베이스에 Helion 기반 fused kernel을 적용하여 학습·추론 성능 및 메모리 효율을 개선하고, 이를 정량적인 벤치마크와 함께 정리한 포트폴리오용 프로젝트로 완성한다.
- 마감: 1차 Helion POC 2026-03-31 (학습/추론 벤치마크 + 정리 문서 수준)

## 1. 로드맵
- [ ] DiffSinger 코드베이스 분석
  - [ ] 주요 모듈 구조 파악 (Acoustic / Variance / Vocoder / Backbone)
  - [ ] WaveNet, LYNXNet, Transformer, NSF-HiFiGAN 등 핵심 경로 정리 :contentReference[oaicite:0]{index=0}
- [ ] 프로파일링 및 병목 구간 식별
  - [ ] torch.profiler 기반 학습/추론 프로파일링 스크립트 작성
  - [ ] 상위 연산/모듈(Conv block, Attention, Vocoder 등) 병목 후보 선정
- [ ] Fused kernel 적용 타겟 확정
  - [ ] WaveNet ResidualBlock의 gated activation / residual 경로
  - [ ] LYNXNet ConvModule의 SwiGLU + LayerNorm + Conv block
  - [ ] Transformer Self-Attention (Flash Attention / SDPA 활용)
  - [ ] EncSALayer의 Add + LayerNorm, FFN block 등 프리노름/FFN 부분
  - [ ] NSF-HiFiGAN의 Conv + Activation + Residual 경로(추론 중심)
- [ ] Helion 학습 및 실험 환경 구축
  - [ ] Helion 설치 및 간단 예제(matmul, activation 등) 구현
  - [ ] Helion autotune 워크플로우 이해 및 캐시 전략 수립
  - [ ] DiffSinger + Helion 통합 테스트용 최소 예제 작성
- [ ] 1차 Fused kernel POC (ROI 높은 순서)
  - [ ] PyTorch 2.x `scaled_dot_product_attention` 또는 Flash Attention 2 적용
  - [ ] Fused LayerNorm (apex 등)로 EncSALayer/ConvNeXt 프리노름 대체
  - [ ] Helion 기반 SwiGLU fused kernel (LYNXNet 등 적용)
  - [ ] Helion 기반 Gated Activation/Residual fused kernel (WaveNet backbone)
- [ ] 성능 벤치마크 및 분석
  - [ ] 학습: steps/sec, epoch 시간, batch size 한계, GPU 메모리 사용량 비교
  - [ ] 추론: RTF, latency, throughput, vocoder 포함 end-to-end 성능 측정
  - [ ] Helion vs torch.compile vs 기존 eager 모드 비교 리포트 작성
- [ ] 코드 정리 및 문서화
  - [ ] `helion_fused/` 모듈로 fused kernel 및 래퍼 정리
  - [ ] README에 라이선스/기여 범위/벤치마크 결과 명시
  - [ ] DiffSinger 포크 기반 프로젝트 구조 설명 및 설치/실행 가이드 작성
- [ ] 포트폴리오/공개 준비
  - [ ] GitHub 공개(또는 private → 필요 시 공개) 및 태그 정리
  - [ ] Helion + DiffSinger 최적화 기술 노트(블로그/노션/Obsidian용) 작성
  - [ ] LLM/AI 엔지니어 포트폴리오에서의 포지셔닝 문구 정리

## 2. 회의/AI 대화 로그

### 2025-12-07
- ChatGPT 요약:
  - openvpi/DiffSinger는 Apache-2.0 라이선스라서 포크 후 Helion 기반 fused kernel을 적용하고, 성능 최적화 프로젝트로 사용하는 것이 법적으로도, 업계 관행 측면에서도 자연스러운 접근이다 (LICENSE 유지, 출처 및 변경 사항 명시 필요).
  - DiffSinger 내에서 fused kernel 적용 우선 타겟은 WaveNet ResidualBlock의 gated activation, LYNXNet ConvModule의 SwiGLU 및 Conv block, Transformer Self-Attention(Flash Attention/SDPA), EncSALayer의 Add+LayerNorm, FFN block, 그리고 NSF-HiFiGAN의 Conv+Activation+Residual 경로 등으로 정리되었다. ROI 기준으로는 Flash Attention → Fused LayerNorm → SwiGLU → Gated Activation → Vocoder 순이 권장된다. :contentReference[oaicite:1]{index=1}
  - 프로젝트 포지셔닝은 “기존 오픈소스 SVS 프레임워크(DiffSinger)를 기반으로 Helion/Triton을 활용한 커널 수준 성능 최적화 및 시스템 통합을 수행한 사례”로 가져가는 것이 적절하며, 이는 LLM/AI 인프라·컴파일러·성능 최적화 포지션에서 강점이 될 수 있다.
