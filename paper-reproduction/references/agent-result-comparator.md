# Agent F: ResultComparator

## Goal

Compare the paper's claimed results against the actual reproduction outputs.
This is the final synthesis step — your report is the definitive output of the
entire reproduction pipeline. Be precise, honest, and careful.

## Input

You will receive:
- `extracted_methodology.md` — Section "Key Claimed Results" (paper's claims)
- `full_run_report.md` — actual values from the reproduction run

## Task

### 1. Build the Comparison Table

For EVERY claimed metric in the paper that has a corresponding reproduced
value, build a comparison row:

```markdown
| # | Metric | Dataset | Paper Value | Reproduced Value | Abs. Diff | Rel. Diff | Tolerance | Verdict |
|---|---|---|---|---|---|---|---|---|
| 1 | Accuracy | CIFAR-10 | 95.2% | 94.8% | 0.4% | 0.42% | ±1% | ✓ Match |
| 2 | F1-score | SQuAD | 88.5 | 85.1 | 3.4 | 3.8% | ±2% | ⚠ Deviation |
| 3 | AUC | ChestX | 0.92 | — | — | — | ±0.05 | ✗ Not comparable |
```

**Tolerance defaults by domain** (override if the paper or domain has established norms):

| Domain | Metrics | Default Tolerance |
|---|---|---|
| **General ML / Vision / NLP** | Accuracy, Precision, Recall, F1 | ±1 percentage point |
| | AUC, AUROC, AUPRC | ±0.02 |
| | BLEU, ROUGE | ±1.5 points |
| | RMSE, MAE | ±5% of paper value |
| | Perplexity | ±5% of paper value |
| **Bioinformatics / Single-cell** | Cell-type-level F1, Precision, Recall | ±3 percentage points (biological variability is higher) |
| | Adjusted Rand Index (ARI), NMI | ±0.05 |
| | Cross-batch / cross-dataset consistency | ±5 percentage points |
| | AUC (cell-type classification) | ±0.03 |
| | Gene expression correlation (Pearson r) | ±0.05 |
| **Tabular / Structured Data** | Accuracy, F1 | ±1 percentage point |
| | Log-loss | ±0.05 |
| | MAE, RMSE, R² | ±5% of paper value |
| **Runtime / Throughput** | Any timing metric | ±20% (hardware differences dominate) |

**Domain detection:** Check `extracted_methodology.md` for domain indicators:
- "cell type", "scRNA-seq", "single-cell", "gene expression", "transcriptome" → bioinformatics
- "CIFAR", "ImageNet", "COCO", "BERT", "Transformer", "GLUE", "supervised/unsupervised" → general ML
- "tabular", "regression", "classification on structured data" → tabular

**How tolerances work:** the reproduced value must satisfy
`|reproduced − paper| ≤ tolerance`. A metric at exactly the tolerance boundary
counts as a match.

### 2. Three-Category Classification

For each row, assign exactly ONE verdict:

**✓ Match** — The reproduced value falls within the tolerance range of the
paper's claimed value. "Within tolerance" means `|repro - paper| ≤ tolerance`.

**⚠ Deviation** — The reproduced value exists but falls outside tolerance.
Report the direction: "reproduced value is LOWER than paper claim" or
"HIGHER than paper claim". Note if the direction is consistent across
multiple metrics (e.g., "all reproduced accuracies are 2-4% lower").

**✗ Not Comparable** — The value could not be reproduced. Reasons include:
- The corresponding experiment failed to run
- The metric was computed on a different dataset split or different protocol
- The paper didn't specify enough detail to reproduce this exact number
- This metric requires proprietary hardware/data not available

### 3. Identify the First Divergence Point

If the results deviate from the paper, trace back to find WHERE the
divergence first appeared:

1. Did a different random seed produce different results? (expected, not
   concerning if within tolerance)
2. Did the training loss curve diverge at a specific epoch?
3. Was there a hyperparameter mismatch between paper and config?
4. Was the data preprocessing different? (normalization, augmentation, split)
5. Is there a fundamental difference? (e.g., paper used 8×V100 but we used
   1×RTX 3090 — batch size and effective learning rate differ)

### 4. Compute Overall Verdict

Count the comparison rows:

| Outcome | Condition |
|---|---|
| **Reproduction Successful** | ≥80% of comparable metrics are "✓ Match" |
| **Data Contradiction** | Code runs but <80% of comparable metrics match, AND at least one metric shows a statistically meaningful deviation |
| **Reproduction Failed** | Fewer than 3 metrics could be compared (most experiments didn't run), OR the orchestrator already marked failure in an earlier step |

### 5. Write the Final Report

Write `paper_reproduction_output/reproduction_report.md`:

```markdown
# Reproduction Report

## Paper Identity
- Title:
- Authors:
- Venue/Year:
- Reproduction date:

## Overall Verdict

**Status: [✓ Reproduced / ⚠ Data Contradiction / ✗ Failed]**

(One paragraph summary. If successful, state confidence. If contradictory,
state the core finding. If failed, state the blocking step.)

## Key Metrics Comparison

| # | Metric | Dataset | Paper | Reproduced | Diff | Verdict |
|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... |

**Summary:** X/Y metrics match (Z%), W metrics deviate, V not comparable.

## Deviation Analysis (if any)

### Metric: (name) — (paper value) vs (reproduced value)
- Absolute difference: ...
- Relative difference: ...%
- Direction: (higher / lower)
- First divergence point: (where in the pipeline this deviation appeared)
- Possible causes:
  1. (ranked by likelihood)
  2. ...

## Experiments That Failed (if any)
| Experiment | Error | Likely Cause |
|---|---|---|
| ... | ... | ... |

## Reproducibility Assessment

### What Worked
- (bullet list of things that went smoothly)

### What Didn't Work
- (bullet list of problems, with severity)

### Recommendations for Someone Trying to Reproduce This Paper
(3-5 concrete, actionable suggestions. E.g., "Pin torch==1.13.0, not 1.13.1
— the latter breaks the custom CUDA extension", "Download the dataset BEFORE
running, the script doesn't auto-download and fails silently")

## Files Produced
- Full comparison table: (path)
- Raw experiment outputs: paper_reproduction_output/run_outputs/outputs/
- Environment details: paper_reproduction_output/env_setup/

## Reproduction Checklist
- [ ] Code repository found and cloned
- [ ] Environment built and all imports verified
- [ ] Smoke test passed
- [ ] All datasets downloaded
- [ ] Full experiments completed (or documented as failed)
- [ ] Results compared against paper claims
- [ ] Discrepancies analyzed
```

### 6. Write TaskLog Entries

Append JSONL entries to `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`:

1. One entry per comparison row (task_id: `compare-metric-<n>`, step: "Compare paper claim vs reproduced value for metric X")
2. If ≥3 metrics deviate in the same direction: one entry with `deviation.type: "surprise"`, noting the systematic bias
3. One entry for the overall verdict (task_id: `overall-verdict`, step: "Compute final reproduction verdict", expected: "Reproduced/Data Contradiction/Failed")

Additionally, read ALL prior entries in `.claude/tasklog/` from other agents. Look for:
- Patterns: same `root_cause` appearing ≥3 times → flag in the "Systematic Issues" section of the final report
- Unresolved entries: `deviation.type != "none"` with `root_cause: null` → note as "undiagnosed failure" in the report
- Fix effectiveness: `fix_worked: false` appearing ≥2 times → broken fix strategy, flag for recommendation

These patterns feed into the "Recommendations for Someone Trying to Reproduce This Paper" section.
