# CAIP: IEEE Access Reproducibility Repository

## Availability and release plan

**The associated manuscript is currently under review. This repository currently
contains this README only.** The simulation source code, synthetic experiment
data, figure-generation scripts, and complete reproduction documentation will be
publicly released in this repository upon acceptance of the associated manuscript.

The workflow below documents the prepared reproduction package. **It cannot yet
be executed from this public repository**, because the implementation, input data,
and dependency file are not included during peer review. The same repository URL
will be used for the full release, with a versioned release and an explicit
open-source license. No software license is granted by this placeholder README.

## Scope

The package evaluates CAIP using synthetic medical-workflow proxies in a shared
radio-access-network (RAN) and Edge simulation. It contains no real patient data,
hospital measurements, or clinically validated deadlines. The endpoints are:

- **S7 on-time rate:** technical deadline attainment measured from S2 group creation.
- **Full S0–S10 completion:** successful full-workflow completion within a 500 ms
  simulation timeout.

The fixed experiment uses two scenarios (`moderate_steady` and `moderate_batch`),
40 paired seeds (`94001`–`94040`), and 200 workflows per seed and scenario.
Each policy receives the same seed-specific exogenous trace. The complete cohort
contains 8,000 initiated workflows per policy per scenario.

The five policies retained in the reproduction data are `EDF`, `EDF+Joint`,
`FG-EDF`, `CAIP`, and `CAIP-no-gate`. The main figures display EDF, EDF+Joint and
CAIP; the gate ablation compares CAIP with CAIP-no-gate. FG-EDF remains part of the
complete comparator data. S7 deadline performance and full-workflow completion
must be interpreted separately, and comparisons with FG-EDF are necessary to
assess whether there is an incremental joint-control contribution.

## Materials planned for release

```text
CAIP-IEEE-Access/
  README.md
  LICENSE
  requirements.txt
  REPRODUCE_EXPERIMENTS_AND_FIGURES.md
  study/
    final_protocol.py
    run_publication.py
    run_v1.py
    run_v2.py
    caip_sim/
    tests/
  scripts/
    derive_type_metrics.py
    verify_release.py
    plot_figure_panels.py
    verify_publication_figures.py
  results/
    formal_94001_94040/
      per_seed/
      per_seed_metrics.csv
      method_summary.csv
      paired_contrasts.csv
      workflow_outcome_flips.csv
      type_stratified_metrics.csv
      run_manifest.json
      completion_status.json
  figures/
  provenance/
  docs/
```

The intended full release includes the raw per-workflow synthetic traces, frozen
summary tables, source-integrity records, seven independent figure-panel PDFs,
and three assembled figure PDFs. The exact release version and license will be
identified when the complete materials are published.

## Reproduction workflow after the full release

The commands in this section apply **only after the complete release**. Run them
from the repository root unless otherwise indicated.

### 1. Download and set up the environment

```bash
git clone https://github.com/lwkang/CAIP-IEEE-Access.git
cd CAIP-IEEE-Access
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

On Windows PowerShell, create and activate the environment using:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

The prepared package uses Python 3.13.5 and pinned dependencies including NumPy
2.3.5, Pandas 2.2.3, SciPy 1.17.0, Matplotlib 3.10.8, Pytest 9.0.2, Pillow 12.3.0,
ReportLab 4.4.9 and PyMuPDF 2.1. The dependency file will accompany the full release.
Font availability and rendering-library versions can affect PDF appearance and
byte-level checksums without changing numerical results.

### 2. Verify the frozen experiment and source

```bash
python scripts/verify_release.py --results results/formal_94001_94040
```

The prepared verification checks frozen simulator source hashes, 80 completed
scenario–seed pairs, 400 method-by-seed metric rows, 80,000 workflow–method
records, aggregate counts, class-specific outcomes and paired rescue/harm totals.

### 3. Run simulation tests

```bash
cd study
python -m pytest -q tests
cd ..
```

The prepared package's recorded reference outcome is 59 passing tests. This is a
local verification record; the public placeholder repository does not yet provide
an executable test suite or continuous-integration result.

### 4. Rebuild Figures 4–6 without rerunning simulations

The full release is planned to include the complete frozen 40-seed results.
Those results allow the figures to be regenerated directly:

```bash
python scripts/derive_type_metrics.py --results results/formal_94001_94040
python scripts/plot_figure_panels.py --data results/formal_94001_94040 --output figures
python scripts/verify_publication_figures.py
```

`plot_figure_panels.py` is the single publication-figure entry point. It writes
seven independent panel PDFs, three assembled PDFs and an input-data manifest.
It does not generate alternate styles, PNG previews or ZIP archives.

| Figure | Independent panels | Assembly and content |
|---|---|---|
| 4 | `figure4a.pdf`, `figure4b.pdf` | Single-column 2×1: paired S7 deadline effects and full-workflow noncompletion |
| 5 | `figure5a.pdf`, `figure5b.pdf`, `figure5c.pdf` | Double-column 1×3: paired outcome transitions, per-seed Edge waiting P95 and gate ablation |
| 6 | `figure6a.pdf`, `figure6b.pdf` | Single-column 2×1: class-specific S7 rates and normalized cumulative S7 completion |

The assembled files are `figure4_deadline_reliability.pdf`,
`figure5_mechanism.pdf` and `figure6_workflow_types_latency.pdf`. Figure 6(b)
requires all 80 raw per-workflow `.csv.gz` files in addition to summary tables.
PDF verification checks intended dimensions, embedded fonts and text bounds.
Subfigure letters are omitted so they can be supplied by LaTeX.

### 5. Optional: run a small paired smoke test

```bash
cd study
python run_publication.py --output ../results/check_one_seed --seeds 94001 --scenarios moderate_batch --workers 1
cd ..
python scripts/derive_type_metrics.py --results results/check_one_seed
python scripts/verify_release.py --results results/check_one_seed --allow-partial --reference results/formal_94001_94040
```

This executes five methods for one scenario–seed pair, producing 1,000
workflow–method records. It checks the execution path and reference agreement;
it is not a new validation cohort. Publication plotting requires the full paired
cohort and intentionally rejects smoke-only data.

### 6. Optional: reproduce the full fixed experiment

The following cross-platform command runs the original 40 seeds for both
scenarios. Runtime depends on available compute resources.

```bash
cd study
python -c "import subprocess,sys; subprocess.run([sys.executable,'run_publication.py','--output','../results/reproduced_94001_94040','--seeds',*[str(s) for s in range(94001,94041)],'--workers','2'],check=True)"
cd ..
python scripts/derive_type_metrics.py --results results/reproduced_94001_94040
python scripts/verify_release.py --results results/reproduced_94001_94040 --reference results/formal_94001_94040
python scripts/plot_figure_panels.py --data results/reproduced_94001_94040 --output figures
python scripts/verify_publication_figures.py
```

Completed scenario–seed pairs are recorded with a workflow-trace file and a
completion marker. Repeating a partially completed run resumes completed-pair
tracking and rebuilds aggregates. Keep the frozen implementation unchanged and
do not mix results generated by different algorithm revisions. Reusing the
original seeds reproduces the fixed experiment; it is not independent validation
of a later algorithm change.

## Citation and updates

Publication metadata, the recommended citation, the full-release tag and license
will be added after acceptance. Until then, this repository serves as the public
reproduction-workflow description and release notice for the manuscript under
review.
