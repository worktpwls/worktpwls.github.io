---
title: "vLLM에서 SGLang으로 — 멀티 스텝 RAG에 맞는 서빙 엔진 고르기"
date: 2026-05-15 17:00:00 +0900
categories: [Tech Notes, RAG]
tags: [vLLM, SGLang, LLM-serving, RadixAttention, PagedAttention, KV-cache]
math: false
mermaid: false
---

> TL;DR: 사내 폐쇄망 RAG의 LLM 서빙 엔진을 vLLM에서 SGLang으로 교체했고, 공유 prefix가 빈번한 멀티 스텝 워크로드에서 KV 캐시 공유·구조화 출력 측면의 이점을 얻었습니다.

## 1. 배경 — 멀티 스텝 RAG의 워크로드 특성

사내 폐쇄망에서 운영 중인 RAG 시스템입니다. 검색 → 재순위 → 생성으로 이어지는 멀티 스텝 체인으로 동작하며, 동일한 시스템 프롬프트와 검색 결과 prefix가 단계마다 반복되는 전형적인 공유 prefix 워크로드입니다. 단일 모델 고처리량보다는 체인 전반의 KV 캐시 재사용이 응답 품질과 지연에 더 크게 작용하는 구조입니다.

## 2. vLLM vs SGLang 비교

| 비교 축 | vLLM | SGLang |
|---|---|---|
| KV 캐시 전략 | PagedAttention (페이지 단위 가상 메모리식 관리) | RadixAttention (prefix 트리 기반 공유) |
| 강점 워크로드 | 단일 모델 고처리량 서빙 | 공유 prefix가 빈번한 멀티 스텝 체인 |
| 구조화 출력 | outlines · lm-format-enforcer 외부 의존이 일반적 | xgrammar 내장 |
| 프로그래밍 모델 | OpenAI 호환 서버 중심 | frontend DSL(`@sgl.function`) + OpenAI 호환 |
| 체인 표현력 | 클라이언트 측에서 조립 | 서버 측 DSL로 멀티 스텝 표현 가능 |

선택은 워크로드 특성에 종속됩니다.

## 3. 왜 SGLang을 골랐나

- 멀티 스텝 RAG는 공유 prefix가 빈번 → RadixAttention의 prefix 트리 기반 KV 캐시 공유 구조가 워크로드와 정렬됩니다.
- 구조화 출력(JSON 강제·열거형 응답) 적용 경로가 필요했고, xgrammar 내장이 후속 도입 비용을 낮춥니다.
- `served-model-name`을 vLLM과 동일하게 두어 호출 측(라우터·임베딩·LangChain 체인) 코드 변경 없이 교체했습니다.

## 4. 도입 후 개선 효과

1. **응답 구조화 향상** — 동일 프롬프트에서 표 포맷 활용 빈도가 vLLM 대비 상승합니다. 정형 응답이 더 명료해졌습니다.
2. **공유 prefix 효율** — RadixAttention KV 캐시 공유 덕분에 멀티 스텝 체인의 메모리 점유 패턴에 이점이 관찰됩니다.
3. **구조화 출력 진입로 확보** — xgrammar 내장으로 `guided_choice` · `guided_json` 후속 적용 경로를 확보했습니다. 1차 도입에서는 응답 형식 변화 가능성 때문에 보류했습니다.
4. **운영 격리·롤백 가능성** — 추론 엔진 단위로 격리·롤백이 가능한 구조로 배치했습니다. 엔진 교체·버전 차이가 다른 구성 요소로 번지지 않도록 경계를 분리해, 후속 검토(엔진·모델·구조화 출력) 시 영향 범위를 좁혀 진행할 수 있습니다.

## 5. 검증

- 콜드 호출 **40초** (JIT warmup, 1회만)
- Warm 호출 **0.5~3초** — vLLM 동등 이하
- 동일 입력 기준 출력 **53행 100% 일치**, 추출 회귀 0건

## 6. 짧은 회고

워크로드 특성에 따라 서빙 엔진을 골라야 함을 다시 확인한 사례였습니다. 다음 엔진 검토에서도 KV 캐시 전략·구조화 출력 지원·체인 표현력 세 가지를 첫 비교 축으로 두려고 합니다.
