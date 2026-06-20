# Agent D: SmokeTester

## Goal

Run the smallest possible "hello world" of the paper's code to verify the
environment works and the code isn't fundamentally broken. This is NOT a full
reproduction — just a sanity check. 5 minutes max, ideally 30 seconds.

## Input

You will receive:
- `setup_report.md` — venv path, activation command, installed packages
- `resource_map.md` — repo URL, README contents

## Task

### 1. Find the Minimal Entry Point

Check these locations in order:

1. **README Quick Start section** — most repos have a 2-3 line "Quick Start"
   or "Getting Started" example
2. **`demo.py` / `example.py` / `quickstart.py`** — look for files with demo
   or example in the name
3. **`tests/` directory** — a minimal unit test that instantiates the model
4. **`__main__` block** — check `if __name__ == "__main__"` in the main module
5. **Jupyter notebook** — `demo.ipynb` or `tutorial.ipynb` with a simple
   first cell

If multiple entry points exist, pick the simplest one — the one that requires
the fewest arguments and downloads the least amount of data.

### 2. If No Entry Point Found

Write one yourself. A good smoke test does the minimum possible thing:

- For an ML model: instantiate the model class with default parameters, run
  one forward pass with random input of the correct shape
- For a data analysis paper: load a tiny subset of data, run one core function
- For a training script: run 1 epoch on 10 samples with minimal config

The self-written smoke test goes in:
`paper_reproduction_output/run_outputs/smoke_test.py`

### 3. Run the Smoke Test

```bash
# Activate the environment first (from setup_report.md)
# Then:
cd <repo_directory>
python <smoke_test_entry_point>
```

Capture stdout and stderr. The test passes if:
- Exit code is 0
- No ImportError or ModuleNotFoundError
- No CUDA out-of-memory (if GPU is needed, use smallest possible batch)

If the test takes more than 5 minutes, it's not a smoke test — kill it and
report that no lightweight entry point was found.

### 4. Troubleshoot Common Failures

| Symptom | Likely Cause | Fix |
|---|---|---|
| `ModuleNotFoundError: No module named 'X'` | Missing package | `pip install X` and retry. If the error came from a function call (not an import), install and retry the SAME single line — don't restart the whole test. |
| `FileNotFoundError: .../data/...` | Data not downloaded | Download a minimal sample or mock the data |
| `CUDA out of memory` | GPU too small | Set `batch_size=1`, use CPU mode if available |
| `AttributeError: module 'X' has no attribute 'Y'` | API version mismatch | Check which version the paper used vs installed |
| `KeyError: 'some_config_key'` | Missing config file | Check for default configs in the repo |
| **Training output floods stdout** | The package prints per-batch loss to stdout (very common in ML repos) | Suppress: wrap the training call with `sys.stdout = open(os.devnull, 'w')` / `sys.stdout = old`. Only suppress during training — keep stderr visible so real errors aren't hidden. |

For each failure, try ONE fix and retry. Don't go down rabbit holes — the smoke
test is a gate, not a debugging session.

### 5. Output Format

Write `paper_reproduction_output/run_outputs/smoke_test_report.md`:

```markdown
# Smoke Test Report

## Entry Point
- File: (which script was run)
- Source: (from README / self-written / from tests/)
- Command: (exact command used)

## Result
- **Status**: (✓ PASSED / ✗ FAILED)
- **Runtime**: (seconds)
- **Exit code**:

## Output (first 50 lines)
```
(paste stdout/stderr — truncate if too long)
```

## Errors Encountered
| Error | Attempted Fix | Result |
|---|---|---|
| ... | ... | ... |

## Self-Written Smoke Test
(If you wrote one, paste it in a code block so the orchestrator can review it.)

```python
# smoke_test.py
...
```

## Assessment
(One sentence: is the environment ready for full reproduction?)

## Time Benchmark (1 unit → full run estimate)
(exact time in seconds, format: "Xs" or "Xmin Ys")
```

### 4.5 — Quick Time Benchmark

After the smoke test passes, run ONE minimal unit of the paper's workload
to estimate how long the full reproduction will take. This prevents the user
from launching Agent E into a 2-hour reproduction without warning.

**How to benchmark:**

1. Identify the smallest reproducible unit: 1 epoch of training, 1 fold of
   cross-validation, 1 dataset subset, or 1 analysis function call.
2. Time it precisely:
   ```python
   import time
   start = time.time()
   # ... run the 1-unit workload ...
   elapsed = time.time() - start
   print(f"1-unit time: {elapsed:.0f}s")
   ```
3. Count how many units the full reproduction needs (from the paper/README):
   - Training: # epochs × # folds
   - Cross-validation: # folds × # repeats
   - Data analysis: # datasets × # functions
4. Estimate total: `1-unit time × total units × 1.5 (overhead buffer)`
5. Write the estimate into the smoke test report under a new section:

```markdown
## Time Benchmark
| Metric | Value |
|--------|-------|
| 1-unit workload | (what you timed: "1 epoch retina/batch=256") |
| 1-unit time | Xs |
| Total units | N (Y epochs × Z folds) |
| Estimated full run | X min Y s |
```

**If the estimated time exceeds 60 minutes**, add a prominent warning so the
user can decide whether to proceed with the full run or try a smaller scope.

### 6. Write TaskLog Entries

Append JSONL entries to `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`:

1. One entry for entry point discovery (task_id: `find-entry-point`, step: "Find minimal smoke test entry point")
2. One entry for the smoke test run (task_id: `smoke-test-run`, step: "Run smoke test and verify exit code 0")
3. If smoke test fails: one entry per error with `deviation.type: "error"`, attempted fix, and result
4. One entry for the time benchmark (task_id: `time-benchmark`, step: "Run 1-unit benchmark for full run estimation")

Minimum fields: `task_id`, `step`, `expected`, `actual`, `deviation`. See SKILL.md "TaskLog Protocol" for the full spec.
