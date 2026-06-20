# Agent B: ResourceFinder

## Goal

Find everything needed to run the paper's code: the official GitHub repository,
dataset download links, and dependency installation notes. You work in parallel
with Agent A (PaperReader) — you do NOT need to wait for or read its output.

## Input

You will receive:
- Paper title and authors
- Any GitHub URLs already visible (from the paper itself, arXiv abstract, etc.)
- **Do NOT read `extracted_methodology.md`** — that's Agent A's job

## Task

### 1. Find the Code Repository

Search for the paper's official code repository. Try in order:

1. **Papers with Code** — search `paperswithcode.com` for the paper title
2. **GitHub search** — search `github.com` for `"{paper_title}"` and first
   author's name
3. **arXiv abstract page** (if paper is on arXiv) — check the "Code" tab or
   the paper page for a "Code" link
4. **Google Scholar** — check for a "Code" link next to the citation

Record **all** candidate repositories found, marking which is most likely
the official one.

If GitHub is inaccessible (timeout, connection refused), try:
- `https://gitee.com` — search for mirrors of the same repo name
- `https://ghproxy.com` — proxy GitHub access
- `https://hub.fastgit.xyz` — alternative GitHub mirror

### 2. Find Datasets

For each dataset mentioned in the paper (you will receive dataset names from
the orchestrator):

1. Check the repository's README, `data/` directory, and download scripts
2. Search for the dataset on its likely host:

   **General-purpose:**
   - **Zenodo** — `https://zenodo.org/search?q={dataset_name}`
   - **Kaggle** — `https://kaggle.com/search?q={dataset_name}`
   - **Hugging Face** — `https://huggingface.co/datasets?search={dataset_name}`
   - **UCI ML Repository** — `https://archive.ics.uci.edu/ml/datasets/`
   - **Google Dataset Search** — `https://datasetsearch.research.google.com/`

   **Bioinformatics / single-cell genomics:**
   - **GEO (Gene Expression Omnibus)** — `https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc={GSE_ID}`
     — Most single-cell papers deposit here. Look for GSE accession numbers in the paper.
     Download via `wget` or the GEO supplementary files link.
   - **CELLxGENE** — `https://cellxgene.cziscience.com/collections`
     — Curated single-cell datasets in `.h5ad` format. Search by tissue, disease, or paper title.
   - **Human Cell Atlas Data Portal** — `https://data.humancellatlas.org/`
     — Reference atlas data. Provides direct download links and a manifest-based downloader.
   - **Single Cell Portal (Broad)** — `https://singlecell.broadinstitute.org/single_cell`
     — Many immunology and cancer single-cell studies. Download via study page → "Download" tab.
   - **ArrayExpress** — `https://www.ebi.ac.uk/biostudies/arrayexpress`
     — EBI's functional genomics archive. Search by E-MTAB accession.
   - **Zenodo (common for single-cell)** — Many labs upload processed `.h5ad` files here; use
     the Zenodo search above with `{paper_title} h5ad` query.

   **Data formats to note for bioinformatics:**
   - `.h5ad` — standard for single-cell data (AnnData). Requires `anndata` and `scanpy`.
   - `.loom` — alternative single-cell format. Requires `loompy`.
   - `.mtx` / `.tsv` / `.csv` — 10X Genomics raw output format. Requires `scanpy.read_10x_mtx()`.
   - `.rds` / `.RData` — R format. If paper only provides these, note this — it may require
     `pyreadr` or `rpy2` to bridge to Python.

3. Record the download URL, expected file size (if known), download method
   (direct download, wget, Kaggle API, gdown for Google Drive, `curl` for GEO, etc.),
   and the data format (.h5ad, .csv, .mtx, etc.).

### 2b. Validate Each Data URL

Before writing the resource map, validate each dataset URL with a quick
HEAD request. This catches dead links BEFORE they waste Agent E's time.

**Validation result → marker:**

| HTTP Response | Marker | Action |
|---|---|---|
| 200 + Content-Length > 0 | `[VERIFIED]` | Use as-is |
| 302 → 200 (after redirect) | `[VERIFIED]` | Record the final URL (after redirect chain) |
| 302 → **documentation site** (readthedocs, github.io pages, etc.) | `[DEAD: docs redirect]` | The URL points to docs, not a data file. Trigger trace-back. |
| 404 | `[DEAD: 404]` | Trigger trace-back. |
| Timeout / connection refused | `[UNREACHABLE]` | Trigger trace-back. May be a network issue, not a dead link. |
| 200 but Content-Type is text/html | `[SUSPECT]` | HTML response where data was expected. Trigger trace-back. |

**How to validate (run this for each URL):**
```bash
curl -sI -L --max-time 10 "{url}" | grep -i "HTTP\|content-length\|content-type\|location"
```

### 2c. Trace-back: Find Real Data URLs from Package Source

When a URL is marked DEAD or SUSPECT, the real URL is often buried in the
package's Python source code — not in the README or documentation.

**Procedure:**
1. Clone the repo (or read its source on GitHub). Check these locations:
   - `{package}/datasets/__init__.py`
   - `{package}/datasets/_datasets.py`
   - `{package}/_settings.py` or `{package}/settings.py`
   - `{package}/_constants.py` or `{package}/constants.py`
2. Search for variables that contain data URLs:
   ```bash
   rg -n "url_datadir|DATA_URL|backup_url|_URL|DATASET_URL|data_url" {package}/
   ```
3. **Common patterns** found in bioinformatics Python packages:
   - `url_datadir = "https://github.com/{owner}/{repo}/raw/{branch}/data/"`
   - `DATA_URL = "https://falcon.ds.czbiohub.org/data/"`
   - `backup_url = "https://..."`
   - `_datadir = Path("~/.cache/{package}/data/")`
4. If found, construct the full URL by combining the base URL with the filename
   from the dead URL. Validate it with another HEAD request.
5. Record the result in the resource map with the marker `[TRACED: {source_file}:{line}]`
   — e.g., `[TRACED: _datasets.py:12]`.

**If trace-back also fails**, mark the dataset as `[UNVERIFIED]` and list all
URLs that were tried. Agent E will generate synthetic data as a last resort.

Record the download URL, expected file size (if known), download method
   (direct download, wget, Kaggle API, gdown for Google Drive, `curl` for GEO, etc.),
   and the data format (.h5ad, .csv, .mtx, etc.).

### 3. Check Dependencies

Look at the repository for:
- `requirements.txt` or `environment.yml` or `setup.py` or `pyproject.toml`
- Conda channel hints: `conda-forge`, `bioconda`, `defaults` — note which channels
  are needed (especially important for bioinformatics packages like `scanpy`,
  `anndata`, `celltypist` that are on conda-forge/bioconda)
- Any `Dockerfile` (fallback: Docker may be easier than manual venv)
- README installation section
- GitHub Issues with tag "installation" or "environment" — these often reveal
  real-world dependency problems that README doesn't mention
- Recent (last 6 months) closed Issues — check if anyone reported build failures
  or conda solve failures

### 4. Assess Repository Health

Quick health check — this informs retry strategy later:

| Factor | Check |
|---|---|
| Last commit | When was the repo last updated? |
| Stars | Community interest level |
| Open Issues | Are there many unresolved problems? |
| CI/CD | Is there a passing CI badge? GitHub Actions? |
| Docker | Is there a working Dockerfile? |

### 5. Output Format

Write to `paper_reproduction_output/paper_reading/resource_map.md`:

```markdown
# Resource Map

## Code Repository
- **Primary URL**:
- **Mirror URL** (if found):
- **Last commit date**:
- **README quality**: (good / partial / minimal — one sentence)
- **Installation instructions in README**: (yes / no — if yes, paste the
  relevant section)
- **Paper citation in README**: (yes — cites the target paper / no — cites a
  different paper / no citation found — plain "none")
  - If the README cites a paper, paste the citation line. This is used by the
    orchestrator to cross-validate that the repo and paper match.
- **Known issues from GitHub Issues**:
  - (list any installation/runtime issues found in Issues)

## Datasets
| Dataset | URL Status | Download URL | Size | Download Method | Format | Notes |
|---|---|---|---|---|---|---|
| ... | [VERIFIED] / [DEAD: reason] / [TRACED: file:line] / [UNREACHABLE] | ... | ... | ... | ... | ... |

## Dependency Files Found
- [ ] requirements.txt
- [ ] environment.yml (conda) — channels used: (e.g., conda-forge, bioconda, defaults)
- [ ] setup.py / pyproject.toml
- [ ] Dockerfile

## Upstream Dependencies
(Check if this tool depends on another tool for preprocessing — a common pattern
in bioinformatics where tool B expects data preprocessed by tool A.)

Look for:
- README instructions that say "first run X to generate Y, then use this tool"
- Data format requirements that imply preprocessing (e.g., "expects .loom files
  generated by velocyto", "requires BAM files processed by CellRanger")
- GitHub Issues where users ask "how do I generate the input files?"
- Example workflows that show a multi-step pipeline before this tool

For each upstream dependency found:
| Upstream Tool | Purpose | Install Method | Input → Output | Notes |
|---|---|---|---|---|
| velocyto CLI | Generate spliced/unspliced counts from BAM | `pip install velocyto` | BAM → .loom | Requires samtools; Python 2/3 compat issues |
| (none) | N/A | N/A | N/A | Tool is self-contained |

If the upstream dependency is optional (the tool provides built-in datasets that
bypass preprocessing), note this — it affects the reproduction strategy.

## Dependency File Contents
(If requirements.txt or environment.yml exists, paste its FULL content here.
For environment.yml, include the `channels:` section and the full `dependencies:` list.)

## Repository Health Assessment
| Factor | Status |
|---|---|
| Last commit | (date) |
| Stars | (count) |
| Open Issues | (count — note any installation-related ones) |
| CI passing | (yes / no / no CI) |
| Docker available | (yes / no) |

## Recommended Approach
(One paragraph: given what you found, what's the recommended way to set up the
environment? E.g., "Use the provided requirements.txt with Python 3.9" or
"Better to use the Dockerfile because of CUDA dependency complexity".)
```

### 6. Write TaskLog Entries

Append JSONL entries to `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`:

1. One entry for repo discovery (task_id: `find-repo`, step: "Search for paper's code repository", deviation: `none` if found, `blocker` if 404/not found)
2. One entry per dataset URL validated (task_id: `validate-data-url-<n>`, step: "Validate dataset URL with HEAD request")
3. If any URL is DEAD/SUSPECT and trace-back is attempted: one entry with `deviation.type: "error"`, root cause = why the URL failed
4. One summary entry for dependency file assessment (task_id: `assess-deps`, step: "Catalog dependency files in repo")

Minimum fields: `task_id`, `step`, `expected`, `actual`, `deviation`. See SKILL.md "TaskLog Protocol" for the full spec.

### 7. Final Assessment — Escalate If Blocked

Before finishing, summarize the data availability picture at the TOP of
`resource_map.md` (first section, before "Code Repository"):

```markdown
## Data Availability Summary

| Status | Count |
|--------|-------|
| Working URLs | N |
| Unreachable (HEAD failed, all mirrors exhausted) | M |
| Unverified (no URL found anywhere) | K |
```

**If `Working URLs = 0` (ALL datasets are unreachable or unverified):**
Write a prominent line at the top of resource_map.md:
`> ⛔ BLOCKED: No working data sources found. The orchestrator will stop the pipeline.`

This single line triggers Gate B in the orchestrator (SKILL.md Step 2.6).
Do NOT make this decision yourself — just surface the binary signal clearly.
