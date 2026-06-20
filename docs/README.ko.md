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

> 논문을 건네주세요. 결과가 실제로 재현되는지 알려드립니다.

**ReproRun**은 휴대 가능한 **AI 에이전트 스킬**로, 호환되는 모든 AI 코딩 어시스턴트가 사용할 수 있으며, *"논문이 X를 주장한다"* 에서 *"X가 실제로 내 컴퓨터에서 돌아간다"* 로 가는 고통스러운 여정을 자동화합니다. 논문은 아름다운 숫자를 내놓지만, 그것을 재현하는 일은 대개 깨진 코드, 빌드할 수 없는 환경, 버전 드리프트 속에서 좌절됩니다. ReproRun은 전체 파이프라인을 처음부터 끝까지 처리합니다:

**논문 읽기 → 코드 및 데이터 찾기 → 환경 구축 → 스모크 테스트 → 전체 실행 → 측정된 숫자를 논문의 주장과 비교.**

---

## ✨ 적응하도록 설계됨 — 모든 플랫폼에서

ReproRun은 하나의 고정된 설정을 가정하는 대신, **주어지는 무엇이든 스스로 적응하도록** 설계되었습니다:

- **OS 교차 지원** — Windows, macOS, Linux에서 실행
- **언어 교차 지원** — Python은 전체 파이프라인; R / MATLAB / Julia는 최소 경로
- **도메인 교차 지원** — 단일세포 생물학, 이미지 ML 등, 도메인별 재구성 없이 처리
- **모드 교차 지원** — *도구 모드*(`pip install` + 호출 스크립트 작성) 또는 *실험 모드*(저장소 클론 후 스크립트 실행) — 논문마다 자동 선택

---

## 🚀 무엇을 하는가

- **제목만으로 논문 찾기** — PDF와 코드 저장소를 자동 웹 검색
- **환경 노후화 자동 진단** — numpy ABI 충돌, torchvision API 변경, 더 이상 사용되지 않는 pandas 메서드… 자동으로 감지하고 수정
- **파라미터 추측 없음** — 설치 후 함수 시그니처를 검사
- **5라운드 의존성 자가 치유 루프** — 오류 분류 → 표적 수정 → 재검증, 최대 5라운드
- **논문 격리** — 모든 논문은 각자의 출력 네임스페이스를 가짐

---

## 🏗️ 아키텍처

하나의 오케스트레이터(`SKILL.md`)가 6개의 전문 에이전트를 구동합니다:

| Agent | 역할 |
|-------|------|
| A · paper-reader | 재현할 수치 주장을 추출 |
| B · resource-finder | 코드 저장소 및 데이터셋 위치 파악 |
| C · environment-builder | 런타임 구축 및 복구 (가장 복잡함) |
| D · smoke-tester | 빠른 스모크 테스트 — 실행 여부 확인 |
| E · full-runner | 전체 재현 실행 |
| F · result-comparator | 측정값과 주장값을 항목별로 비교 |

---

## ✅ 검증

서로 다른 도메인과 언어의 4편의 논문에서 처음부터 끝까지 검증되었습니다:

| 논문 | 도메인 | 언어 | 스모크 테스트 |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | 단일세포 | Python | ✓ |
| ScType (Nat Comms 2022) | 단일세포 | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | 단일세포 | Python | ✓ |
| Robust Stitching (ICML 2023) | 이미지 ML | Python | ✓ |

**스모크 테스트 통과율 4/4 · P1 블로커 0 · 검증된 개선 18건**

---

## 📦 시작하기

ReproRun은 호환되는 모든 AI 코딩 어시스턴트와 함께 작동하는 에이전트 스킬입니다. 사용하려면:

1. 스킬을 로드할 수 있는 AI 코딩 에이전트를 사용합니다.
2. `paper-reproduction/` 폴더를 에이전트가 스킬을 로드하는 위치에 둡니다.
3. 세션에서 그냥 요청하세요 — 예: *"reproduce scVelo"* 또는 *"reproduce Table 2 from this paper."* 스킬이 자동으로 트리거됩니다.

---

## 👥 팀

| 역할 | 구성원 |
|------|--------|
| 수석 아키텍트 | [@WUBING2023](https://github.com/WUBING2023) |
| 개발 엔지니어 | [@TXZ-star](https://github.com/TXZ-star) |
| 테스트 엔지니어 | [@qaqcrane](https://github.com/qaqcrane) |
| 운영 | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 라이선스

**비상업적 용도로만 사용 가능.** 비상업적 목적(연구, 학습, 개인 프로젝트)을 위해 ReproRun을 **사용**하고 **수정**할 수 있습니다. 사전 허가 없이는 **상업적 사용이 허용되지 않습니다**.

라이선스: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [중국어 라이선스 보기 →](../LICENSE.zh-CN.md)

---

## 📌 상태

**v1.0.0** · 안정 (유지보수 모드)
