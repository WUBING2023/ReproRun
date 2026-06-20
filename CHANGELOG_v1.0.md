# CHANGELOG: v1 draft → v1.0.0 stable

**升级日期**: 2026-06-16 | **作者**: ReproRun Team + Claude Code

## 一句话

从"能跑 2 篇单细胞论文的 Python tool 原型"升级为"覆盖 4 篇多领域论文、双模式、18 项工程化改进、零 P1 阻断的生产可用 skill"

## 数字对比

| 指标 | v1 draft (6/10) | v1.0.0 (6/16) |
|------|:--:|:--:|
| 测试论文 | 2（均为单细胞） | **4**（单细胞×3 + 图像ML×1） |
| 覆盖语言 | Python + R（scope violation） | Python full + R/J/M minimal |
| 覆盖模式 | tool（pip install） | **tool + experiment**（clone repo） |
| 已验证改进 | 10 项 | **18 项** |
| P1 阻断 | 2（TaskLog 路径不一致、目录覆盖） | **0** |
| Smoke Test 成功率 | 3/3 | **4/4 (100%)** |
| Loop Engineering | 无 → 每次 session 从零问 | **完整循环体系**（test→发现→入队→修→验证） |
| 审计问题 | 无审计机制 | **全量审计 20 项 → 5 项修复** |
| API 探测 | 硬编码参数名 | **inspect.signature() 自动读** |
| 交付文档 | README + DEV_JOURNAL | **+ CHANGELOG + loop_state + 验收结论** |

## 合作者 9 条反馈：全部处理完成

| # | 反馈（来自 6/10 .docx） | 处理结果 | 对应文件 |
|---|--------------------------|---------|---------|
| 🔴1 | numpy ABI 阻断：pip 成功 import 崩溃 | Agent C 新增 §1.5 numpy<2.0 预 pin + §4a ABI 探针自动恢复 | agent-environment-builder.md |
| 🔴2 | 依赖冲突的"重启 Agent C 3 次"太粗暴 | 升级为 §4c 结构化诊断循环（5 轮、5 类错误分类、定点修复） | agent-environment-builder.md |
| 🔴3 | 输出目录在 skill 目录内不合理 | 移到项目根目录 `paper_reproduction_output/` | SKILL.md Step 0 |
| 🟡4 | Agent F 的跨 agent 模式分析需要 TaskLog 数据 | 全部 6 个 Agent 加 TaskLog 写入指令；Agent F 特别加跨 agent 模式分析 | 全部 7 个文件 |
| 🟡5 | R/MATLAB 应该降级为 minimal path 而非阻断 | Scope 从"Python-only + STOP"改为"Python full + R/M/J minimal" | SKILL.md + Agent C/D/E/F |
| 🟢6 | WebFetch 在大段代码块里不靠谱 | 改为自然语言指示（非代码块） | 全部 Agent 文件 |
| 🟢7 | reproduction_report.md 有矛盾表格 | 删除重复/矛盾内容 | reproduction_report.md |
| 🟢8 | 开发日志缺核验审计环节 | TaskLog 协议 + loop state + 审计机制（6/16） | SKILL.md + loop_state.md |
| 🔵9 | R/MATLAB 定位表述不清 | 写清"Python 完整 + R/M/Julia 最小路径"，加决策表 | SKILL.md + agent-environment-builder.md |

**额外：第 10 项 numpy ABI 遗漏**（开发日志标"已修复"但 Agent C 指令里没写）→ 已补入 Agent C §1.5 + §4a。

## 6/10 之后新增核心能力

### 测试驱动（2 篇新论文→5 项改进）

| 论文 | 发现 | 改进 |
|------|------|------|
| **Annotatability** (Nat Comp Sci 2024) | 6 次 API 调试才找到正确调用方式 | Agent C §4.5：pip install 后用 `inspect.signature()` 读函数签名 |
| | `scvi.data.retina()` 运行时炸 `pooch` | Agent C §4c：运行时 ModuleNotFoundError 自动 pip install |
| | 每个 batch 打印 loss 刷屏 | Agent D §4：stdout 抑制 + stderr 保持可见 |
| **Robust Stitching** (ICML 2023) | 5 层深 bit-rot 链（torchvision API + pandas append + 缺失包） | experiment 模式全流程验证 + import 惰性加载策略 |

### 全局审计驱动（20 项审计→5 项修复）

| 审计问题 | 修复 |
|----------|------|
| P2 输出目录扁平（第二篇覆盖第一篇） | 论文级命名空间 `paper_reproduction_output/<slug>/` |
| P1 TaskLog 路径不一致（6 个 Agent 写不同位置） | 全部统一为 `.claude/tasklog/YYYYMMDD-<slug>.jsonl` |
| P2 reproduction_manifest.md 无人生成 | Step 6.5：orchestrator 自动汇总产出清单 |
| P2 Agent D 缺时间基准 | §4.5：1 单位计时→全量外推→超 60 分警告 |
| P1 PDF 找不到/数据全死 → 无分支处理 | Step 2.6：Gate A 降级继续 / Gate B 停止写报告 |

### 架构驱动（Loop Engineering）

| 新增文件 | 作用 |
|---------|------|
| `CLAUDE.md` | 用户画像 + 项目架构 + Loop 触发 |
| `AGENTS.md` | 子 Agent 行为规范（不上报用户、TaskLog 协议、死循环防护） |
| `.claude/loop/loop_state.md` | 循环状态（阶段、改进队列、已测论文、遗留问题） |
| `.claude/loop/LOOP.md` | 循环协议文档（session 流程、优先级、3 次重试上限） |
| `.claude/settings.json` | 权限预设（减少弹窗） |

## 遗留问题（坦诚说明）

| 问题 | 状态 | 为什么现在不修 |
|------|------|---------------|
| scVelo dynamics EM 产出 57-62% NaN | 待 v2 | scVelo EM solver 内部数值不稳定，需 EM 算法专家诊断。v1 能检测、能记录、能降级到随机模式 |
| GEO 大数据下载阻断 | 永不修 | 网络基础设施问题，skill 管不了。改用内置数据集（scvi-tools/torchvision/scanpy） |
| Nature PDF SSL 证书错误 | 已降级 | PMC 摘要替代方案已工作。PDF 获取不是核心路径 |

## 文件变更总汇

| 操作 | 文件 |
|------|------|
| 新增 | CLAUDE.md, AGENTS.md, CHANGELOG_v1.0.md, .claude/loop/LOOP.md, .claude/loop/loop_state.md, .claude/settings.json |
| 大幅修改 | SKILL.md (558→~600 行), README.md (142→110 行重写), DEVELOPMENT_JOURNAL.md (649→750 行) |
| 细节修改 | 全部 6 个 Agent reference 文件（TaskLog 路径、API 探测、时间基准、失败处理） |
| 归档 | paper_reproduction_output/ 下 3 篇→4 篇 slug 隔离目录 |
