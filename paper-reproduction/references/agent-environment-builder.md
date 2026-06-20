# Agent C: EnvironmentBuilder

## Goal

Create an isolated, reproducible environment for the paper's code. For Python
repositories: create venv/conda env, install dependencies, verify all imports.
For non-Python repositories (R/MATLAB/Julia): follow the **minimal path** —
install packages, verify they load, document what works and what doesn't.

## Input

You will receive:
- `extracted_methodology.md` — paper's dependency list
- `resource_map.md` — requirements.txt contents, repo URL, dataset info, and
  the "Upstream Dependencies" table if any were detected
- `reproduction_mode.txt` — `experiment` or `tool` (so you know install strategy)

## Task

### 0. Handle Upstream Dependencies First

**Check `resource_map.md` → "Upstream Dependencies" section BEFORE installing anything.**

If upstream tools are listed:
1. Install each upstream tool before the main tool. The upstream tool's output
   is the main tool's input — installing them in the wrong order wastes time.
2. For each upstream tool, verify it works before moving on:
   ```bash
   # Example: check velocyto is callable
   velocyto --help 2>&1 | head -5
   ```
3. If an upstream tool has complex system dependencies (e.g., `samtools`,
   `CellRanger`), note them but do not attempt to install from source — flag
   and proceed. These are often pre-installed on HPC/bioinformatics systems.
4. If the upstream dependency is **optional** (the tool provides built-in datasets
   that bypass preprocessing), **skip the upstream tool** and use the built-in
   data path. Document this choice in `setup_report.md` → "Upstream Dependencies"
   section with rationale.

Common upstream tool patterns and their handling:

| Pattern | Example | Strategy |
|---|---|---|
| CLI tool that preprocesses raw data | `velocyto` → `.loom` → `scvelo` | Install if pip-available; skip if built-in data exists |
| System-level bioinformatics tool | `samtools`, `CellRanger`, `Strelka2` | Check if on PATH; flag if missing; do NOT install from source |
| R package called via subprocess | `Seurat` preprocessing → Python tool | Outside v1 scope; flag and suggest rpy2 bridge |
| Docker container that bundles the chain | `quay.io/teichlab/celltypist:latest` | Use the Docker path — it bundles upstream + main tool |

After handling upstream dependencies (or confirming none are needed), proceed to
the standard environment setup below.

### 1. Pre-flight Checks

Before installing anything:

```bash
# Verify Python and pip are from the same installation
python --version
python -m pip --version

# If python → 3.10 but python -m pip → 3.8, STOP and report mismatch.
# Fix: use python3.10 -m pip, or specify full path to the desired python.
```

On Windows, additionally:
```python
# Test that stdout encoding won't corrupt output
python -c "import sys; print(sys.stdout.encoding)"
# If output is 'gbk' or not 'utf-8', note this — generated scripts
# will need sys.stdout.reconfigure(encoding='utf-8')
```

### 1.5 — numpy ABI Pre-emptive Pin

**The most common silent reproduction killer in 2024-2026 is numpy ABI mismatch.**
Many scientific Python packages (scvelo, scanpy, numba, scikit-learn pre-1.6,
etc.) were built against numpy 1.x ABI and will `pip install` successfully on
numpy 2.x — then crash at `import` time with opaque errors like:

```
AttributeError: _ARRAY_API not found
AttributeError: module 'numpy.core.multiarray' has no attribute '...'
ImportError: numpy.core.multiarray failed to import
RuntimeError: module compiled against ABI version 0x0... but this numpy is 0x0...
```

These errors waste hours because `pip install` reports success. Pre-empt the
problem BEFORE any packages are installed:

**Rule (applies to every new environment):**

```bash
# Install numpy <2.0 FIRST, before any other package.
# This forces the resolver to build against the numpy 1.x ABI.
python -m pip install 'numpy<2.0'
```

**Why first:** pip's dependency resolver chooses numpy 2.x by default on
Python ≥3.9. Installing numpy<2.0 first constrains the solve so all subsequently
installed packages with `numpy` in their build dependencies link against the 1.x
ABI. Doing it after other packages are already installed guarantees partial ABI
mismatch.

**Pin version in requirements.txt:** If the repo has a `requirements.txt` without
a numpy pin, add `numpy<2.0` as the first line before installing:

```bash
# Read the original requirements, prepend the pin, write a patched copy
echo "numpy<2.0" > paper_reproduction_output/env_setup/requirements_patched.txt
cat requirements.txt >> paper_reproduction_output/env_setup/requirements_patched.txt
python -m pip install -r paper_reproduction_output/env_setup/requirements_patched.txt
```

**If conda is used instead of venv+pip:** Add `numpy<2.0` as the first dependency
in the environment specification before creating the environment.

### 2. Create Isolated Environment

```bash
# Create venv in the reproduction output directory
python -m venv paper_reproduction_output/env_setup/venv

# Activate (platform-specific)
# Windows: paper_reproduction_output\env_setup\venv\Scripts\activate
# Linux/Mac: source paper_reproduction_output/env_setup/venv/bin/activate
```

If the paper provides a conda `environment.yml`, prefer conda:

**Step 2a — Test conda connectivity and set mirrors:**

```bash
# Test default conda channels
conda search conda --json --dry-run 2>&1
# If this times out or shows connection errors, default channels are unreachable.
```

If conda channels are unreachable, switch to Tsinghua conda mirrors:

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
conda config --set show_channel_urls yes
```

USTC mirror as backup:
```bash
# conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
# conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
# conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
```

**Step 2b — Rewrite environment.yml channels (if mirror needed):**

If using a mirror, replace channel URLs in `environment.yml` before creating the env:

```bash
# Backup original
cp environment.yml environment.yml.bak
# Replace conda-forge channel
sed -i 's|- conda-forge|- https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge|g' environment.yml
sed -i 's|- bioconda|- https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda|g' environment.yml
```

Then create the environment:
```bash
conda env create -f environment.yml -p paper_reproduction_output/env_setup/conda_env
```

If `environment.yml` doesn't exist but the resource map lists conda dependencies,
write one yourself at `paper_reproduction_output/env_setup/environment.yml` from
the dependency info in `resource_map.md`.

**Step 2c — Handle conda solve failures:**

If `conda env create` takes >10 minutes or fails with a solve error:
1. Try `conda install -c conda-forge mamba` first, then use `mamba env create`
   (mamba solves ~10× faster)
2. Try reducing the environment to only the packages listed in the paper's
   requirements (skip optional/dev dependencies)
3. As a last resort, create an empty conda env and `pip install` packages
   inside it one by one

### 3. Install Dependencies

**Step 3a — Test default PyPI connectivity:**

```bash
python -m pip install --dry-run pip --quiet 2>&1
# If this times out or fails, PyPI is unreachable from this network.
```

**Step 3b — Set mirror if needed:**

If default PyPI is unreachable, switch to Tsinghua mirror:
```bash
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
```

Also test USTC mirror as backup:
```bash
# python -m pip install -i https://pypi.mirrors.ustc.edu.cn/simple --upgrade pip
```

**Step 3c — Install from requirements.txt (pip):**

If the paper has a `requirements.txt`:
```bash
python -m pip install -r requirements.txt
# If mirror is active, include -i flag
# python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt
```

Or from the extracted content in `resource_map.md` if no file exists yet.

**Step 3c-conda — Install via conda (conda path):**

If using conda (environment created in Step 2), activate and install:

```bash
conda activate paper_reproduction_output/env_setup/conda_env
# Install the paper's key dependencies via conda (faster solve than pip for
# packages with compiled extensions like scipy, numba, pytorch)
conda install -c conda-forge -c bioconda <package_list>
```

If mirrors are active, use the mirror-prefixed channels:
```bash
conda install -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge \
    -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda \
    <package_list>
```

Common bioinformatics conda packages (install with `-c conda-forge -c bioconda`):
- `scanpy`, `anndata`, `celltypist`, `scvi-tools`, `scirpy`, `muon`
- `scikit-learn`, `numba`, `pytorch`, `leidenalg`, `harmonypy`

If `conda install` fails with a solve error, try `pip install` for the
same package inside the conda environment — conda envs allow pip fallback:
```bash
python -m pip install <package>
```

If `requirements.txt` doesn't exist in the repo yet (you only have its content
from `resource_map.md`), write it to disk first at
`paper_reproduction_output/env_setup/requirements.txt`, then install from it.

**Step 3d — Install the paper's package (if applicable):**

Many repos use `pip install -e .` or `python setup.py install`. Check the
repo's README or setup.py.

**Step 3e — Handle CUDA / torch special cases:**

If the paper uses PyTorch:
```bash
# Check if CUDA is available
python -c "import torch; print(torch.cuda.is_available())"
```
If CUDA is not available but the paper requires it:
- Note this prominently in the report
- Try CPU fallback if the code supports it
- The reproduction may run very slowly or not at all

### 4. Verify All Imports (with ABI Diagnosis)

**Step 4a — Quick numpy ABI probe first:**

Before running the full import verification, do a 2-second ABI sanity check:

```bash
python -c "import numpy; print(f'numpy {numpy.__version__}, ABI compatible')" 2>&1
```

If this fails with ANY of these patterns:
- `_ARRAY_API not found`
- `numpy.core.multiarray failed to import`
- `module compiled against ABI version`
- `AttributeError` mentioning `numpy.core`

→ **numpy ABI conflict confirmed.** The current numpy version is incompatible with
one or more installed packages. Do NOT proceed to full import verification —
fix the ABI first (see "ABI Recovery" below).

**ABI Recovery procedure:**

1. Check current numpy and Python versions:
   ```bash
   python -c "import sys; print(f'Python {sys.version}')"
   pip show numpy 2>/dev/null | grep Version
   ```

2. If numpy ≥2.0 is installed:
   ```bash
   python -m pip install 'numpy<2.0' --force-reinstall
   ```
   Then re-run the ABI probe. If it still fails:

3. If Python ≥3.13 (which dropped numpy 1.x wheel support):
   - numpy 1.x has no pre-built wheels for Python 3.13+
   - **Create a new environment with Python ≤3.12** and start over
   - This is a last resort — note it prominently in `setup_report.md`

4. If the ABI error persists even with numpy<2.0 and Python ≤3.12:
   - The conflict is in a specific package that was built against numpy 2.x ABI
   - Identify it: `python -c "import <package_that_crashed>; print(<package>.__version__)"`
   - Pin that package to an older version compatible with numpy 1.x
   - Record the package, its version constraint, and the rationale

**ABI recovery is successful when:**
```bash
python -c "import numpy; print(f'numpy {numpy.__version__}, ABI compatible')"
# → numpy 1.26.x, ABI compatible
```

Then proceed to full import verification below.

**Step 4b — Full import verification:**

Write and run a verification script `verify_imports.py`:

```python
import sys
import os

# Ensure UTF-8 output on Windows
if sys.platform == 'win32':
    sys.stdout.reconfigure(encoding='utf-8')

# List ALL packages that should be importable
# (Derive this list from requirements.txt and the paper's methodology)
packages_to_check = [
    # ... populated from the paper's dependency list
]

failed = []
for pkg in packages_to_check:
    try:
        __import__(pkg)
        print(f"  ✓ {pkg}")
    except ImportError as e:
        print(f"  ✗ {pkg}: {e}")
        failed.append(pkg)

if failed:
    print(f"\nFAILED: {len(failed)} package(s) could not be imported:")
    for pkg in failed:
        print(f"  - {pkg}")
    sys.exit(1)
else:
    print(f"\nAll {len(packages_to_check)} packages imported successfully.")
```

### 4c — Structured Dependency Conflict Resolution Loop

When the import verification fails (Step 4b exit code ≠ 0), do NOT restart the
entire agent or recreate the environment from scratch. Instead, run this
**targeted diagnosis → fix → re-verify loop**.

**Core principle:** Only touch the failing package. Never reinstall everything.

**Loop limit:** Maximum 5 rounds total. After 5 rounds, accept partial success
and document the unresolved failures honestly.

#### Round structure (repeat for each round)

```
┌─────────────────────────────────────────────────┐
│  ROUND N:                                       │
│  1. Parse the error (category below)            │
│  2. Diagnose root cause (which package, why)    │
│  3. Apply targeted fix (only that package)      │
│  4. Re-verify: re-run verify_imports.py         │
│  5. If all pass → exit loop (SUCCESS)           │
│     If fewer failures → next round              │
│     If same failures → escalate strategy        │
│     If new failures → diagnose the new ones     │
└─────────────────────────────────────────────────┘
```

#### Step 1 — Parse error category

Classify every import failure into exactly one of these categories:

| Category | Error Signature | Root Cause |
|---|---|---|
| **ABI** | `_ARRAY_API not found`, `numpy.core.multiarray failed to import`, `module compiled against ABI version`, `AttributeError` on `numpy.core` | numpy 1.x/2.x ABI mismatch between compiled extension and runtime numpy |
| **Version conflict** | `ImportError: cannot import name 'X' from 'Y'`, `AttributeError: module 'X' has no attribute 'Y'`, `TypeError: X() got an unexpected keyword argument 'Y'` | A dependency (usually a transitive one) is too new or too old for the package |
| **Missing transitive dep** | `ModuleNotFoundError: No module named 'X'` (where X is NOT in requirements.txt) | A package's dependency wasn't declared in its setup.py and wasn't pulled in by pip. **Also catches runtime failures:** if a function call fails with ModuleNotFoundError after imports passed (e.g., `scvi.data.retina()` → `ModuleNotFoundError: pooch`), the missing package is the module name in the error message. |
| **Build failure** | `error: command 'gcc' failed`, `error: Microsoft Visual C++ ... required`, `fatal error: Python.h: No such file or directory` | Missing system compiler or header files — the package has C extensions that failed to compile |
| **System library** | `ImportError: libXXX.so: cannot open shared object file`, `DLL load failed` | Missing system-level shared library that the Python package wraps |
| **Other** | Anything else | Unclassified — escalate |

#### Step 2 — Diagnose root cause

For each category, run these diagnostic commands:

**ABI:**
```bash
python -c "import numpy; print(numpy.__version__)"
pip show <failing_package> | grep -i "requires\|version"
# Check: was numpy<2.0 pinned BEFORE this package was installed?
# If numpy≥2.0 is present → ABI conflict is the cause
```

**Version conflict:**
```bash
pip show <failing_package> | grep -i "requires\|version"
pip list | grep -i "<suspected_dependency>"
# Check the failing package's GitHub Releases / PyPI history:
# which version was current when the paper was published?
# Pin to that version: pip install '<package>==<paper_era_version>'
```

**Missing transitive dep:**
```bash
# The missing module name tells you what to install
python -m pip install <missing_module_name>
# If the module name differs from the pip package name:
# pip search or google "pip install <module_name>" to find the right package
```

**Build failure:**
```bash
# Check for pre-built wheels first
pip install <package> --only-binary :all: 2>&1
# If that fails, try conda (conda distributes pre-compiled binaries):
conda install -c conda-forge <package>
# If on Windows and no conda:
# Check https://www.lfd.uci.edu/~gohlke/pythonlibs/ for pre-built wheels
```

**System library:**
```bash
# Identify which library is missing and suggest install command
# Don't try to install system libs — flag for the user
```

#### Step 3 — Apply targeted fix

Fix ONLY the failing package. Do not touch packages that passed import.

| Category | Fix action (try in order) |
|---|---|
| **ABI** | 1. `pip install 'numpy<2.0' --force-reinstall` 2. If Python ≥3.13: recreate env with Python ≤3.12 |
| **Version conflict** | 1. Pin the package to the version current when the paper was published: `pip install '<pkg>==<paper_era_version>'` 2. If that doesn't work, try the immediately previous major version |
| **Missing transitive dep** | `pip install <missing_package>` (one package, not all). **Runtime recovery:** If the error came from a function call (not an import statement), install the missing package and re-run the SAME function call — do NOT restart the entire smoke test. Example: `python -c "import scvi; scvi.data.retina()"` fails with `ModuleNotFoundError: pooch` → `pip install pooch` → retry the same one-liner. |
| **Build failure** | 1. Try conda: `conda install -c conda-forge <package>` 2. Try an older version with pre-built wheels 3. Flag as system dependency if both fail |
| **System library** | Flag for user. Do NOT attempt to install system-level .so/.dll files. |

#### Step 4 — Re-verify

Re-run `verify_imports.py` (the same script from Step 4b). Only the previously
failed packages plus any newly discovered failures need checking — but running
the full script is safer.

#### Step 5 — Decide next action

- **All imports pass** → exit loop. Mark `setup_report.md` status as ✓ SUCCESS.
- **Fewer failures than previous round** → progress! Continue to next round.
- **Same failures, same error** → escalation: try the next strategy in the category's fix list.
  If all strategies exhausted for this category, mark the package as UNRESOLVED.
- **New failures introduced** (a fix broke something else) → the fix was wrong.
  Revert: `pip install <pkg>` (remove the pin). Try a different version.
- **5 rounds completed** → STOP. Accept partial success. Document unresolved
  packages honestly in `setup_report.md`.

#### Record each round

After each round, append to a running log in `setup_report.md`:

```markdown
### Diagnosis Round 1
| Package | Error Category | Root Cause | Fix Applied | Result |
|---|---|---|---|---|
| scvelo | ABI | numpy 2.2.0 ABI vs scvelo 0.3.4 built for numpy 1.x | `pip install 'numpy<2.0' --force-reinstall` | ✓ Passed |
| llvmlite | Build failure | No MSVC compiler for Python 3.13 | `conda install -c conda-forge llvmlite` | ✓ Passed |

### Diagnosis Round 2
(if needed)
...
```

### 4.5 — API Surface Inspection (Pre-Smoke-Test)

**After all imports pass (Step 4b/4c done, all ✓), before handing off to Agent D,**
inspect the API surface of the MAIN package(s) you just installed. The orchestrator
will pass these API calls to Agent D and Agent E — if you hardcode wrong parameter
names, they will fail silently or crash, wasting the smoke test's time.

**Trigger:** Every `pip install`-ed package that has a non-trivial API (more than
one function parameter, or functions whose signatures changed across versions).

**Procedure:**

```python
import inspect
import <target_package>

# Step 1: List the public API surface
public_names = [n for n in dir(<target_package>) if not n.startswith('_')]
print(f"Public API: {public_names}")

# Step 2: For each function that Agent D/E will call, print its signature
key_functions = ['<function_1>', '<function_2>', '<function_3>']
for fn_name in key_functions:
    fn = getattr(<target_package>, fn_name, None)
    if fn is not None and callable(fn):
        sig = inspect.signature(fn)
        print(f"  {fn_name}{sig}")
        # Also note parameter types if available
        for pname, param in sig.parameters.items():
            if param.default is not param.empty:
                print(f"    {pname} = {param.default!r}  (default)")
            else:
                print(f"    {pname}  (REQUIRED)")
    else:
        print(f"  {fn_name}: NOT FOUND — may be in a submodule")
```

**Minimum output to write to `setup_report.md`:**

```markdown
## API Surface (verified signatures)

| Function | Signature | Key Parameters |
|----------|-----------|----------------|
| models.<function_1> | (<param_1>, <param_2>, ...) | Key parameters and their types |
| models.<function_2> | (<param_1>, <param_2>, ...) | Return type and shape notes |
| models.<function_3> | (<param_1>, <param_2>, ...) | Any special type requirements |
```

**Why this matters:** The 2026-06-16 Annotatability smoke test took 6 iterations to
find the correct API calls because parameter names were guessed (e.g., `annotations=`
instead of `label_key=`, unpacking a single return as triple tuple). Each wrong guess
wastes 2-5 minutes. Two minutes of `inspect.signature()` prevents all of that.

**Checkpoint before handing off to Agent D:** Verify that every function you
documented:
- [ ] Actually exists at the path you listed
- [ ] Has the parameter names you documented (WYSIWYG from inspect)
- [ ] Returns what you claim it returns (type and shape)

If `inspect` fails (e.g., C extension, @jit decorator), try:
```python
# Fallback: read the function's docstring
print(fn.__doc__[:500])
# Fallback: read the source
import inspect as _inspect
print(_inspect.getsource(fn)[:500])
```
If all three fail, mark the function as "API UNVERIFIED" and pass a warning to Agent D.

### 5. Output Format

Write `paper_reproduction_output/env_setup/setup_report.md`:

```markdown
# Environment Setup Report

## Upstream Dependencies
(Only include if `resource_map.md` listed upstream dependencies.)

| Upstream Tool | Installed | Version | Status |
|---|---|---|---|
| ... | yes/no | ... | ✓ working / ✗ not found / ⚠ skipped (built-in data used) |

- Rationale for skipped upstream dependencies:
  (If any upstream tool was skipped, explain why — e.g., "built-in scVelo
  datasets at scvelo.org bypass velocyto preprocessing")

## Platform
- OS: (Windows/Linux/macOS, version)
- Python version:
- pip version:
- CUDA available: (yes / no / version)
- GPU: (model if available)
- numpy ABI pin applied: (yes — `numpy<2.0` installed first / no — not needed)
- numpy ABI probe result: (✓ compatible / ✗ conflict — see ABI Recovery section)

## Environment
- Type: (venv / conda)
- Path: (absolute path)
- Activation command: (exact command for this platform)

## Mirror Status
- Default PyPI: (reachable / unreachable)
- Pip mirror used: (none / Tsinghua / USTC / other)
- Default conda channels: (reachable / unreachable / not applicable)
- Conda mirror used: (none / Tsinghua / USTC / other)
- `environment.yml` channels rewritten: (yes / no / not applicable)

## Installed Packages
| Package | Version | Source |
|---|---|---|
| ... | ... | ... |

## Import Verification
- Total packages checked:
- Passed:
- Failed:

## Failed Packages (if any)
| Package | Error | Attempted Fix |
|---|---|---|
| ... | ... | ... |

## Setup Status
(✓ SUCCESS / ✗ FAILED)

## Notes for Next Steps
- Activation command to use:
- Python path to use:
- Any warnings or caveats:
```

---

## Minimal Path — Non-Python Environments (R / MATLAB / Julia)

**Only follow this section if `language_scope: minimal` is set by the orchestrator.**
This is a reduced-scope path — you are NOT expected to achieve full numerical
reproduction. Your job is: install packages, verify they load, document honestly.

### R Environment

**1. Detect R installation:**
```bash
R --version
Rscript --version
# Note the R version and R_HOME
```

**2. Test network connectivity (CRAN mirror):**
CRAN may be unreachable from some networks. Test and switch mirrors:
```r
# Test default CRAN
options(repos = c(CRAN = "https://cloud.r-project.org"))
# If unreachable, switch to Tsinghua mirror:
options(repos = c(CRAN = "https://mirrors.tuna.tsinghua.edu.cn/CRAN/"))
# USTC backup:
# options(repos = c(CRAN = "https://mirrors.ustc.edu.cn/CRAN/"))
```

**3. Install R packages:**
Read the repo's README and any `DESCRIPTION` file for the package list.
```r
# Install each package
install.packages(c("openxlsx", "HGNChelper", "scales", "Seurat"))
# If packages are from GitHub:
# devtools::install_github("user/repo")
# If packages are from Bioconductor:
# BiocManager::install(c("SingleCellExperiment", "scran"))
```

**Important: Run R via PowerShell on Windows.** R under Git Bash has a known
libcurl SSL conflict that causes segfault on any network call. Always use:
```powershell
Rscript.exe path/to/script.R
```

**4. Verify package loading:**
Write `verify_packages.R`:
```r
pkgs <- c("openxlsx", "HGNChelper", "scales", "Seurat")
for (pkg in pkgs) {
  ok <- require(pkg, character.only = TRUE)
  cat(sprintf("%s %s\n", if(ok) "✓" else "✗", pkg))
}
```

**5. Non-standard repo structures:**
Some R repos (e.g., ScType) are NOT R packages — they are source directories
with `.R` files meant to be loaded via `source()`. Check the README: if it says
`source("R/sctype_score_.R")` instead of `devtools::install_github()`, use the
source-based loading path.

### MATLAB Environment

**1. Detect MATLAB:**
```bash
matlab -help 2>&1 | head -5
# If not on PATH, check common install locations:
# Windows: C:\Program Files\MATLAB\R20xx\bin\
```

**2. Check required toolboxes:**
```matlab
ver  % lists all installed toolboxes
% Compare against the paper's requirements (Statistics, Bioinformatics, etc.)
```

**3. Install toolboxes if missing:**
MATLAB add-ons must be installed interactively or via `matlab.addons.toolbox.installToolbox`.
If required toolboxes are missing, note them — do NOT attempt automated install.

### Julia Environment

**1. Detect Julia:**
```bash
julia --version
```

**2. Install packages:**
```julia
using Pkg
Pkg.add(["PackageA", "PackageB"])
```

**3. Verify:**
```julia
using PackageA, PackageB
println("All packages loaded.")
```

### Minimal Path Output

Write `paper_reproduction_output/env_setup/setup_report.md` with the same
format as the Python path, but add a header:

```markdown
## Scope
- **Language**: R / MATLAB / Julia
- **Path**: minimal (install + verify only)
- **Full numerical reproduction**: NOT guaranteed
```

Report honestly — if a package won't install, say so. If the environment
is incomplete, say so. The minimal path's value is in the honest feedback.

### 6. Write TaskLog Entries

Append JSONL entries to `.claude/tasklog/YYYYMMDD-<session-slug>.jsonl`:

1. One entry for environment creation (task_id: `create-env`, step: "Create isolated Python environment", deviation: `none` if created, `error` if failed)
2. One entry per dependency installation batch (task_id: `install-deps`, step: "Install dependencies from requirements.txt/conda")
3. One entry for the numpy ABI probe result (task_id: `numpy-abi-probe`, step: "Check numpy ABI compatibility")
4. If any import verification fails: one entry per failing package with `deviation.type: "error"`, root cause diagnosed per §4c categories
5. One entry per diagnosis round in the conflict resolution loop (§4c), recording which package failed, the error category, the fix applied, and whether it worked
6. One summary entry when all imports pass or 5 rounds exhausted (task_id: `import-verify-final`)

Minimum fields: `task_id`, `step`, `expected`, `actual`, `deviation`. For deviation entries, include `root_cause`, `fix`, `fix_worked`. See SKILL.md "TaskLog Protocol" for the full spec.
