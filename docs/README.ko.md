<p align="center">
  <a href="../README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.es.md">Español</a> ·
  <a href="README.fr.md">Français</a> ·
  <a href="README.de.md">Deutsch</a> ·
  <a href="README.ja.md">日本語</a> ·
  <b>한국어</b> ·
  <a href="README.pt-BR.md">Português</a> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> 논문을 주세요. 그 결과가 실제로 재현되는지 알려드립니다.

**ReproRun** 은 호환되는 모든 AI 코딩 어시스턴트에서 사용할 수 있는 이식 가능한 **AI 에이전트 스킬** 로, *"어떤 논문이 X를 주장한다"* 에서 *"X가 실제로 내 컴퓨터에서 동작한다"* 로 가는 고통스러운 과정을 자동화합니다. 논문은 아름다운 수치를 내놓지만, 그것을 재현하는 일은 보통 깨진 코드, 빌드되지 않는 환경, 그리고 버전 드리프트에서 좌절됩니다. ReproRun 은 전체 파이프라인을 끝에서 끝까지 처리합니다:

**논문 읽기 → 코드와 데이터 찾기 → 환경 구축 → 스모크 테스트 → 전체 실행 → 측정된 수치를 논문의 주장과 비교.**

---

## ✨ 적응하도록 설계됨 — 모든 플랫폼에 걸쳐

ReproRun 은 하나의 고정된 설정을 가정하는 대신, **주어진 무엇에든 스스로 적응하도록** 설계되었습니다:

- **Cross-OS** — Windows, macOS, Linux 에서 실행
- **Cross-language** — Python 의 경우 전체 파이프라인; R / MATLAB / Julia 의 경우 최소 경로
- **Cross-domain** — 단일 세포 생물학, 이미지 ML 등, 도메인별 재배선 없이
- **Cross-mode** — *tool mode* (`pip install` + 호출 스크립트 작성) 또는 *experiment mode* (저장소를 clone 하고 해당 스크립트 실행) — 논문마다 자동 선택

---

## 🚀 무엇을 하는가

- **제목만으로 논문 찾기** — PDF 와 코드 저장소를 자동 웹 검색
- **환경 노후화 자동 진단** — numpy ABI 충돌, torchvision API 변경, 더 이상 사용되지 않는 pandas 메서드… 자동으로 감지하고 수정
- **파라미터 추측 없음** — 설치 후 함수 시그니처를 검사
- **5회 라운드 의존성 자가 치유 루프** — 오류 분류 → 표적 수정 → 재검증, 최대 5회 라운드
- **논문 격리** — 모든 논문이 각자의 출력 네임스페이스를 가짐

---

## 🏗️ 아키텍처

하나의 오케스트레이터(`SKILL.md`)가 6개의 전문화된 에이전트를 구동합니다:

| 에이전트 | 역할 |
|-------|------|
| A · paper-reader | 재현할 수치적 주장을 추출 |
| B · resource-finder | 코드 저장소와 데이터셋 위치 파악 |
| C · environment-builder | 런타임 구축 및 복구 (가장 복잡) |
| D · smoke-tester | 빠른 스모크 테스트 — 실행 여부 확인 |
| E · full-runner | 전체 재현 실행 |
| F · result-comparator | 측정값과 주장값을 항목별로 비교 |

---

## ✅ 검증

ReproRun 은 여러 도메인에 걸친 실제 논문들에 대해 끝에서 끝까지 실행되었습니다. 단순히 "성공" 도장을 찍지 않습니다 — 모든 논문에 대해 **정직한 판정** 을 반환합니다: 수치 재현됨, 파이프라인 재현됨, 또는 *현재 상태로는 재현 불가* — 항상 근본 원인과 함께.

| 논문 | 도메인 | 결과 |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | 차원 축소 | ✅ **수치 재현됨** — 14개 중 11개 k-NN 정확도 지표가 ±0.01 이내로 일치; MNIST 및 Fashion-MNIST 가 소수점 3자리까지 확인됨 |
| **scVelo** (Bergen 2020, Nat Biotech) | 단일 세포 | ✅ **파이프라인 재현됨** — 100% NaN 을 유발하는 numpy 2.x ABI 버그를 포착, 1.26.4 로 다운그레이드하여 수정 |
| **Robust Stitching** (Ruiz 2023, ICML) | 이미지 ML | ✅ **파이프라인 재현됨** — 5건의 노후화 손상(torchvision API, pandas `append`, 누락된 의존성)을 복구 |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | 단일 세포 | ✅ **파이프라인 재현됨** — 6회의 API 디버그 라운드로 누락된 `pooch` 의존성을 드러냄 |
| **ScType** (Ianevski 2022, Nat Comms) | 단일 세포 (R) | ✅ **파이프라인 재현됨** — 버전 다운그레이드 후 R 경로 검증됨 |
| **Cropformer** (Wang 2025, Plant Communications) | 작물 유전체학 | ⚠️ **부분 재현** — 코드, 모델, 학습은 검증되었으나, 저장소가 10개의 데모 샘플만 제공하여 논문의 PCC=0.92 를 현재 상태로는 재현할 수 없음 |

**논문 6편 · 전체 지표 재현 1건 · 파이프라인 재현 4건 · 정직한 부분 재현 1건 · 검증된 스킬 개선 18건**

> **설계상 정직함.** UMAP 은 78.6% 지표 일치로 마무리되었는데 — 우리의 80% "깔끔한 재현" 기준에 *약간 못 미치는* 수준 — ReproRun 은 이를 반올림하지 않고 데이터 모순으로 보고합니다. Cropformer 의 경우 프레임워크는 끝에서 끝까지 실행되지만, 발표된 수치는 저장소가 결코 제공하지 않는 실제 작물 유전체 데이터를 필요로 하므로 — Pass 가 아니라 Partial 로 표시됩니다.

<details>
<summary><b>사례 연구 — Cropformer (⚠️ 부분 재현)</b></summary>

**검증됨 ✅**
- 저장소 발견 및 clone (`jiekesen/Cropformer`; 논문의 URL 은 잘못되어 있었음)
- 환경 구축 — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- 모델 아키텍처 — Conv1d + 8-head self-attention (2.6M 파라미터)
- RTX 4090 에서 GPU 추론; 사전 학습된 가중치 로드 및 실행
- 학습 루프 수렴 — loss 89,540 → 23,771

**재현할 수 없음 ❌**
- 논문 지표 (PCC=0.92, …) — 저장소가 실제 작물 데이터가 아닌 무작위 데모 샘플 10개만 제공
- 분류 작업 — `model_class.py` 에 핵심 함수가 누락됨
- 중첩 교차 검증, MIC 특징 선택, 0–9 SNP 인코딩 — 저장소에 구현되어 있지 않음

**근본 원인:** 공개 저장소는 데모 전용이며, 전체 파이프라인(MIC 선택, 중첩 CV, Optuna 튜닝)과 실제 데이터셋이 포함되어 있지 않습니다. 충실한 재현을 위해서는 실제 작물 유전체 데이터 → PLINK 처리 → 기술된 파이프라인 재구현이 필요합니다(데이터 + 연산 작업 약 1~2주).
</details>

---

## 📦 시작하기

ReproRun 은 호환되는 모든 AI 코딩 어시스턴트에서 작동하는 에이전트 스킬입니다. 사용하려면:

1. 스킬을 로드할 수 있는 AI 코딩 에이전트를 사용하세요.
2. `paper-reproduction/` 폴더를 에이전트가 스킬을 로드하는 위치에 두세요.
3. 세션에서 그냥 요청하세요 — 예: *"reproduce scVelo"* 또는 *"reproduce Table 2 from this paper."* 스킬이 자동으로 트리거됩니다.

---

## 👥 팀

| 역할 | 구성원 |
|------|--------|
| Chief Architect | [@WUBING2023](https://github.com/WUBING2023) |
| Development Engineer | [@TXZ-star](https://github.com/TXZ-star) |
| Test Engineer | [@qaqcrane](https://github.com/qaqcrane) |
| Operations | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 라이선스

**비상업적 용도로만 사용 가능.** 비상업적 목적(연구, 학습, 개인 프로젝트)을 위해 ReproRun 을 **사용** 하고 **수정** 하는 것은 자유입니다. **상업적 사용은** 사전 허가 없이는 **허용되지 않습니다.**

라이선스: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [중국어 라이선스 보기 →](../LICENSE.zh-CN.md)

---

## 📌 상태

**v1.0.0** · 안정적 (유지보수 모드)
