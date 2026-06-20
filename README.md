<p align="center">
  <b>English</b> ·
  <a href="docs/README.zh-CN.md">简体中文</a> ·
  <a href="docs/README.es.md">Español</a> ·
  <a href="docs/README.fr.md">Français</a> ·
  <a href="docs/README.de.md">Deutsch</a> ·
  <a href="docs/README.ja.md">日本語</a> ·
  <a href="docs/README.ko.md">한국어</a> ·
  <a href="docs/README.pt-BR.md">Português</a> ·
  <a href="docs/README.ru.md">Русский</a>
</p>

# ReproRun

> Give it a paper. It tells you whether the results actually reproduce.

**ReproRun** is a portable **AI-agent skill** — usable by any compatible AI coding
assistant — that automates the painful path from *"a paper claims X"* to *"X
actually runs on my machine."* Papers ship beautiful numbers; reproducing them usually dies in broken
code, unbuildable environments, and version drift. ReproRun handles the whole
pipeline end to end:

**read paper → find code & data → build environment → smoke test → full run →
compare measured numbers against the paper's claims.**

---

## ✨ Built to adapt — across every platform

ReproRun is designed to **adapt itself to whatever it's handed**, instead of
assuming one fixed setup:

- **Cross-OS** — runs on Windows, macOS, and Linux
- **Cross-language** — full pipeline for Python; minimal path for R / MATLAB / Julia
- **Cross-domain** — single-cell biology, image ML, and more, with no per-domain rewiring
- **Cross-mode** — *tool mode* (`pip install` + write a calling script) or
  *experiment mode* (clone the repo & run its scripts) — auto-selected per paper

---

## 🚀 What it does

- **Find a paper from just its title** — auto web-search for PDF & code repo
- **Auto-diagnose environment bit-rot** — numpy ABI clashes, torchvision API
  changes, deprecated pandas methods… detected and fixed automatically
- **No guessing parameters** — inspects function signatures after install
- **5-round dependency self-healing loop** — classify error → targeted fix →
  re-verify, up to 5 rounds
- **Paper isolation** — every paper gets its own output namespace

---

## 🏗️ Architecture

One orchestrator (`SKILL.md`) drives 6 specialized agents:

| Agent | Role |
|-------|------|
| A · paper-reader | Extract the numerical claims to reproduce |
| B · resource-finder | Locate code repo & datasets |
| C · environment-builder | Build & repair the runtime (most complex) |
| D · smoke-tester | Quick smoke test — confirm it runs |
| E · full-runner | Full reproduction run |
| F · result-comparator | Compare measured vs. claimed, item by item |

---

## ✅ Validation

ReproRun has been run end to end on real papers across domains. It doesn't just
rubber-stamp "success" — for every paper it returns an **honest verdict**: numbers
reproduced, pipeline reproduced, or *can't reproduce as-is* — always with the root cause.

| Paper | Domain | Result |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | dimensionality reduction | ✅ **Numbers reproduced** — 11/14 k-NN accuracy metrics match within ±0.01; MNIST & Fashion-MNIST confirmed to 3 decimals |
| **scVelo** (Bergen 2020, Nat Biotech) | single-cell | ✅ **Pipeline reproduced** — caught a numpy 2.x ABI bug causing 100% NaN, fixed by downgrading to 1.26.4 |
| **Robust Stitching** (Ruiz 2023, ICML) | image ML | ✅ **Pipeline reproduced** — repaired 5 bit-rot breakages (torchvision API, pandas `append`, missing deps) |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | single-cell | ✅ **Pipeline reproduced** — 6 API-debug rounds surfaced a missing `pooch` dependency |
| **ScType** (Ianevski 2022, Nat Comms) | single-cell (R) | ✅ **Pipeline reproduced** — R path validated after a version downgrade |
| **Cropformer** (Wang 2025, Plant Communications) | crop genomics | ⚠️ **Partial** — code, model & training verified, but the repo ships only 10 demo samples, so the paper's PCC=0.92 can't be reproduced as-is |

**6 papers · 1 full-metric reproduction · 4 pipeline reproductions · 1 honest partial · 18 verified skill improvements**

> **Honest by design.** UMAP landed at 78.6% metric match — just *below* our 80%
> "clean reproduce" bar — and ReproRun reports it as a data contradiction rather than
> rounding up. For Cropformer, the framework runs end to end, but the published numbers
> need real crop-genome data the repo never ships — so it's flagged Partial, not Pass.

<details>
<summary><b>Case study — Cropformer (⚠️ Partial reproduction)</b></summary>

**Verified ✅**
- Repo found & cloned (`jiekesen/Cropformer`; the paper's URL was wrong)
- Environment built — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- Model architecture — Conv1d + 8-head self-attention (2.6M params)
- GPU inference on RTX 4090; pretrained weights load & run
- Training loop converges — loss 89,540 → 23,771

**Could not reproduce ❌**
- Paper metrics (PCC=0.92, …) — repo ships only 10 random demo samples, not real crop data
- Classification task — `model_class.py` is missing key functions
- Nested cross-validation, MIC feature selection, 0–9 SNP encoding — not implemented in the repo

**Root cause:** the public repo is demo-only; the full pipeline (MIC selection, nested
CV, Optuna tuning) and the real datasets are not included. A faithful reproduction would
need the real crop-genome data → PLINK processing → reimplementing the described pipeline
(~1–2 weeks of data + compute work).
</details>

---

## 📦 Getting started

ReproRun is an agent skill that works with any compatible AI coding assistant. To use it:

1. Use an AI coding agent that can load skills.
2. Place the `paper-reproduction/` folder where your agent loads skills.
3. In a session, just ask — e.g. *"reproduce scVelo"* or
   *"reproduce Table 2 from this paper."* The skill triggers automatically.

---

## 👥 Team

| Role | Member |
|------|--------|
| Chief Architect | [@WUBING2023](https://github.com/WUBING2023) |
| Development Engineer | [@TXZ-star](https://github.com/TXZ-star) |
| Test Engineer | [@qaqcrane](https://github.com/qaqcrane) |
| Operations | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 License

**Non-commercial use only.** You are free to **use** and **modify** ReproRun for
non-commercial purposes (research, study, personal projects). **Commercial use is
not permitted** without prior permission.

License: **[PolyForm Noncommercial License 1.0.0](LICENSE)**
· 中文版：**[查看中文协议 →](LICENSE.zh-CN.md)**

---

## 📌 Status

**v1.0.0** · stable (maintenance mode)
