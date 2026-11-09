---
name: wilcoxon-rank-sum-testing-of-cell-proportions
description: Use when you have per-cell state labels from a single-cell classifier
  (e.g. TILPRED on GSE120575 CD8 T cells) paired pre/post treatment across a handful
  of patients, and need a p-value for state-proportion shifts between responder and
  non-responder groups.
when_to_use_negative:
- Input is raw counts or unclassified cells — you must first assign per-cell states
  (e.g. with TILPRED) before computing state proportions
- The data are murine (e.g. B16CD8TIL_SCE example SCE) — predictTilState must be run
  with human=FALSE; the GSE120575 responder-analysis workflow assumes human=TRUE
- You have many matched samples and can assume normality of proportion differences
  — a paired parametric test would be more appropriate than a rank-sum test on 7 patients
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TILPRED
  role: Logistic regression-based classifier assigning each CD8 T cell a state (exhausted,
    memory-like, naive, EM-like) to derive per-patient EM-like proportions
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Container for the CD8 T cell count matrix passed to predictTilState()
- name: AUCell
  role: Bioconductor dependency used by TILPRED for curated gene-signature enrichment
    scoring during state assignment
- name: R
  role: Runtime for TILPRED and for the one-sided Wilcoxon rank-sum test (e.g. wilcox.test)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:4d7a9bc41f0f04fb601824129e2d49c4943bff845fb5f8118542e2f15b78e554
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  repo_url: https://github.com/carmonalab/TILPRED
  tools:
  - AUCell
  - BiocManager
  - R
  - SingleCellExperiment
  - TILPRED
  - doParallel
  - doRNG
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Wilcoxon rank-sum testing of cell proportions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-parametric one-sided Wilcoxon rank-sum test on per-patient proportions of a predicted CD8 TIL state (e.g. EM-like, as assigned by TILPRED) before versus after ICB therapy, comparing responders to non-responders to determine whether treatment induces expansion of that state.

## When to use

Use when you have per-cell state labels from a single-cell classifier (e.g. TILPRED on GSE120575 CD8 T cells) paired pre/post treatment across a handful of patients, and need a p-value for state-proportion shifts between responder and non-responder groups.

## When NOT to use

- Input is raw counts or unclassified cells — you must first assign per-cell states (e.g. with TILPRED) before computing state proportions
- The data are murine (e.g. B16CD8TIL_SCE example SCE) — predictTilState must be run with human=FALSE; the GSE120575 responder-analysis workflow assumes human=TRUE
- You have many matched samples and can assume normality of proportion differences — a paired parametric test would be more appropriate than a rank-sum test on 7 patients

## Inputs

- SingleCellExperiment object containing the single-cell expression matrix of CD8 T cells (GSE120575 melanoma, human mode)
- Per-cell predictedState labels and stateProbabilityMatrix from TILPRED predictTilState()
- Per-patient metadata mapping samples to patient ID, pre/post-ICB timepoint, and responder status

## Outputs

- Per-patient EM-like proportions before and after ICB
- One-sided Wilcoxon rank-sum test statistic and p-value (expected p = 0.0317)
- CSV file emlike_wilcoxon_results.csv with per-cell predictions and test results

## How to apply

First obtain per-cell predicted states by running predictTilState(data=sce, human=TRUE, scoreThreshold=0.3) on a SingleCellExperiment object of CD8 T cells (e.g. GSE120575 melanoma), extracting the predictedState labels and state probability matrix. Restrict to patients with matched pre/post-ICB samples and at least 30 cells per sample (here: responders P1, P7, P28; non-responders P2, P3, P12, P20). Compute each patient's proportion of EM-like cells before and after therapy, then run a one-sided Wilcoxon rank-sum test comparing responders versus non-responders on those per-patient proportions. A rank-sum test is appropriate because only 7 patients are available, so normality cannot be assumed. Save the per-cell predictions and test statistic/p-value to emlike_wilcoxon_results.csv for reproducibility.

## Related tools

- **TILPRED** (Logistic regression-based classifier assigning each CD8 T cell a state (exhausted, memory-like, naive, EM-like) to derive per-patient EM-like proportions) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Container for the CD8 T cell count matrix passed to predictTilState())
- **AUCell** (Bioconductor dependency used by TILPRED for curated gene-signature enrichment scoring during state assignment)
- **R** (Runtime for TILPRED and for the one-sided Wilcoxon rank-sum test (e.g. wilcox.test))

## Examples

```
res <- wilcox.test(em_prop_pre[responders], em_prop_post[responders], alternative = "less", exact = TRUE)  # one-sided test of EM-like increase upon ICB
```

## Evaluation signals

- Reproduced one-sided Wilcoxon p-value matches the reported p = 0.0317
- Filtered cohort contains exactly the seven patients with paired pre/post samples and >= 30 CD8 T cells (P1, P7, P28 responders; P2, P3, P12, P20 non-responders)
- Responders, not non-responders, show the increased EM-like proportion upon therapy (sign check on the per-patient pre/post differences)
- Per-cell labels fall within the expected TILPRED state set (exhausted, memory-like, naive, EM-like) and non-CD8 cells are labeled Treg, CD4T, NKT, Tcell_unknown or Non-Tcell
- emlike_wilcoxon_results.csv contains both per-cell predictions and the test statistic/p-value

## Limitations

- TILPRED is no longer maintained; the authors recommend using ProjecTILs instead
- Very small n (3 responders, 4 non-responders) limits statistical power and generalizability of the rank-sum comparison
- Minimum of 30 cells per sample required, so low-cell-count samples must be excluded from the paired analysis

## Evidence
