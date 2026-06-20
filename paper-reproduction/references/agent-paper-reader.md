# Agent A: PaperReader

## Goal

Read the target paper and extract everything needed for reproduction:
methodology, dependencies, and claimed numerical results. You are the sole
source of truth for what the paper actually claims — every later comparison
depends on your accuracy.

## Input

You will receive ONE of:
- A PDF file path (read it with the PDF reading tools available)
- A paper title and/or authors (search for the PDF on arXiv / OpenReview /
  semanticscholar / Google Scholar, then download and read it)

## Task

### 1. Read and Understand

Read the full paper. If the paper has supplementary material, read that too.
Pay special attention to:
- **Method section**: algorithm descriptions, model architecture, training
  procedure, hyperparameters
- **Experiment section**: datasets used, evaluation protocol, metrics reported
- **Results tables**: every numerical claim (main results, ablation, comparison
  tables)

### 2. Extract Methodology

Write a structured methodology summary:

```markdown
## Paper Identity
- Title:
- Authors:
- Venue/Year:
- arXiv ID (if any):

## Method Summary
(3-5 sentence plain-language summary of what the paper does)

## Algorithm / Model Architecture
(Detailed description. Include diagram text descriptions, layer dimensions,
loss functions, training procedure. Be specific enough that someone can
re-implement from this description alone.)

## Hyperparameters
| Parameter | Value | Paper Section |
|---|---|---|
| ... | ... | ... |

## Datasets Used
| Dataset | Role (train/val/test) | Size | Source |
|---|---|---|---|
| ... | ... | ... | ... |

## Dependencies Mentioned
- Python packages: (list all mentioned: torch, transformers, numpy, etc.)
- System dependencies: (CUDA version, system libraries, etc.)
- External tools: (any non-Python dependencies)

## Key Claimed Results
(Extract EVERY numerical result that can be compared against. Be precise.)
| Metric | Value | Dataset | Context (which table/figure) | Comparison Baseline |
|---|---|---|---|---|
| ... | ... | ... | Table X / Figure Y | ... |

## Reproducibility Notes from Paper
- Does the paper claim to have released code? URL?
- Does the paper claim to have released data? URL?
- Are there any known errata or corrections?
- Appendix/supplementary with additional implementation details?

## Potential Pitfalls
(Mark anything that looks like it could cause trouble during reproduction.)
- [ ] (e.g.) "Authors mention 'standard data augmentation' without specifying"
- [ ] (e.g.) "Hyperparameter θ is only given for Dataset A, not B"
- [ ] (e.g.) "Uses a custom CUDA kernel — may not compile on all GPUs"
- [ ] (e.g.) "Random seed not specified for key experiment"
```

### 3. Save Output

Write this to `paper_reproduction_output/paper_reading/extracted_methodology.md`.

### 4. Write TaskLog Entries

Append JSONL entries to `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`:

1. One entry for PDF acquisition (task_id: `find-pdf`, step: "Search and download paper PDF", deviation type: `none` if found, `blocker` if not found)
2. One entry for methodology extraction (task_id: `extract-methodology`, step: "Extract methodology and claims from paper")
3. If any claimed result is ambiguous or incomplete, one entry with `deviation.type: "surprise"`

Minimum fields: `task_id`, `step`, `expected`, `actual`, `deviation`. See SKILL.md "TaskLog Protocol" for the full spec.

## Quality Bar

Before finishing, verify:
- [ ] At least 3 key claimed results with exact values extracted
- [ ] All mentioned Python packages listed in the dependencies section
- [ ] At least 2 potential pitfalls identified
- [ ] Dataset names, sizes, and roles are explicit

If you cannot find the PDF after searching, write a clear error in
`extracted_methodology.md` under "## PDF Status: NOT FOUND" with the search
terms you tried.
