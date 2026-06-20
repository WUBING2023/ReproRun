<p align="center">
  <a href="../README.md">English</a> ·
  <b>简体中文</b> ·
  <a href="README.es.md">Español</a> ·
  <a href="README.fr.md">Français</a> ·
  <a href="README.de.md">Deutsch</a> ·
  <a href="README.ja.md">日本語</a> ·
  <a href="README.ko.md">한국어</a> ·
  <a href="README.pt-BR.md">Português</a> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> 给它一篇论文，它会告诉你这些结果到底能不能复现。

**ReproRun** 是一个可移植的 **AI-agent skill** —— 任何兼容的 AI 编程助手都能使用 —— 它把从*"某篇论文声称 X"*到*"X 真的能在我的机器上跑起来"*这条痛苦的路径自动化了。论文给出的数字往往很漂亮，但复现时通常会死在坏掉的代码、装不起来的环境，以及版本漂移上。ReproRun 把整条流水线从头到尾全部接管：

**读论文 → 找代码和数据 → 搭建环境 → 冒烟测试 → 完整运行 → 把实测数字和论文的声称逐项对比。**

---

## ✨ 为适配而生 —— 覆盖每一种平台

ReproRun 的设计理念是**让自己去适配手上拿到的任何东西**，而不是假设只有一套固定的配置：

- **跨操作系统** —— 可在 Windows、macOS 和 Linux 上运行
- **跨语言** —— Python 走完整流水线；R / MATLAB / Julia 走精简路径
- **跨领域** —— 单细胞生物学、图像 ML 等等，无需针对每个领域重新接线
- **跨模式** —— *工具模式*（`pip install` + 写一个调用脚本）或*实验模式*（克隆仓库并运行它的脚本）—— 按每篇论文自动选择

---

## 🚀 它能做什么

- **仅凭标题就能找到论文** —— 自动联网搜索 PDF 和代码仓库
- **自动诊断环境腐化** —— numpy ABI 冲突、torchvision API 变更、被弃用的 pandas 方法…… 全部自动检测并修复
- **不靠猜测参数** —— 安装后会检查函数签名
- **5 轮依赖自愈循环** —— 分类错误 → 针对性修复 → 重新验证，最多 5 轮
- **论文隔离** —— 每篇论文都有自己独立的输出命名空间

---

## 🏗️ 架构

一个编排器（`SKILL.md`）驱动 6 个专职 agent：

| Agent | 职责 |
|-------|------|
| A · paper-reader | 提取需要复现的数值声称 |
| B · resource-finder | 定位代码仓库和数据集 |
| C · environment-builder | 搭建并修复运行环境（最复杂） |
| D · smoke-tester | 快速冒烟测试 —— 确认能跑起来 |
| E · full-runner | 完整复现运行 |
| F · result-comparator | 逐项对比实测值与声称值 |

---

## ✅ 验证

已在 4 篇跨领域、跨语言的论文上完成端到端验证：

| 论文 | 领域 | 语言 | 冒烟测试 |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | 单细胞 | Python | ✓ |
| ScType (Nat Comms 2022) | 单细胞 | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | 单细胞 | Python | ✓ |
| Robust Stitching (ICML 2023) | 图像 ML | Python | ✓ |

**冒烟测试通过率 4/4 · P1 级阻塞问题 0 · 18 项已验证改进**

---

## 📦 快速上手

ReproRun 是一个可与任何兼容的 AI 编程助手配合使用的 agent skill。使用方法：

1. 使用一个能加载 skill 的 AI 编程 agent。
2. 把 `paper-reproduction/` 文件夹放到你的 agent 能加载 skill 的位置。
3. 在会话里直接提要求即可 —— 例如*"复现 scVelo"*或*"复现这篇论文的 Table 2"*。skill 会自动触发。

---

## 👥 团队

| 角色 | 成员 |
|------|--------|
| 首席架构师 | [@WUBING2023](https://github.com/WUBING2023) |
| 开发工程师 | [@TXZ-star](https://github.com/TXZ-star) |
| 测试工程师 | [@qaqcrane](https://github.com/qaqcrane) |
| 运营 | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 许可证

**仅限非商业用途。** 你可以自由地为非商业目的（研究、学习、个人项目）**使用**和**修改** ReproRun。未经事先许可，**不允许商业用途**。

许可证：**[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [查看中文协议 →](../LICENSE.zh-CN.md)

---

## 📌 状态

**v1.0.0** · 稳定（维护模式）
