---
name: paper-reproduction
version: v1.0.0
status: stable
tested_on: scVelo(2020), ScType(2022), Annotatability(2024), Robust-Stitching(2023)
description: >
  Use whenever the user wants to reproduce, replicate, or verify results from
  an academic paper — especially ML/CV/NLP papers with public GitHub repos and
  Python codebases. Triggers on: 复现, reproduce, replicate, 复现论文, 跑论文代码,
  verify results, paper reproduction, 验证实验结果, 论文复现, check if a paper's
  results are real, run this paper's code, reproduce table X from paper Y.
  Handles the full pipeline: read paper → find code & data → build environment →
  smoke test → full run → compare results against paper claims. Even if the user
  only mentions a paper title without a PDF, invoke this skill — it will search
  for the PDF automatically.
---

# Paper Reproduction

Six-step automated paper reproduction pipeline. One orchestrator (you) plus six
specialized sub-agents. Agents communicate only through structured markdown files
— they do not share conversation context.

## Scope (v1)

**v1 positioning: Python = full 6-agent reproduction with numerical comparison. R / MATLAB / Julia = minimal viable — identify, install, smoke-test, honest feedback. This is a deliberate scope choice: the Python path must be rock-solid before expanding to languages where error diagnostics are harder for AI to resolve.**

### Language Support

| Language | Support Level | What It Does |
|---|---|---|
| **Python** | **Full** | All 6 agents active. Environment build → smoke test → full run → numerical comparison. |
| **R** | **Minimal** | Identify R + install packages + smoke test + honest feedback. Does NOT promise full numerical reproduction. |
| **MATLAB** | **Minimal** | Identify MATLAB + check toolboxes + smoke test + honest feedback. Does NOT promise full numerical reproduction. |
| **Julia** | **Minimal** | Same as R/MATLAB minimal path. |
| **No code / theory paper** | **Blocked** | Write `scope_violation.md` and STOP. No repository to reproduce. |

Detection is two-phase:

| Phase | When | How |
|---|---|---|
| **Pre-flight** | Step 0 (before launching any agents) | Quick check via file extension heuristics and user-provided URLs. `pip install X` / PyPI → Python. `install.packages()` / `devtools::install_github()` / `library(X)` → R. `.m` files / MATLAB → MATLAB. `.jl` / `Pkg.add()` → Julia. |
| **Confirmation** | Step 2 (after resource_map is available) | Read "Dependency Files Found" checklist. Confirm language matches pre-flight. If pre-flight was unknown (only paper title), this is the definitive detection. |

### Minimal Path (R / MATLAB / Julia)

When a non-Python language is detected, do NOT block. Instead, write
`paper_reproduction_output/<paper_slug>/reduced_scope.md`:

```markdown
# Reduced Scope — Non-Python Repository

## Detected Language: R / MATLAB / Julia
## Evidence
- (dependency files found, README commands, etc.)

## What This Means
This paper's code is in {language}, which is outside Python — the skill's
full reproduction path. The skill will proceed on a **minimal path**:
1. ✓ Identify the language environment
2. ✓ Install packages (best effort)
3. ✓ Run a smoke test (does the code start?)
4. ✗ Full numerical reproduction is NOT guaranteed
5. ✗ Result comparison against paper claims is NOT guaranteed

## What the User Gets
- Honest feedback on whether the code runs
- Known blockers documented with concrete error messages
- TaskLog entries for all failures — reusable across papers
```

Then proceed to Step 1 normally. The orchestrator sets an internal flag
`language_scope: minimal` and passes it to all agents. Agents adapt:

- **Agent C (EnvironmentBuilder):** Instead of creating a Python venv, install
  packages in the detected language (R: `install.packages()` from CRAN mirror;
  MATLAB: check `ver` for required toolboxes). Verify each package loads.
- **Agent D (SmokeTester):** Run a minimal "hello world" in the detected language
  (R: `source("main.R")` or equivalent; MATLAB: `run('main.m')`).
- **Agent E (FullRunner):** Attempt to run what's possible. Write a wrapper if
  needed. Report honestly what ran and what didn't.
- **Agent F (ResultComparator):** Note the reduced scope. List which paper claims
  could be checked and which couldn't. Do NOT fabricate comparisons.

### Dataset Requirements

Paper must have a public dataset (Zenodo, Kaggle, Hugging Face, UCI, GEO,
CELLxGENE, ArrayExpress, etc.). If no public dataset is found, mark it in the
resource map and continue on a best-effort basis.

### Two Reproduction Modes (Python only)

The orchestrator auto-detects which mode applies after Step 2. Only relevant
for Python repositories — non-Python repos always use the minimal path above.

| Mode | Repo Shape | Reproduction Path | Example |
|---|---|---|---|
| **experiment** | `main.py`, `train.py`, config files, shell scripts | Clone → env → run scripts → compare | Most ML papers |
| **tool** | `setup.py`/`pyproject.toml`, `pip install` instructions, Python API | pip/conda install → write API-calling script → run → compare | CellTypist, Scanpy, tools with `pip install X` |

Mode detection is automatic — read "Step 2.5 — Detect Reproduction Mode" below.
The orchestrator passes `reproduction_mode: tool|experiment` to Agent C and Agent E
so they adapt their behavior.

## Quick Reference: Six Agents

| Agent | Reference File | Inputs | Output |
|---|---|---|---|
| A: PaperReader | `references/agent-paper-reader.md` | PDF path or paper title | `extracted_methodology.md` |
| B: ResourceFinder | `references/agent-resource-finder.md` | Paper title, any GitHub URLs from A | `resource_map.md` |
| C: EnvironmentBuilder | `references/agent-environment-builder.md` | A + B outputs | `setup_report.md` |
| D: SmokeTester | `references/agent-smoke-tester.md` | C output + resource_map | `smoke_test_report.md` |
| E: FullRunner | `references/agent-full-runner.md` | All prior outputs | `full_run_report.md` + outputs/ |
| F: ResultComparator | `references/agent-result-comparator.md` | A (claims) + E (actuals) | `reproduction_report.md` |

Read each reference file **only when launching that agent**. Do not load all
six into context at once.

## Orchestration Pipeline

### Step 0 — Setup

1. Confirm the paper identity. If the user gave a PDF path, use it. If only a
   title, Agent A will search for the PDF.
2. **Language pre-flight check**: If the user provided a GitHub URL or package
   name, do a quick pre-flight before launching agents:
   - `pip install X` / PyPI mention / `conda install` → Python → **full path**
   - `devtools::install_github()` / `install.packages()` / `BiocManager::install()` / CRAN / Bioconductor → R → **minimal path** (write `reduced_scope.md`, set `language_scope: minimal`, continue)
   - `.jl` files / `Julia` mention / `Pkg.add()` → Julia → same minimal path
   - `.m` files / `MATLAB` / `addpath()` → MATLAB → same minimal path
   - No code repo / theory paper only → **block** (write `scope_violation.md`, STOP)
   - Unknown (no URL given, only a paper title) → proceed; Step 2 will confirm
3. **Generate paper slug.** From first-author-lastname + year: `bergen2020`,
   `nitzan2024`, etc. Strip diacritics, lowercase, no spaces. If year unknown,
   use `0000`. This slug namespaces ALL downstream output so running multiple
   papers never overwrites each other's artifacts.
4. Create the output directory tree **in the working directory** (not inside the
   skill directory). The skill itself (SKILL.md + references/) should remain
   clean — `paper_reproduction_output/` is a runtime artifact that lives
   alongside the skill, not inside it.

```
<working_directory>/
├── paper-reproduction/          ← skill body (SKILL.md + references/)
└── paper_reproduction_output/   ← runtime outputs (created below)
    └── <paper_slug>/            ← ONE SUBDIRECTORY PER PAPER
        ├── paper_reading/
        ├── env_setup/
        ├── run_outputs/
        │   └── outputs/
        ├── resources/
        ├── reproduction_report.md
        ├── reproduction_manifest.md
        ├── paper_reference_mismatch.md   (only if mismatch detected)
        ├── scope_violation.md            (only if non-Python detected)
        ├── reduced_scope.md              (only if R/MATLAB/Julia detected)
        ├── data_unavailable.md           (only if ALL data sources dead)
        └── degraded_mode.md              (only if web tools unavailable)
```

The orchestrator passes `<paper_slug>` to all agents so they write to the
correct subdirectory. **Each paper gets a fresh slug** — if the same paper
is re-tested after skill improvements, reuse the SAME slug so artifacts
accumulate under one namespace.

5. **Web availability check**. Before launching Agents A or B, verify web tools
   are functional with a 2-second sanity check:

   Use the **WebFetch tool** (not a shell command) to probe GitHub:
   `WebFetch https://github.com --prompt "Does this page load?"`

   **If web tools are available** → proceed to Step 1.

   **If web tools are unavailable** → **degraded mode**. Write
   `paper_reproduction_output/<paper_slug>/degraded_mode.md`:

   ```markdown
   # Degraded Mode — Web Tools Unavailable

   ## What happened
   WebSearch and/or WebFetch returned errors in this session (HTTP 400,
   ECONNREFUSED, or timeout). The reproduction pipeline itself
   (Steps 3-7) is unaffected, but the paper/code discovery phase
   (Steps 1-2) requires the user to provide inputs manually.

   ## What still works
   - Environment setup (Step 3)
   - Smoke test + time benchmark (Steps 4-4.5)
   - Full run (Step 5)
   - Result comparison (Step 6)
   - Merge and presentation (Step 7)

   ## What does not work
   - Automatic paper PDF search (Agent A's search path)
   - Automatic GitHub repo search (Agent B's search path)
   - Data URL discovery from paper references
   ```

   Present the degraded mode to the user:

   ```
   Web tools are currently unavailable. The reproduction pipeline itself
   is unaffected, but I cannot search for the paper PDF or GitHub repo.
   How would you like to proceed?

   A) I'll provide the PDF path and GitHub URL — continue in degraded mode
   B) Retry the web check (maybe a transient failure)
   C) Abort — try again when web tools are restored
   ```

   - If A: Ask the user for the paper PDF path (or accessible URL) and the
     GitHub repository URL. Pass these directly to Step 1 — Agent A reads the
     provided PDF, Agent B verifies the provided repo URL.
   - If B: Re-run the web check once. If still failing, fall through to A/C.
   - If C: **BLOCKED**. Document the situation and exit.

6. Determine the working directory. Prefer a dedicated `paper_repro/` under the
   project root, or ask the user where.

### Step 1 — Parallel Launch: PaperReader + ResourceFinder

Launch Agent A and Agent B **simultaneously in one message** with two Agent
tool calls. Each agent receives only the context it needs — do not pass the
full conversation history.

**Agent A context to pass:**
- PDF path (if given) or paper title
- Instructions from `references/agent-paper-reader.md`
- Target output: `paper_reproduction_output/paper_reading/extracted_methodology.md`

**Agent B context to pass:**
- Paper title and authors
- Any GitHub URLs already visible in the paper metadata
- Instructions from `references/agent-resource-finder.md`
- Target output: `paper_reproduction_output/paper_reading/resource_map.md`

These two agents are independent — Agent B does not need Agent A's output.

### Step 2 — Validate A+B Outputs

After both agents complete:

1. Read `extracted_methodology.md`. Verify it contains:
   - Method description
   - Dependency list or requirements.txt reference
   - At least one claimed numerical result (for later comparison)
   - "Potential pitfalls" section
2. Read `resource_map.md`. Verify it contains:
   - GitHub repository URL (or explicit "not found")
   - Dataset download URL(s) (or explicit "not found")
   - Dependency installation notes
3. If either output is critically incomplete, re-launch that agent once with
   specific feedback on what's missing.
4. **Paper-Repo cross-validation**: Verify the paper and repo are about the same tool.
   - Read `extracted_methodology.md` → "Reproducibility Notes from Paper" → does
     the paper claim a code URL?
   - Read `resource_map.md` → does the repo's README/description cite the same paper?
   - **Match**: The paper title appears in the repo README, or the repo URL appears
     in the paper's code availability statement → proceed normally.
   - **Mismatch or unverified**: The paper claims repo X but the resource map found
     repo Y, or the repo README cites a different paper. Write a one-line warning to
     `paper_reproduction_output/paper_reference_mismatch.md` with the evidence (what
     the paper claims vs what the repo claims). **Do not block reproduction** — the
     user may have intentionally paired a paper with an alternative implementation.
     Continue with the repo that was actually found and flag the mismatch for Agent F.
   - **Only one source has a URL**: If only the paper or only the repo has a citation,
     note the asymmetry and proceed. Many papers pre-date their code release, and many
     repos don't cite the paper in their README.
5. **Repo selection (when 2+ candidates available)**: If ResourceFinder found multiple
   GitHub repositories for the same paper, the orchestrator selects one using the
   following criteria, applied in priority order:

   | Priority | Criterion | How to check |
   |---|---|---|
   | 1 | **Framework match** | Does the repo use the same framework the paper describes? (PyTorch vs TensorFlow, scikit-learn vs homegrown). Read the repo README and `requirements.txt` / `pyproject.toml`. |
   | 2 | **Dependency file present** | `requirements.txt`, `pyproject.toml`, or `setup.py` exists — signals the environment can be reproduced. |
   | 3 | **CI / verified badge** | Green CI badge on README, or README says "reproduced" or "verified". Signals the code has been tested recently. |
   | 4 | **Recent commit** | Last commit within 2 years. A 6-year-old untouched repo may have bit-rotted dependencies. |
   | 5 | **Original authors** | Repo by paper authors preferred over third-party reimplementation — but only if criteria 1-4 are comparable. |

   Document the selection in a one-line file:
   `paper_reproduction_output/repo_selection.md` with all candidates, scores
   per criterion, and final choice with rationale. Flag any deselected repos
   for Agent F so it can note that results may differ between implementations.

   If no repo passes criteria 1-3 → flag as a risk but continue. The user
   may know something we don't (e.g., the original TF repo still works even
   if it's 6 years old and has no CI).

   **Example**: GAT paper has 3 repos. Diego999/pyGAT scores highest (PyTorch=✓,
   requirements.txt=✓, green CI=✓, updated 2024=✓, third-party=✗). PyG built-in
   scores second (PyTorch=✓, no requirements.txt=✗). Original TF repo scores
   lowest (TF=✓ but 5 years stale=✗, no CI=✗). Select Diego999/pyGAT.

6. **Language confirmation gate**: Read `resource_map.md` → "Dependency Files Found"
   checklist. Confirm the language matches the pre-flight from Step 0. If the
   pre-flight was unknown (only paper title given), this is the definitive detection.
   - R signals: only `DESCRIPTION` + `NAMESPACE` (no `setup.py`/`requirements.txt`),
     README shows `install.packages()`, `devtools::install_github()`, `library()`,
     repo file extensions are `.R`/`.Rmd`/`.rda`
   - Julia signals: `Project.toml` with `[deps]` section (no `pyproject.toml`),
     README shows `using PackageName`, `Pkg.add()`, `.jl` files only
   - MATLAB signals: `.m` files only, no Python files detected
   If non-Python detected → write `reduced_scope.md` if not already done in
   Step 0. Set `language_scope: minimal`. Continue — do NOT block.
   If no code at all (theory paper) → write `scope_violation.md` and **STOP**.

### Step 2.5 — Detect Reproduction Mode

Before launching Agent C, determine the reproduction mode by reading
`resource_map.md`:

**Experiment mode** — the repo is structured as an experiment launcher:
- Contains `main.py`, `train.py`, `run.sh`, `scripts/`, or config files (`.yaml`, `.jsonnet`)
- README shows commands like `python train.py --config ...` or `bash scripts/run.sh`
- No `pip install` or `python setup.py install` as the primary usage path

**Tool mode** — the repo is a pip/conda-installable package:
- Root has `setup.py`, `pyproject.toml` with `[project.scripts]`, or `setup.cfg`
- README shows `pip install X` or `conda install -c conda-forge X` as the first step
- Usage is Python API calls (`import X; X.train()`) rather than shell command experiments
- Repo may have a `demo.py`, `tutorial.ipynb`, or API docs but no `main.py` with --config

Record the decision in a one-line file:
`paper_reproduction_output/reproduction_mode.txt` — content: `tool` or `experiment`.

Pass this mode to Agent C and Agent E so they adapt. Agent C needs it for install
strategy (`pip install -e .` vs `pip install -r requirements.txt`). Agent E needs it
for run strategy (write an API-calling reproduction script vs find and execute
experiment shell commands).

### Step 2.6 — Pre-Environment Gate Checks

Before launching Agent C, check two blocking conditions:

**Gate A: Agent A found no usable paper content.**
If `extracted_methodology.md` contains `## PDF Status: NOT FOUND` or the claims
section is empty with no fallback content → degraded path.
- Write `paper_reproduction_output/<slug>/degraded_mode.md` documenting what was
  tried and why PDF acquisition failed.
- Continue with whatever content IS available (PMC summary, web search, etc.).
  The `Key Claimed Results` section may be sparser than normal — Agent F will
  note this in its comparison.
- Do NOT block — a partial reproduction is better than none.

**Gate B: Agent B found NO usable data sources.**
If the resource map shows ALL datasets marked as `[UNREACHABLE]` or
`[UNVERIFIED]` with zero working download URLs:
- Write `paper_reproduction_output/<slug>/data_unavailable.md`:
  ```markdown
  # Data Unavailable
  ## Paper: {title}
  ## Status: BLOCKED
  All datasets for this paper could not be reached. This is likely a network
  infrastructure issue (GEO blocked, Zenodo timeout, etc.), not a skill bug.
  ## Attempted Sources
  | Dataset | URL | Error |
  |---------|-----|-------|
  ```
- **STOP.** Do not launch Agent C, D, E, or F. There is nothing to reproduce
  without data. Present the `data_unavailable.md` to the user and ask whether
  to proceed on a best-effort basis or pick a different paper.
- If at least ONE dataset has a working URL → proceed normally.

### Step 3 — EnvironmentBuilder

Launch Agent C with:
- `extracted_methodology.md` content
- `resource_map.md` content
- `reproduction_mode.txt` content (so it knows install strategy)
- Instructions from `references/agent-environment-builder.md`
- Target output: `paper_reproduction_output/env_setup/setup_report.md`

**Retry policy:** Agent C now has a built-in structured dependency conflict
resolution loop (see Agent C reference §4c). The agent diagnoses each import
failure by category (ABI / version conflict / missing transitive dep / build
failure / system library), applies a targeted fix to only the failing package,
and re-verifies — up to 5 rounds internally. The orchestrator no longer needs
to re-launch Agent C for routine dependency conflicts.

If Agent C still reports unresolved failures after 5 diagnosis rounds (or if
it crashes/exceeds timeout), re-launch it **once** with the full failure log
and the diagnosis-rounds table from `setup_report.md`. The re-launched agent
should try a fundamentally different strategy (e.g., switch from venv+pip to
conda, or use Docker if available).

If the re-launch also fails → **reproduction failed path**. Write
`reproduction_report.md` with the section "## Reproduction Status: FAILED"
explaining exactly which packages couldn't be installed, what was tried in
each round, and why.

### Step 4 — SmokeTester

Launch Agent D with:
- `setup_report.md` content (to know the venv path and installed packages)
- `resource_map.md` content (to know the repo structure)
- Instructions from `references/agent-smoke-tester.md`
- Target output: `paper_reproduction_output/run_outputs/smoke_test_report.md`

If smoke test fails → **reproduction failed path**. Write
`reproduction_report.md` with "## Reproduction Status: FAILED — Smoke Test"
and the error details.

### Step 4.5 — Quick Benchmark (Time Estimation)

Before committing to a full run, estimate how long it will take. This prevents
the worst reproduction failure mode: a silent multi-hour hang.

Agent D (SmokeTester) should include this as the **last step** of its smoke test.
If Agent D already completed without a benchmark, the orchestrator runs it directly.

**1. Run a 1-unit benchmark** on the smallest dataset from the paper's experiments:

| Paper type | Benchmark unit | How to measure |
|---|---|---|
| Epoch-based (ML) | 1 epoch of training | `time python train.py --epochs 1 --dataset [smallest]` |
| Inference (CV) | 1 inference pass on a 10% subset | `time python infer.py --max-samples N` |
| Cross-validation | 1 fold | `time python cv.py --folds 1` |
| Script/tool | 1 complete run on minimal input | `time python run.py --input [smallest test case]` |

**2. Read the paper to determine total work** — total epochs, folds, datasets.
If the paper doesn't state exact numbers, read the experiment script's defaults.

**3. Estimate**:

```
total_estimated = benchmark_time × total_units × dataset_count × 1.2 (I/O overhead)
```

**4. Decision gate**:

| Estimated total | Action |
|---|---|
| < 5 minutes | Proceed to Step 5 silently |
| 5–10 minutes | Brief note: "Full run estimated at ~N minutes." Proceed |
| 10–60 minutes | Warn via AskUserQuestion: "~N min estimated. Continue or scale down?" |
| > 60 minutes | Strong warn: "~N hours estimated. Consider: (a) smallest dataset only, (b) write a self-contained script for the user to run independently, (c) continue anyway." |

**5. Output**: Append to `smoke_test_report.md`:

```markdown
## Time Estimate
- Benchmark: N.Ns for 1 [unit] on [dataset_name] (H:MM:SS wall clock)
- Total work: E epochs × D datasets = T total units
- Estimated full run: ~N min (range: Nmin–Nmax depending on dataset)
- Decision: [proceed / warn-user / offer-scale-down]
```

If the benchmark itself fails (code crashes during first epoch), log the error
and proceed to Step 5 on a best-effort basis. A failed benchmark does not
block reproduction — it just means we go in blind.

### Step 5 — FullRunner

Launch Agent E with:
- All prior outputs (methodology, resource map, setup report, smoke test report)
- `reproduction_mode.txt` content — so it knows which strategy to use
- Instructions from `references/agent-full-runner.md`
- Target output: `paper_reproduction_output/run_outputs/full_run_report.md`
  and actual output files in `paper_reproduction_output/run_outputs/outputs/`

Agent E should record wall-clock time for the full run. If the run takes
longer than 1 hour, it should checkpoint intermediate results.

**In experiment mode**, Agent E finds and runs the paper's experiment scripts
(`main.py`, `train.py`, shell scripts).

**In tool mode**, Agent E writes a reproduction script that imports the
installed package and calls its API to reproduce each claimed result.
The script goes in `paper_reproduction_output/run_outputs/reproduction_script.py`.
See Agent E's reference file for the full instructions for each mode.

If full run fails partway → **reproduction failed path**. Document which step
failed and any partial results obtained.

### Step 6 — ResultComparator

Launch Agent F with:
- `extracted_methodology.md` (paper's claimed values)
- `full_run_report.md` (actual values from reproduction)
- `tasklog.jsonl` (accumulated structured log from all prior agents — Agent F
  uses this to detect systematic patterns, undiagnosed failures, and broken fix
  strategies; see Agent F reference §6)
- Instructions from `references/agent-result-comparator.md`
- Target output: `paper_reproduction_output/reproduction_report.md`

This is the final synthesis step. Agent F produces the definitive report.

### Step 6.5 — Generate reproduction_manifest.md

After Agent F completes, the orchestrator (you) generates the manifest.
This is a simple markdown table listing every artifact produced — it does NOT
require launching a new agent.

Read the `<paper_slug>` output directory and produce:

```markdown
# Reproduction Manifest — {paper_title}

| File | Generated By | Size | Description |
|------|-------------|------|-------------|
| paper_reading/extracted_methodology.md | Agent A | X KB | Extracted methodology and claims |
| paper_reading/resource_map.md | Agent B | X KB | Code repos, datasets, dependency files |
| env_setup/setup_report.md | Agent C | X KB | Environment setup log + API surface |
| run_outputs/smoke_test_report.md | Agent D | X KB | Smoke test result + time benchmark |
| run_outputs/full_run_report.md | Agent E | X KB | Full reproduction run log |
| run_outputs/outputs/* | Agent E | X MB | Reproduced numerical outputs |
| reproduction_report.md | Agent F | X KB | Paper claims vs reproduced values |
```

Write this to `paper_reproduction_output/<paper_slug>/reproduction_manifest.md`.
This is the index the user opens first to navigate the results.

### Step 7 — Merge and Present

Read `reproduction_report.md`. Present the top-line result to the user:

- **Reproduced ✓**: "≥80% of key metrics match within tolerance. The
  reproduction is usable as a baseline."
- **Data Mismatch ⚠**: "Code runs but metrics deviate from paper claims.
  See the detailed comparison table."
- **Failed ✗**: "Reproduction stopped at step [N]. See the failure analysis."

Then point the user to `reproduction_report.md` for the full report and
`reproduction_manifest.md` for a file inventory.

## TaskLog Protocol

Every agent MUST write structured JSONL entries to
`.claude/tasklog/YYYYMMDD-<session-slug>.jsonl` for key actions. This turns
"一段描述" into machine-analyzable, cross-paper reusable root cause records.

**Schema reference:** `.claude/tasklog/SCHEMA.md` (read only if you need
the full field spec). Agents use a simplified subset below.

### When to write

| Trigger | Action |
|---|---|
| Key step completed successfully | Write one entry with `deviation.type: "none"` |
| Any step fails (error, timeout, unexpected output) | Write immediately — include root cause if known, `null` if still diagnosing |
| Fix applied and verified | Write entry with `fix_worked: true/false` and `lesson` if the pattern is generalizable |
| Agent completed all work | Write one summary entry listing all task_ids covered |

### Minimum fields per entry

```json
{
  "task_id": "kebab-case-id",
  "step": "what was attempted",
  "expected": "what should happen",
  "actual": "what actually happened",
  "deviation": {"type": "none|error|blocker|surprise|slow_path|wrong_turn|missing_context"},
  "root_cause": null,
  "fix": null,
  "fix_worked": null
}
```

Full field spec and deviation type definitions are in `.claude/tasklog/SCHEMA.md`.
Agents: do not read SCHEMA.md unless you need the complete reference — the
subset above is sufficient for routine entries.

### Where to write

All agents write to the same file: `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`.
Append-only, one JSON object per line, UTF-8. Do NOT overwrite previous entries.

### Orchestrator's TaskLog responsibility

After each agent completes, the orchestrator reads the new entries in
`tasklog.jsonl` and checks:
- Any `deviation.type: "error"` with `root_cause: null` → the agent left a
  failure undiagnosed. Ask the agent to fill in the root cause.
- Any `fix_worked: false` → the fix strategy failed. Do not retry the same fix.
- Three or more entries with the same `root_cause` → pattern detected. Flag
  for the final `reproduction_report.md`.

This replaces the previous "一段描述" failure feedback with structured data
that Agent F can use to write a more precise deviation analysis.

## Three Outcome Paths

### Reproduced ✓
- ≥80% of key claimed metrics are within acceptable error (default ±5% for
  accuracy-like metrics, ±0.05 for AUC-like metrics; override per domain).
- Write final status as "Reproduced" with a confidence assessment.
- The output directory is ready to serve as a baseline for future work.

### Data Mismatch ⚠
- Code runs to completion but one or more key metrics deviate significantly.
- Report: which metric, paper value, reproduced value, absolute and relative
  deviation, the first point in the pipeline where the deviation was observed.
- Flag whether the deviation direction (better/worse) is consistent or
  inconsistent across metrics.

### Failed ✗
- Any of Steps 3-5 cannot complete after retries.
- Report: which step failed, full error trace, what was tried, and a concrete
  suggestion for what the user can try manually.
- Partial outputs (if any) are preserved in the output directory.

## Platform-Specific Handling

These are baked into Agent C and Agent E's instructions. You don't need to
handle them at the orchestrator level, but verify they were addressed:

| Issue | Handling |
|---|---|
| GitHub inaccessible | Agent B: try Gitee mirrors, ghproxy.com |
| Python/pip version mismatch | Agent C: `python -m pip` to ensure consistency |
| pip default index unreachable | Agent C: auto-switch to `https://pypi.tuna.tsinghua.edu.cn/simple` |
| conda channels unreachable | Agent C: switch to Tsinghua conda mirrors, rewrite `environment.yml` channels |
| Windows GBK encoding | All generated Python scripts: `sys.stdout.reconfigure(encoding='utf-8')` |
| Incomplete README | Agent B + D: also check GitHub Issues, Wiki, and source code directly |

## Reference Files

| File | Read When |
|---|---|
| `references/agent-paper-reader.md` | Launching Agent A |
| `references/agent-resource-finder.md` | Launching Agent B |
| `references/agent-environment-builder.md` | Launching Agent C |
| `references/agent-smoke-tester.md` | Launching Agent D |
| `references/agent-full-runner.md` | Launching Agent E |
| `references/agent-result-comparator.md` | Launching Agent F |

Read each reference file only when you are about to launch the corresponding
agent. Pass the full content as part of the agent's task description.
