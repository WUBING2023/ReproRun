# ReproRun

> Give it a paper. It tells you whether the results actually reproduce.
> 给它一篇论文，它告诉你：论文里的实验数据，到底能不能复现。

**ReproRun** is a custom [Claude Code](https://claude.com/claude-code) skill that
automates the painful path from *"a paper claims X"* to *"X actually runs on my
machine."* Papers ship beautiful numbers; reproducing them usually dies in broken
code, unbuildable environments, and version drift. ReproRun handles the whole
pipeline end to end:

**read paper → find code & data → build environment → smoke test → full run →
compare measured numbers against the paper's claims.**

> 中文：ReproRun 是一个 Claude Code 自定义 skill。论文常常贴出漂亮的数字，
> 但别人复现时往往卡在"代码跑不起来、环境装不上、版本对不上"。ReproRun 把
> 这套流程全自动化：读论文 → 找代码和数据 → 搭环境 → 冒烟测试 → 完整运行 →
> 把实测数字和论文里的数字逐项比对。

---

## ✨ Built to adapt — across every platform / 为"跨平台适应"而生

ReproRun is designed to **adapt itself to whatever it's handed**, instead of
assuming one fixed setup:

- **Cross-OS** — runs on Windows, macOS, and Linux
- **Cross-language** — full pipeline for Python; minimal path for R / MATLAB / Julia
- **Cross-domain** — single-cell biology, image ML, and more, with no per-domain rewiring
- **Cross-mode** — *tool mode* (`pip install` + write a calling script) or
  *experiment mode* (clone the repo & run its scripts) — auto-selected per paper

> 中文：ReproRun 的核心设计目标就是**自适应**——不预设某一种固定环境，而是
> 根据拿到的论文自动调整：跨操作系统（Windows / macOS / Linux）、跨语言
> （Python 全流程，R / MATLAB / Julia 最小路径）、跨领域（单细胞、图像 ML 等
> 无需逐一改造）、跨模式（工具模式 / 实验模式按论文自动切换）。

---

## 🚀 What it does / 核心能力

- **Find a paper from just its title** — auto web-search for PDF & code repo
  （只给标题就能找论文）
- **Auto-diagnose environment bit-rot** — numpy ABI clashes, torchvision API
  changes, deprecated pandas methods… detected and fixed automatically
  （自动诊断并修复环境腐烂）
- **No guessing parameters** — inspects function signatures after install
  （装完包先读函数签名，不瞎猜参数）
- **5-round dependency self-healing loop** — classify error → targeted fix →
  re-verify, up to 5 rounds （5 轮依赖诊断自愈循环）
- **Paper isolation** — every paper gets its own output namespace
  （每篇论文独立产出目录，互不覆盖）

---

## 🏗️ Architecture / 架构

One orchestrator (`SKILL.md`) drives 6 specialized agents:

| Agent | Role / 职责 |
|-------|-------------|
| A · paper-reader | Extract the numerical claims to reproduce / 提取待复现的数值声明 |
| B · resource-finder | Locate code repo & datasets / 找代码和数据 |
| C · environment-builder | Build & repair the runtime (most complex) / 搭建并修复环境 |
| D · smoke-tester | Quick smoke test — confirm it runs / 快速冒烟测试 |
| E · full-runner | Full reproduction run / 完整复现运行 |
| F · result-comparator | Compare measured vs. claimed, item by item / 逐项比对 |

---

## ✅ Validation / 验证情况

Validated end to end on 4 papers across different domains and languages:

| Paper | Domain | Language | Smoke Test |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | single-cell | Python | ✓ |
| ScType (Nat Comms 2022) | single-cell | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | single-cell | Python | ✓ |
| Robust Stitching (ICML 2023) | image ML | Python | ✓ |

**Smoke-test pass rate 4/4 · P1 blockers 0 · 18 verified improvements**
（冒烟测试成功率 4/4 · P1 阻断 0 · 累计已验证改进 18 项）

---

## 📦 Getting started / 快速开始

ReproRun is a Claude Code skill. To use it:

1. Install [Claude Code](https://claude.com/claude-code).
2. Place the `paper-reproduction/` folder where Claude Code can load skills.
3. In a Claude Code session, just ask — e.g. *"复现一下 scVelo 这篇论文"* or
   *"reproduce Table 2 from this paper."* The skill triggers automatically.

> 中文：ReproRun 是一个 Claude Code skill。装好 Claude Code 后，把
> `paper-reproduction/` 放到 skill 目录，然后在对话里直接说"复现某篇论文"即可，
> skill 会自动触发。

---

## 👥 Team / 团队

| Role / 角色 | Member |
|------|--------|
| Chief Architect / 总架构 | [@WUBING2023](https://github.com/WUBING2023) |
| Development Engineer / 开发工程师 | [@TXZ-star](https://github.com/TXZ-star) |
| Test Engineer / 测试工程师 | [@qaqcrane](https://github.com/qaqcrane) |
| Operations / 运营 | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 License / 许可协议

**Non-commercial use only.** You are free to **use** and **modify** ReproRun for
non-commercial purposes (research, study, personal projects). **Commercial use is
not permitted** without prior permission.

License: **[PolyForm Noncommercial License 1.0.0](LICENSE)**
· 中文版：**[查看中文协议 →](LICENSE.zh-CN.md)**

> 中文：**仅限非商业用途。** 你可以自由地**使用**和**修改** ReproRun 用于
> 非商业目的（科研、学习、个人项目），但**未经许可不得商用**。
> 采用 **PolyForm Noncommercial 1.0.0** 协议，中文翻译见
> **[LICENSE.zh-CN.md](LICENSE.zh-CN.md)**（仅供参考，以英文原文为准）。

---

## 📌 Status / 状态

**v1.0.0** · stable (maintenance mode) · maintained on Claude Code
