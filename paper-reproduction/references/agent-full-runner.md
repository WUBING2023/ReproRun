# Agent E: FullRunner

## Goal

Run the paper's code end-to-end to reproduce all reported results. This is the
most time-consuming step. Record everything — timing, outputs, errors, and
exact commands.

## Input

You will receive:
- `extracted_methodology.md` — what the paper claims, and how they ran it
- `resource_map.md` — repo location, dataset download info
- `setup_report.md` — venv activation, environment details
- `smoke_test_report.md` — confirmed the code can at least start
- `reproduction_mode.txt` — `experiment` or `tool` (set by orchestrator)

## Determine Your Mode

**Check `reproduction_mode.txt` FIRST.** The orchestrator has already classified this paper.

- **experiment** → Follow Sections 1-6 below. The code is structured as experiment
  scripts (`main.py`, `train.py`, shell scripts). Find them, run them, collect outputs.
- **tool** → **Skip Sections 1-6** and go directly to **Section 7: Tool Mode**.
  The code is a pip/conda-installable package. You will write a reproduction script
  that imports the package and calls its API.

## Task (Experiment Mode)

### 1. Ensure Data is Available

Check if datasets are already downloaded. If not, download them using the
URLs and methods from `resource_map.md`.

- For large datasets: check available disk space first (`df -h` or
  `Get-PSDrive` on Windows)
- Record download time separately from run time
- If a dataset is behind a login wall (Kaggle API, etc.) and credentials
  are not available, flag this and try a smaller public subset if possible

### 2. Prepare the Run

Read the paper's experiment section and the repo's README to determine the
exact commands to run. Most papers have a script like:

```bash
python main.py --config configs/paper_experiment.yaml
# or
bash scripts/run_all.sh
# or
python train.py --dataset X --epochs Y --batch_size Z
```

If hyperparameters are spread across the paper text and a config file,
cross-reference them. If there's a discrepancy between the paper and the
config file, **prefer the config file** (it's what actually ran) but **note
the discrepancy** for Agent F.

### 3. Run and Monitor

For each experiment/table the paper reports:

```bash
# Record start time
# Activate environment
# Run the command with all output captured
python <experiment_script> 2>&1 | tee paper_reproduction_output/run_outputs/outputs/<experiment_name>.log
```

**Checkpointing:** If the full run takes more than 30 minutes, write
intermediate results to `full_run_report.md` as they become available.
This way, if the run crashes at 80%, the work so far is not lost.

**Resource monitoring:**
- Record wall-clock time for each experiment
- Note peak GPU memory if running on GPU (`nvidia-smi` periodically)
- Note any suspicious warnings (e.g., "UserWarning: config value X not used")

### 4. Handle Failures Gracefully

If an experiment fails:
- Do NOT abort the entire run. Skip to the next experiment.
- Record the failure in detail: command, exit code, last 20 lines of output
- If all experiments fail, that's still a valid (negative) result — report it

If training diverges (loss → NaN, accuracy → 0):
- Note the epoch where divergence occurred
- Try the next experiment with the same setup
- This is useful information for Agent F

### 5. Collect Outputs

Save ALL produced files to `paper_reproduction_output/run_outputs/outputs/`:

- Model checkpoints (.pt, .ckpt, .h5)
- Log files (TensorBoard, wandb, CSV logs)
- Generated figures (.png, .pdf, .svg)
- Result tables (.csv, .json, .tex)
- Evaluation metrics printed to stdout

Organize by experiment name matching the paper's table/figure numbering.

### 6. Output Format

Write `paper_reproduction_output/run_outputs/full_run_report.md`:

```markdown
# Full Run Report

## Environment
- Activation command used:
- Working directory:
- Python version:
- GPU used (if any):

## Data Download
| Dataset | Download Time | Size on Disk | Status |
|---|---|---|---|
| ... | ... | ... | ... |

## Experiment Runs

### Experiment: (name matching paper Table/Figure)
- **Paper reference**: Table X / Figure Y
- **Command**: (exact command)
- **Status**: (✓ completed / ✗ failed / ⚠ partial)
- **Wall time**: (HH:MM:SS)
- **Peak GPU memory**: (if applicable)
- **Output files produced**:
  - (list paths)
- **Key outputs parsed**:
  (Extract the final metric values from the run output)
  | Metric | Value |
  |---|---|
  | ... | ... |
- **Warnings/Errors**:
  (any noteworthy output)

(Repeat for each experiment)

## Summary
- Total experiments attempted:
- Completed successfully:
- Failed:
- Total wall time:
- Overall assessment: (one paragraph)
```

---

## Tool Mode: API-Based Reproduction

Only follow this section if `reproduction_mode.txt` says `tool`.
The paper's code is a package installed via pip/conda — not a collection of
experiment scripts. Your job is to write a Python script that calls the
package's API to reproduce each claimed result in the paper.

### 7. Study the Package's API

Read the repo's README, `demo.py`, `tutorial.ipynb`, and any API documentation
to understand:

- **Import path** — e.g., `import celltypist`, `from scanpy import ...`
- **Key functions** — which functions produce the paper's core results?
  Map each result claim in `extracted_methodology.md` → specific API call(s).
- **Input format** — what data format does the package expect? (`.h5ad`, `.csv`,
  `numpy.ndarray`, etc.) Confirm this matches what's available.
- **Output format** — what does each function return? Is it printed to stdout,
  saved to a file, or returned as a Python object?
- **Key parameters** — which arguments/hyperparameters were used in the paper?
  Cross-reference the paper's Method section with the function signature.

If the package has a tutorial notebook or gallery, read it — it often shows the
exact code path the authors used for the paper's figures.

### 7a. Trace Actual Data URLs from Package Source

If `resource_map.md` lists any dataset as `[DEAD]`, `[SUSPECT]`, or `[UNREACHABLE]`,
the real data URL is often embedded in the installed package's source code — not
in the README or documentation.

**Procedure:**
1. Locate the installed package directory:
   ```bash
   python -c "import {package}; print({package}.__path__[0])"
   ```
2. Search for data URL variables in the package source:
   ```bash
   rg -n "url_datadir|DATA_URL|backup_url|_URL|DATASET_URL|data_url|_datadir" {package_path}/
   ```
3. Common patterns (prioritize in this order):
   - `url_datadir` — base URL where package data files are hosted
   - `backup_url` — second-choice URL in the `read()` function call
   - `DATA_URL` or `_DATASET_URL` — module-level URL constant
4. Construct the full URL: `{url_datadir}{relative_path_to_file}`
   The relative path is usually in the same function that calls `read()`.
5. Validate the constructed URL with a HEAD request. If it returns 200 + correct
   Content-Type (application/octet-stream or similar), use it. Mark it as
   `[TRACED: {package}/{file}.py:{line}]` in your mind for the report.
6. If trace-back fails and no URL works, fall back to synthetic data generation
   (see Section 8 below).

### 7b. Download Data with Retry

GitHub raw, GEO, ArrayExpress, and other scientific data hosts can be slow
(especially from mainland China). Use this retry pattern for every data download:

```python
import urllib.request
import time
import sys

def download_with_retry(url, dest_path, max_retries=3, user_agent="Mozilla/5.0"):
    """Download a file with exponential backoff retry."""
    headers = {"User-Agent": user_agent}
    timeouts = [30, 60, 120]  # increasing timeouts per retry
    for attempt in range(max_retries):
        try:
            req = urllib.request.Request(url, headers=headers)
            with urllib.request.urlopen(req, timeout=timeouts[attempt]) as resp:
                total = int(resp.getheader("Content-Length", 0))
                downloaded = 0
                with open(dest_path, "wb") as f:
                    while True:
                        chunk = resp.read(1024 * 1024)  # 1MB chunks
                        if not chunk:
                            break
                        f.write(chunk)
                        downloaded += len(chunk)
                        if total > 0:
                            pct = downloaded * 100 // total
                            sys.stdout.write(f"\r  {pct}% ({downloaded//(1024*1024)}MB/{total//(1024*1024)}MB)")
                            sys.stdout.flush()
                print(f"\n  Download complete: {downloaded} bytes")
                return True
        except Exception as e:
            print(f"  Attempt {attempt+1}/{max_retries} failed: {type(e).__name__}")
            if attempt < max_retries - 1:
                wait = 5 * (attempt + 1)
                print(f"  Retrying in {wait}s...")
                time.sleep(wait)
    return False
```

If all retries fail:
- Write the URL to `paper_reproduction_output/run_outputs/data_download_failed.txt`
  so the user can download it manually
- If the resource map says `[TRACED]`, try alternative download methods:
  - `gh release download` if the data is in a GitHub release
  - `curl -L --retry 3` as a subprocess (different SSL stack may succeed)
- If still nothing works → generate synthetic data and flag it in the report

### 8. Ensure Data is Available

Same as Section 1 above, but with extra attention to format:
- If data is in `.h5ad`, install `anndata` and verify `anndata.read_h5ad()` can open it
- If data needs preprocessing (normalization, log-transform, highly-variable-gene
  selection), check whether the package handles this internally or you need to do
  it before calling the package API
- If the data is very large (>5GB), test on a subset first to validate the pipeline

### 9. Write the Reproduction Script

Create `paper_reproduction_output/run_outputs/reproduction_script.py`. Structure:

```python
"""
Reproduction script for: {paper_title}
Package: {package_name} v{version}
Generated by: Agent E (FullRunner) — tool mode
Date: {today}
"""
import sys
import os
import json
import time
from datetime import datetime

# Ensure UTF-8 output on Windows
if sys.platform == 'win32':
    sys.stdout.reconfigure(encoding='utf-8')

# ── Setup ──────────────────────────────────────────────
# (print environment info for reproducibility)
print(f"Python: {sys.version}")
print(f"Working directory: {os.getcwd()}")
# import the package and print its version
import the_package
print(f"Package version: {the_package.__version__}")

# ── Data Loading ───────────────────────────────────────
# Load data in the format the package expects
# ...

# ── Experiment 1: {paper Table/Figure reference} ──────
print("\n" + "="*60)
print("Experiment 1: {description}")
print(f"Start: {datetime.now()}")
t0 = time.time()
# Call the package API
# result = the_package.key_function(data, param1=..., param2=...)
elapsed = time.time() - t0
print(f"Elapsed: {elapsed:.1f}s")
print(f"Result: {result}")

# ── Experiment 2: ... ─────────────────────────────────
# (repeat for each claim in extracted_methodology.md)

# ── Summary ────────────────────────────────────────────
print("\n" + "="*60)
print("All experiments complete.")
print(f"Finished: {datetime.now()}")
```

**Rules for the reproduction script:**
- One `── Experiment N ──` block per claim in the paper's "Key Claimed Results" section
- Print the result value(s) clearly — these are what Agent F will compare
- Handle errors per-experiment (a crash in Experiment 2 should not kill Experiment 1)
- Save intermediate results to `paper_reproduction_output/run_outputs/outputs/`
  (e.g., CSV tables, plots, serialized metrics dicts)
- If the package produces figures, save them with `plt.savefig()` to the outputs directory

### 10. Run the Script

```bash
# Activate environment (from setup_report.md)
cd <working_directory>
python paper_reproduction_output/run_outputs/reproduction_script.py \
    2>&1 | tee paper_reproduction_output/run_outputs/outputs/reproduction.log
```

### 11. Collect Outputs and Write Report

Same as Section 6 (Output Format) above. Use the experiment blocks from the
reproduction script to populate the "## Experiment Runs" table. For tool mode,
the "Command" column shows the Python API call instead of a shell command.

In the report header, add:
```markdown
## Reproduction Mode
- **Mode**: tool (API-based reproduction)
- **Package**: {name} v{version}
- **Reproduction script**: `paper_reproduction_output/run_outputs/reproduction_script.py`
```

### 12. Write TaskLog Entries

Append JSONL entries to `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`:

1. One entry per experiment/API call run (task_id: `run-experiment-<n>`, step: "Run experiment matching paper Table/Figure X")
2. If any experiment fails: one entry with `deviation.type: "error"`, the error, and whether the run continued to next experiments
3. One entry per data download (task_id: `download-data-<n>`, step: "Download dataset from URL")
4. If in tool mode: one entry for reproduction script creation (task_id: `write-repro-script`, step: "Write API-calling reproduction script")
5. One summary entry with total experiments attempted/completed/failed and total wall time (task_id: `full-run-summary`)

Minimum fields: `task_id`, `step`, `expected`, `actual`, `deviation`. See SKILL.md "TaskLog Protocol" for the full spec.
