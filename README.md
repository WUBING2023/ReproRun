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

**ReproRun** is a custom [Claude Code](https://claude.com/claude-code) skill that
automates the painful path from *"a paper claims X"* to *"X actually runs on my
machine."* Papers ship beautiful numbers; reproducing them usually dies in broken
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

Validated end to end on 4 papers across different domains and languages:

| Paper | Domain | Language | Smoke Test |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | single-cell | Python | ✓ |
| ScType (Nat Comms 2022) | single-cell | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | single-cell | Python | ✓ |
| Robust Stitching (ICML 2023) | image ML | Python | ✓ |

**Smoke-test pass rate 4/4 · P1 blockers 0 · 18 verified improvements**

---

## 📦 Getting started

ReproRun is a Claude Code skill. To use it:

1. Install [Claude Code](https://claude.com/claude-code).
2. Place the `paper-reproduction/` folder where Claude Code can load skills.
3. In a Claude Code session, just ask — e.g. *"reproduce scVelo"* or
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

**v1.0.0** · stable (maintenance mode) · maintained on Claude Code
