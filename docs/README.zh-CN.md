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

> 给它一篇论文，它会告诉你论文里的结果到底能不能复现。

**ReproRun** 是一个可移植的 **AI-agent skill**——任何兼容的 AI 编程助手都能使用——它自动化了从*“一篇论文声称 X”*到*“X 真的能在我的机器上跑起来”*这条令人头疼的路径。论文里给出的数字往往很漂亮，但复现通常会死在坏掉的代码、装不起来的环境和版本漂移上。ReproRun 端到端地处理整条流水线：

**读论文 → 找代码和数据 → 搭建环境 → 冒烟测试 → 完整运行 → 把实测数字和论文声称的数字对比。**

---

## ✨ 为适应而生——覆盖每一个平台

ReproRun 的设计理念是**让自己去适应手头拿到的任何东西**，而不是假设只有一种固定的配置：

- **跨操作系统** — 可在 Windows、macOS 和 Linux 上运行
- **跨语言** — Python 走完整流水线；R / MATLAB / Julia 走最小路径
- **跨领域** — 单细胞生物学、图像 ML 等等，无需为每个领域重新接线
- **跨模式** — *工具模式*（`pip install` + 写一个调用脚本）或*实验模式*（克隆仓库并运行其脚本）——按论文自动选择

---

## 🚀 它能做什么

- **仅凭标题就能找到论文** — 自动联网搜索 PDF 和代码仓库
- **自动诊断环境腐化** — numpy ABI 冲突、torchvision API 变更、已废弃的 pandas 方法……都能自动检测并修复
- **不靠猜参数** — 安装后检查函数签名
- **5 轮依赖自愈循环** — 分类错误 → 针对性修复 → 重新验证，最多 5 轮
- **论文隔离** — 每篇论文都有自己独立的输出命名空间

---

## 🏗️ 架构

一个编排器（`SKILL.md`）驱动 6 个专门的 agent：

| Agent | 角色 |
|-------|------|
| A · paper-reader | 提取需要复现的数值声明 |
| B · resource-finder | 定位代码仓库和数据集 |
| C · environment-builder | 搭建并修复运行时（最复杂） |
| D · smoke-tester | 快速冒烟测试——确认能跑起来 |
| E · full-runner | 完整复现运行 |
| F · result-comparator | 逐项对比实测值与声称值 |

---

## ✅ 验证

ReproRun 已经在多个领域的真实论文上端到端运行过。它不会简单地盖一个“成功”的章——对每一篇论文它都会给出一个**诚实的结论**：Numbers reproduced、Pipeline reproduced，或者*无法照原样复现*——并且始终附带根本原因。

| 论文 | 领域 | 结果 |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | 降维 | ✅ **Numbers reproduced** — 14 项 k-NN 准确率指标中有 11 项在 ±0.01 内吻合；MNIST 和 Fashion-MNIST 在 3 位小数上得到确认 |
| **scVelo** (Bergen 2020, Nat Biotech) | 单细胞 | ✅ **Pipeline reproduced** — 抓出了一个导致 100% NaN 的 numpy 2.x ABI bug，通过降级到 1.26.4 修复 |
| **Robust Stitching** (Ruiz 2023, ICML) | 图像 ML | ✅ **Pipeline reproduced** — 修复了 5 处腐化故障（torchvision API、pandas `append`、缺失依赖） |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | 单细胞 | ✅ **Pipeline reproduced** — 6 轮 API 调试暴露出一个缺失的 `pooch` 依赖 |
| **ScType** (Ianevski 2022, Nat Comms) | 单细胞 (R) | ✅ **Pipeline reproduced** — 在一次版本降级后验证了 R 路径 |
| **Cropformer** (Wang 2025, Plant Communications) | 作物基因组学 | ⚠️ **Partial** — 代码、模型和训练均已验证，但仓库只提供了 10 个示例样本，因此论文里的 PCC=0.92 无法照原样复现 |

**6 篇论文 · 1 次全指标复现 · 4 次流水线复现 · 1 次诚实的部分复现 · 18 项已验证的 skill 改进**

> **Honest by design.** UMAP 落在 78.6% 的指标吻合度——刚好*低于*我们 80% 的“干净复现”门槛——ReproRun 把它如实报告为数据矛盾，而不是四舍五入往上凑。对于 Cropformer，框架能端到端运行，但发表的数字需要仓库从未提供的真实作物基因组数据——所以它被标记为 Partial，而不是 Pass。

<details>
<summary><b>Case study — Cropformer (⚠️ 部分复现)</b></summary>

**Verified ✅**
- 找到并克隆了仓库（`jiekesen/Cropformer`；论文里的 URL 是错的）
- 搭好了环境 — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- 模型架构 — Conv1d + 8 头自注意力（2.6M 参数）
- 在 RTX 4090 上做 GPU 推理；预训练权重能加载并运行
- 训练循环收敛 — loss 89,540 → 23,771

**Could not reproduce ❌**
- 论文指标（PCC=0.92，……） — 仓库只提供了 10 个随机示例样本，而不是真实作物数据
- 分类任务 — `model_class.py` 缺少关键函数
- 嵌套交叉验证、MIC 特征选择、0–9 SNP 编码 — 仓库里都没有实现

**Root cause：** 公开仓库只是演示用途；完整流水线（MIC 选择、嵌套 CV、Optuna 调参）和真实数据集都没有包含在内。一次忠实的复现需要真实的作物基因组数据 → PLINK 处理 → 重新实现论文描述的流水线（大约 1–2 周的数据 + 算力工作量）。
</details>

---

## 📦 快速开始

ReproRun 是一个 agent skill，可与任何兼容的 AI 编程助手配合使用。使用方法：

1. 使用一个能够加载 skill 的 AI 编程 agent。
2. 把 `paper-reproduction/` 文件夹放到你的 agent 加载 skill 的位置。
3. 在一次会话里直接提问即可——例如*“复现 scVelo”*或*“复现这篇论文的 Table 2”*。skill 会自动触发。

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

**仅限非商业用途。** 你可以出于非商业目的（研究、学习、个人项目）自由地**使用**和**修改** ReproRun。未经事先许可，**不允许商业使用**。

许可证：**[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [查看中文协议 →](../LICENSE.zh-CN.md)

---

## 📌 状态

**v1.0.0** · 稳定（维护模式）
