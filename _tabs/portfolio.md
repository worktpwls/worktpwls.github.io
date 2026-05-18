---
title: Portfolio
icon: fas fa-briefcase
order: 1
---

## About

5년차 응용 AI 엔지니어. 산업·공공 도메인에서 LLM 온프레미스 RAG, 산업용 컴퓨터 비전, AR, 온디바이스 AI를 다룹니다. 학습 데이터 라벨링부터 모델 학습, 후처리, AR 정합, 모바일 배포, SDK화까지 전 사이클을 단일 엔지니어 책임 범위에서 수행한 경험을 보유합니다. 온프레미스·오프라인·엣지 환경처럼 외부 의존을 최소화해야 하는 산업·공공 도메인을 지향합니다.

프로젝트 내역은 아래 Featured Projects를 참조하세요 (최근 → 과거 순).

---

## Core Competencies

### Enterprise RAG · On-Premise LLM Deployment
vLLM·SGLang 비교 평가 후 SGLang 채택, Qdrant 벡터 DB, BGE 임베딩, Gemma 계열 fine-tuning, LangChain 파이프라인, DGX Spark 기반 온프레미스 GPU 인프라 구축·운영. 운영 자동화 스크립트로 모델 핫스왑·로딩·언로딩을 관리하는 MLOps 워크플로 구축.

### Computer Vision for Industrial Inspection
YOLO Detection·Segmentation·Pose Estimation 3종 결합, OpenCV 기반 후처리(Hu-Moment 깊이 추론·기준선 침범 판정), PyTorch 학습 파이프라인, Roboflow·Label Studio 라벨링 운영.

### AR & Mobile AI Engineering
Unity / AR Foundation 기반 Markerless AR, Point Cloud·Anchor를 활용한 3D 모델 정합, Android·iOS 온디바이스 추론, Google Maps API 연동 GPS·공간 매칭.

### SDK Productization & End-to-End Delivery
학습→배포→Unity Package SDK 패키지화 전 사이클. DTA-Core 사례에서 검출 로직을 추상화하여 모델 교체만으로 동일 파이프라인을 재사용 가능하게 SDK화.

### On-Device / Edge AI
PyTorch fine-tuning 후 Native 라이브러리화(Android / iOS), 경량화 모델의 오프라인 추론, 네트워크 복원 시 서버 모델과 통신해 IoU 정합도를 보정하는 오프라인-온라인 보정 로직 구현.

---

## Featured Projects

### Project 1. 삼성 그룹사 RAG 온프레미스 시스템

**기간**: 2026.01 ~ 진행 중
**Client / Context**: 삼성 그룹사 사내 환경. 시스템 디테일은 보안상 비공개.

**Problem**
사내 폐쇄망 정책으로 외부 LLM API 사용이 불가능한 환경에서 사내 자료 검색·요약 자동화 요구. 보안(폐쇄망 준수), 성능(다중 워크로드 동시 처리), 운영(다수 모델 동시 서빙·교체)의 트레이드오프를 동시에 만족시켜야 함.

**Approach**
- Ollama는 초기 테스트·프로토타입 단계에서만 활용하고, 실서빙 후보로 vLLM과 SGLang을 동일 하드웨어·동일 워크로드 조건에서 비교 평가.
- 두 엔진의 KV 캐시 전략 차이를 직접 측정:
  - vLLM은 PagedAttention 기반 KV 캐시 관리로 단일 모델 고처리량 서빙에 강점.
  - SGLang은 RadixAttention 기반 KV 캐시 공유 구조로 구조적 출력(JSON·함수 호출)·복합 프롬프트·멀티턴 워크플로우에 강점.
- 본 프로젝트의 검색-재순위-생성 체인 특성(공유 prefix가 빈번한 멀티 스텝 RAG)에 더 적합하다고 판단하여 SGLang을 메인 서빙 엔진으로 채택.
- 한국 내에서 vLLM과 SGLang을 직접 비교 평가한 사례가 흔치 않은 상황에서, 서빙 엔진별 KV 캐시 전략과 메모리 점유 패턴을 직접 측정하여 워크로드 적합도를 판단.
- Gemma 계열 모델 fine-tuning 후 사내 도메인 평가셋으로 검증.
- 벡터 DB는 Qdrant, 임베딩은 BGE 모델로 사내 문서 인덱싱 파이프라인 구축.
- LangChain 기반 검색·재순위·생성 체인 구성, 검색 컨텍스트 윈도우·청크 전략을 도메인 제약에 맞춰 조정.
- 인프라는 온프레미스 GPU **DGX Spark**. 모델 로딩·언로딩·핫스왑을 운영 자동화.

**System Composition**
- 추론 서빙: vLLM·SGLang 비교 평가 후 SGLang 메인 채택, Ollama는 개발·프로토타이핑 전용
- LLM: Gemma 계열 다종 동시 운영 (개발용 경량 모델 + 배포용 대형 모델)
- 임베딩: BAAI/BGE-M3 (1024차원)
- Reranker: BAAI/BGE-Reranker-v2-M3
- Vector DB: Qdrant, 도메인별 다중 컬렉션 운영
- 검색 전략: 검색 Top-12 → Reranker Top-8 → Agentic 재검색 최대 2 라운드
- 청킹 전략: Parent 2,048 토큰 / Child 128 토큰
- 평가셋: 자체 Golden Dataset 32개 Q&A 구축, 평가 파이프라인 운영 중
- 인프라: 온프레미스 GPU (DGX Spark)
- 코드 규모: Python 약 26,700줄

**Tech Stack**
- LLM Serving: SGLang (메인 채택), vLLM (비교 평가), Ollama (테스트/프로토타입)
- Vector & Embedding: Qdrant, BGE
- Framework: LangChain, Python
- Infra: DGX Spark (On-Premise GPU)

**Role & Ownership**
서빙 엔진 선정·구성, 임베딩·검색 파이프라인 구축, fine-tuning 평가 루프, 운영 자동화 스크립트 작성. 폐쇄망 보안 요건을 시스템 설계 제약으로 변환하여 아키텍처 단계부터 반영.

---

### Project 2. 공공 R&D 기관 교량 검측 AI

**기간**: 2025.05 ~ 2026.02
**Client / Context**: 공공 R&D 기관. 시스템 디테일은 보안상 비공개.

**Problem**
기존 마커 기반 AR은 현장 자유도가 부족하여 교각·구조물 검측 적용이 어려움. 교각 손상 검측이 수작업에 의존, 객체 위치·크기·깊이를 동시에 추정해 3D 모델과 정합시킬 자동화 파이프라인 필요.

**Approach**
- YOLO Detection·Segmentation·Pose Estimation 3종을 결합. Detection으로 객체 후보를 추리고 Segmentation으로 형상 마스크를 확보, Pose로 자세를 추정해 마커 없이 객체를 직접 인식.
- OpenCV Hu-Moment 형상 기술자로 Z축 거리(깊이)를 추론, 단안 카메라 입력만으로 3D 위치 정보 확보.
- Unity AR Foundation + Point Cloud + Anchor 조합으로 3D 모델을 현장 좌표계에 정합·오버레이.
- Label Studio 기반 라벨링 파이프라인을 운영, 데이터 수집·검수·재학습 사이클을 단축.
- **DTA-Core 추상화**: 본 프로젝트의 검출·정합 로직을 범용 Unity Package SDK인 DTA-Core로 추상화. 모델만 학습해 갈아끼우면 동일 파이프라인을 재사용 가능하게 SDK화하여 후속 프로젝트(싱가포르 공공기관·온디바이스 추론)에 재활용.

**System Composition**
- 모델: YOLOv8 기반 Detection·Segmentation·Pose Estimation 3종, Coping 단일 클래스 학습 + COCO 일반 80 클래스 병행
- 후처리: OpenCV Hu-Moment 형상 기술자로 Z축 깊이 추론
- AR 정합: Unity AR Foundation + Point Cloud + Anchor 기반 마커리스 정합
- 입력 해상도: 640×640
- 추론 엔진: Unity AI Inference 2.2.0
- SDK화: DTA-Core (com.ymx.dta-core v0.1.0) Unity Package로 추상화, 모델 교체만으로 동일 파이프라인 재사용
- 아키텍처: MVP + VContainer 의존성 주입
- 타겟 플랫폼: Android (API Level 24+)
- 데이터: Label Studio 기반 라벨링 파이프라인 운영

**Tech Stack**
- ML: PyTorch, YOLO (Detection / Segmentation / Pose), OpenCV
- AR / Client: Unity, AR Foundation, Point Cloud, Android·iOS
- Data: Label Studio

**Role & Ownership**
라벨링 운영, 모델 학습, 후처리(Hu-Moment 깊이 추론), AR 정합, 모바일 배포, SDK 패키지화까지 전 사이클을 소규모 팀에서 핵심 역할로 기여.

---

### Project 3. 싱가포르 공공기관 AI 불법 적치물 검출

**기간**: 2025.02 ~ 2025.08
**Client / Context**: 싱가포르 공공기관. 시스템 디테일은 보안상 비공개.

**Problem**
공공주택 복도에 방치된 불법 적치물의 식별·범칙금 부과 대상 판정 자동화 요구. 적치물 종류·위치뿐 아니라 어느 호수와 관련되는지 매핑까지 동시에 처리해야 함.

**Approach**
- YOLO Segment 모델을 Roboflow 기반 싱가포르 공공기관 환경 특화 데이터셋으로 학습. 복도·벽·문 등 구조물과 적치물을 픽셀 단위로 분리.
- AR Foundation으로 바닥·벽면을 인식, 기준선(통행 가능 폭) 침범 객체를 식별하는 룰 레이어를 구성.
- Google Maps API GPS 위치 + 현장 3D 스캔 데이터 매칭으로 적치물이 위치한 호수를 추론.
- 검출 결과를 웹 서버 리포트 시스템으로 전송, 관리자가 검토·승인하는 워크플로 구축.

**System Composition**
- 모델: YOLOv8n Segmentation, FP16 양자화로 13MB 경량화
- 데이터셋 학습: Roboflow + 싱가포르 공공기관 환경 특화 라벨링
- AR 시각화: Unity AR Foundation으로 바닥·벽면 인식 기반 기준선 침범 자동 판정
- 위치 추적: Google Maps API GPS + 현장 3D 스캔 데이터 매칭으로 호수 정보 추론
- 멀티 플랫폼: Android + iOS 동시 빌드
- ONNX 변환: 5개 변형 (iOS·Sentis·일반 등 플랫폼별 최적화)
- 워크플로: 검출 결과를 웹 서버 리포트로 전송, 관리자 검토·승인 워크플로 구축

**Tech Stack**
- ML: YOLO Segment, Roboflow
- AR / Client: Unity, AR Foundation, OpenCV
- Geo / Backend: Google Maps API, 웹 서버 리포트 시스템

**Role & Ownership**
객체 검출 학습, AR 시각화, GPS·공간 매핑, 서버 연동까지 소규모 팀에서 핵심 역할로 기여.

---

### Project 4. 온디바이스 LLM R&D

**기간**: 2025.10 ~ 진행 중
**Client / Context**: (사내 R&D) — 싱가포르 공공기관 객체 검출 정확도 보정 등 다중 적용 시나리오 대상. 시스템 디테일은 보안상 비공개.

**Problem**
네트워크 없는 현장 환경에서 이미지 추론 필요. 모바일 디바이스의 메모리·연산 자원 제약 아래에서 동작하면서, 네트워크가 복원되면 서버 모델 수준의 정확도까지 끌어올려야 함.

**Approach**
- Pre-trained 모델 + 자체 Labeling Tool로 수집한 데이터로 Fine-tuning.
- 학습된 모델을 Native code 라이브러리(Android / iOS)로 패키지화하여 앱 측에서 직접 호출.
- 경량화 모델을 디바이스에 이식, 네트워크가 없어도 추론이 가능한 오프라인 모드 구현.
- 네트워크 연결 시 기존 서버 모델과 통신하여 IoU 정확도를 보정하는 오프라인-온라인 정합 로직 추가.

**System Composition**
- 학습: Pre-trained 모델 + 자체 Labeling Tool로 수집한 도메인 데이터로 Fine-tuning
- 경량화: 모바일 디바이스 이식 가능 수준으로 최적화 (13MB 수준)
- 네이티브 라이브러리화: Android·iOS Native code로 패키지화하여 앱에서 직접 호출
- 오프라인 추론: 네트워크 없는 현장 환경에서도 동작
- 온라인 보정: 네트워크 연결 시 서버 모델과 통신해 IoU 정합도를 보정하는 오프라인-온라인 정합 로직
- ONNX 변환: 5개 플랫폼별 변형

**Tech Stack**
- ML: PyTorch, Python
- Mobile: Android Native, iOS Native
- OS / Build: Linux, Native 라이브러리 빌드 체인

**Role & Ownership**
Fine-tuning 파이프라인, 자체 Labeling Tool 개발, Native 라이브러리화, 오프라인-온라인 보정 로직 설계에 기여.

---

## Supporting Projects

### 삼성 그룹사 반도체 디지털 트윈
**기간**: 2024.08 ~ 2025.02
삼성 그룹사 반도체 라인을 대상으로 NVIDIA Omniverse·Isaac Sim 기반 디지털 트윈 구성. 물리 시뮬레이션·시각화 파이프라인 구축 기여. 시스템 디테일은 보안상 비공개.

### CAD/BIM Unity 솔루션 다년 경험 (연우피씨엔지니어링 시기)
**기간**: 2021 ~ 2023
UniCAD 3D Viewer, UniCAD Hub, AutoCAD 도면 자동화, PC공법 시공 시뮬레이션을 단일 카드로 묶음. AutoCAD ↔ Unity 데이터 변환, IFC 기반 시공 시뮬레이션, ObjectARX 플러그인 개발, 크로스플랫폼 Avalonia UI 구성 등을 수행. CAD/BIM 도메인 데이터 모델과 Unity 런타임 사이 변환 계층을 다수 구축한 경험.

---

## Tech Stack

```
Expert (실프로젝트 + 트레이드오프 결정 가능):
  Unity, C#, Python, YOLO (Detection / Segmentation / Pose),
  LangChain, RAG 파이프라인, OpenCV

Proficient (실프로젝트 배포 경험):
  vLLM, SGLang, Ollama, Qdrant, BGE, PyTorch,
  AR Foundation, FastAPI, Docker, MySQL, SQLite,
  MLOps (모델 핫스왑·운영 자동화)

Familiar (POC·학습·일부 적용 경험):
  Android Native, iOS Native, NVIDIA Omniverse, Isaac Sim,
  SLAM, ROS2, Label Studio 운영, Roboflow
```

---

## Career

```
와이엠엑스(YMX) · 2024 - 재직 중
  - 응용 AI Engineer
  - 주요 프로젝트: 삼성 그룹사 RAG 온프레미스, 공공 R&D 기관 교량 검측 AI,
    싱가포르 공공기관 객체 검출, 삼성 그룹사 Omniverse, 온디바이스 LLM 추론

연우피씨엔지니어링 · 2021 – 2023
  - Client / Unity Developer
  - 주요 프로젝트: UniCAD 3D Viewer, UniCAD Hub,
    AutoCAD 도면 자동화, PC공법 시뮬레이션
```

---

## Contact / Links

- Email: worktpwls@gmail.com
- GitHub: https://github.com/worktpwls
