---
name: rare-cell-detection-benchmarking
description: Use when comparing rare-cell-type identification methods on scRNA-seq
  data, or when quantifying how few differentially expressed genes a detector needs
  and how runtimes scale to tens of thousands of profiles.
when_to_use_negative:
- The goal is doublet detection rather than rare-cell detection — FiRE does not discriminate
  between an outlier and cells representing minor cell types, and doublet rates (1%–30%)
  do not necessarily comply with the notion of cell rareness
- The analysis requires binary-only outputs to be avoided, i.e. when graded rareness
  scores per cell are needed for downstream ranking rather than the binary calls RaceID/GiniClust
  provide
- The dataset is a feature/summary table (e.g. already-clustered or dimensionality-reduced)
  rather than a cell x gene expression profile matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: 'Primary rare-entity scorer under benchmark: assigns a rareness score to every
    expression profile via LSH-based density estimation (L=100, M=50, H=1017881)'
  repo: https://github.com/princethewinner/FiRE
- name: RaceID
  role: 'Baseline comparator: R-based rare-cell identifier whose runtime is benchmarked
    (took ~79 h for just 5k cells)'
- name: GiniClust
  role: 'Baseline comparator: R-based rare-cell identifier benchmarked for runtime
    and failure behaviour (hit a runtime error beyond ~45k profiles)'
- name: Python
  role: Environment for installing the FiRE module ('python setup.py install'), scoring
    cells, computing AUC-ROC, and timing runs
- name: splatter
  role: R package used to simulate two-group single-cell expression data with controlled
    numbers of DE genes for the sensitivity curve
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:4e37aca1479187fc413f405cec4ecae332694458cf5c90e63abf477dcfe6c24d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princethewinner/FiRE
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - Python
  - RaceID
  - Seurat
  - splatter
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rare-cell-detection-benchmarking

## Summary

Benchmark rare-cell detection methods (FiRE, LOF, RaceID, GiniClust) on simulated and real scRNA-seq data by measuring AUC-ROC sensitivity to the number of DE genes and single-core runtime scaling from 1k to ~68k cells. This reproduces the FiRE paper's evaluation strategy and provides decision support for choosing a rare-entity finder.

## When to use

Use when comparing rare-cell-type identification methods on scRNA-seq data, or when quantifying how few differentially expressed genes a detector needs and how runtimes scale to tens of thousands of profiles.

## When NOT to use

- The goal is doublet detection rather than rare-cell detection — FiRE does not discriminate between an outlier and cells representing minor cell types, and doublet rates (1%–30%) do not necessarily comply with the notion of cell rareness
- The analysis requires binary-only outputs to be avoided, i.e. when graded rareness scores per cell are needed for downstream ranking rather than the binary calls RaceID/GiniClust provide
- The dataset is a feature/summary table (e.g. already-clustered or dimensionality-reduced) rather than a cell x gene expression profile matrix

## Inputs

- splatter-simulated two-group scRNA-seq count matrix (batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05), de.facLoc=0.4, de.facScale=0.8)
- Human PBMC scRNA-seq expression matrix from GEO accession GSE94820 (~68k cells)
- Binary rare/abundant cell labels for AUC-ROC computation
- Installed FiRE Python module plus LOF (Python), RaceID (R) and GiniClust (R) implementations

## Outputs

- auc_roc_curve.csv: mean AUC-ROC versus number of DE genes (averaged over 1000 repetitions per count)
- Comparison table of single-core wall-clock runtimes, one row per method per subset size (1k to ~68k cells), including failure records

## How to apply

Build the FiRE Python module from the repository with './INSTALL --py' (which runs 'python setup.py install --record fireInstall.log'). For sensitivity analysis, simulate two-group expression data with the splatter R package (batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05)) so the rare population is 5% of cells; for each DE-gene count from 1 to 150, replace that many non-DE genes with pre-identified DE genes using de.facLoc=0.4 and de.facScale=0.8, score every cell with FiRE (e.g. model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489)) over 1000 repetitions, compute AUC-ROC with the rare group as positives, and average per count to obtain the mean AUC-ROC vs DE-gene-count curve. For runtime scaling, download the ~68k-cell human PBMC dataset (GEO GSE94820), construct nested subsets (1k, 5k, 10k, 15k, ... up to ~68k), run FiRE, LOF, RaceID and GiniClust single-core on each subset, and log wall-clock times and failures. Preprocess expression matrices with ranger_preprocess (keeping the top 1000 variable genes, log2 transformed) before scoring, and use IQR-based thresholding (q3 + 1.5*iqr of FiRE scores) for binary rare-cell calls.

## Related tools

- **FiRE** (Primary rare-entity scorer under benchmark: assigns a rareness score to every expression profile via LSH-based density estimation (L=100, M=50, H=1017881)) — https://github.com/princethewinner/FiRE
- **RaceID** (Baseline comparator: R-based rare-cell identifier whose runtime is benchmarked (took ~79 h for just 5k cells))
- **GiniClust** (Baseline comparator: R-based rare-cell identifier benchmarked for runtime and failure behaviour (hit a runtime error beyond ~45k profiles))
- **Python** (Environment for installing the FiRE module ('python setup.py install'), scoring cells, computing AUC-ROC, and timing runs)
- **splatter** (R package used to simulate two-group single-cell expression data with controlled numbers of DE genes for the sensitivity curve)

## Examples

```
python -c "import FiRE; m=FiRE.FiRE(L=100,M=50,H=1017881,seed=5489); m.fit(X); s=m.score(X)"  # after ./INSTALL --py
```

## Evaluation signals

- Mean AUC-ROC at >=20 DE genes should show a sharp improvement over the low-DE-gene regime, matching Fig. 4 of the paper
- FiRE should complete the full ~68k-cell PBMC data in ~26 s single-core (paper reports ~31 s class runtimes); material deviation should be flagged
- GiniClust should fail/terminate beyond ~45k cells and RaceID should require on the order of ~79 h at 5k cells; absence of these behaviours suggests a misconfigured benchmark
- auc_roc_curve.csv must contain one averaged AUC-ROC value per DE-gene count from 1 to 150, each averaged over exactly 1000 simulation-and-scoring repetitions
- Higher FiRE scores must correspond to rare cells, and IQR-based thresholding (q3 + 1.5*iqr) should yield a minority subset consistent with the 5% planted rare group

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types; outlier cells may get submerged into minor cell clusters when clustering FiRE-detected rare cells with dropClust
- Doublet rate may vary between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness, limiting application to doublet detection
- Baseline methods fail or scale unbearably slowly as sample size grows to the order of tens of thousands (GiniClust beyond ~45k profiles, RaceID ~79 h for 5k cells), so full-grid comparisons may be impossible
- README notes FiRE was tested on Ubuntu 14.04/16.04 and results in the manuscript were generated with the Python module; inplace Python installs may break uninstallation of the generated .so file

## Evidence

- [results] FiRE struggled to detect the minor population with few DE genes but improves at >=20 DE genes: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime finding for FiRE on 68k PBMC data: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [discussion] FiRE provides scores rather than binary predictions: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [abstract] Existing methods scale poorly with sample size: "Existing algorithms to find rare cells scale unbearably slowly or terminate, as the sample size grows to the order of tens of thousands."
- [readme] README FiRE model construction parameters: [evidence span withheld — anchor status: external_doc]
