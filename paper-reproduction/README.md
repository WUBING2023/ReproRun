# paper-reproduction skill — v1.0 合作者交付包

**版本**: v1.0.0 | **状态**: stable（维护模式） | **验收日期**: 2026-06-16

## 这是什么

一个 Claude Code skill，用 1 个编排器 + 6 个专职 Agent 自动复现学术论文的数值声明。
4 篇不同类型论文全流程验证通过。

## 快速导航

| 你要看什么 | 打开这个文件 |
|---|---|
| **📖 完整开发过程 + 验收结论** | `DEVELOPMENT_JOURNAL.md` |
| **📋 v1.0 竣工验收标准** | `../.claude/loop/loop_state.md` |
| skill 主编排逻辑 | `SKILL.md` |
| 6 个 Agent 的详细指令 | `references/agent-*.md` |

## 4 篇论文复现统计

| # | 论文 | 领域 | 语言 | 模式 | Smoke Test | 关键发现 |
|---|------|------|------|------|:--:|------|
| 1 | **scVelo** (Bergen 2020, Nat Biotech) | 单细胞 | Python | tool | ✓ | numpy 2.x ABI 导致 100% NaN — 降级到 1.26.4 后恢复 |
| 2 | **ScType** (Ianevski 2022, Nat Comms) | 单细胞 | R | minimal | ✓ | R 语言检测门正确阻断，降级后手动验证 |
| 3 | **Annotatability** (Nitzan 2024, Nat Comp Sci) | 单细胞 | Python | tool | ✓ | 6 次 API 调试 → 发现 pooch 缺失依赖 + 训练输出噪音 |
| 4 | **Robust Stitching** (Ruiz 2023, ICML) | 图像ML | Python | experiment | ✓ | 5 个 bit-rot 问题（torchvision API + pandas append + 缺失包链） |

**Smoke Test 成功率**: 4/4 (100%) | **P1 阻断**: 0 | **已验证改进**: 18 项

## Skill 核心能力

| 能力 | 说明 |
|------|------|
| 自动 find paper | 给标题就能找论文（WebSearch/PMC 降级） |
| 环境 bit-rot 检测 | numpy ABI、torchvision API、pandas 废弃 — 自动诊断+修复 |
| API 签名 inspect | pip install 后自动读函数签名，不瞎猜参数名 |
| 5 轮依赖诊断循环 | 分类错误 → 定点修复 → 重验证，最多 5 轮 |
| 多语言支持 | Python 全路径 + R/MATLAB/Julia 最小路径 |
| 实验模式 + 工具模式 | clone repo 跑脚本 / pip install 写调用脚本 |
| 论文隔离命名空间 | paper_reproduction_output/<slug>/ 互不覆盖 |
| Loop Engineering | test→发现→入队→修→验证→更新状态，session 间继承 |

## 交付物

```
fuxian_skill/
├── paper-reproduction/                     ← skill 本体
│   ├── README.md                           ← 你正在读的文件
│   ├── SKILL.md (v1.0.0)                   ← 编排器
│   ├── DEVELOPMENT_JOURNAL.md              ← 完整开发日志 + 验收
│   └── references/                         ← 6 个 Agent 指令
├── paper_reproduction_output/              ← 4 篇论文产出（按 slug 隔离）
│   ├── bergen2020/   (scVelo)
│   ├── ianevski2022/ (ScType)
│   ├── nitzan2024/   (Annotatability)
│   └── szeged2023/   (Robust Stitching)
├── .claude/loop/                           ← Loop Engineering 基础设施
│   ├── loop_state.md                       ← 当前状态 + 改进队列
│   └── LOOP.md                             ← 循环协议
├── .claude/tasklog/                        ← 结构化审计日志
├── CLAUDE.md                               ← 用户画像 + PM 思维模式
└── AGENTS.md                               ← 子 Agent 行为规范
```

---

*交付日期：2026-06-16 · 版本：v1.0.0 (stable)*
