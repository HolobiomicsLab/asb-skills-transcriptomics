---
name: pre-post-immunotherapy-patient-sample-matching
description: Use when scRNA-seq data contain paired pre-/post-immunotherapy (ICB)
  samples per patient and you must select patients with both timepoints and adequate
  cell counts before testing whether a CD8 TIL state (e.g. EM-like) expands upon therapy
  in responders.
when_to_use_negative:
- Patient samples are unpaired (only one timepoint per patient) — a pre/post proportion-change
  comparison cannot be made
- A patient's sample has fewer than 30 CD8 T cells, making the per-patient EM-like
  proportion too noisy to include
- The goal is cell-state classification of non-CD8 populations (Treg, CD4T, NKT) rather
  than comparing paired CD8 state proportions across treatment timepoints
tools:
- name: TILPRED
  role: Logistic regression classifier that assigns each CD8 T cell a TIL state (exhausted,
    memory-like, naive, EM-like) and probability, providing the state labels whose
    per-patient proportions are matched pre/post ICB
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Container format for the CD8 T-cell expression matrix passed to predictTilState
  repo: https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- name: remotes
  role: Used to install TILPRED from its Git repository before classification
  repo: https://cran.r-project.org/web/packages/remotes/index.html
- name: AUCell
  role: Bioconductor dependency for the curated gene-signature enrichment scoring
    underlying TILPRED state assignment
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:37b3f424da8cbd2cbb3f5f128d0ee6886ba30075eaf8b8e1910686da5e4def23
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# pre-post-immunotherapy-patient-sample-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify and retain melanoma patients with matched pre- and post-ICB single-cell samples and sufficient CD8 T-cell depth (>=30 cells per sample) so that per-patient TIL state proportions (e.g. EM-like, via TILPRED) can be validly compared between responders and non-responders.

## When to use

Use when scRNA-seq data contain paired pre-/post-immunotherapy (ICB) samples per patient and you must select patients with both timepoints and adequate cell counts before testing whether a CD8 TIL state (e.g. EM-like) expands upon therapy in responders.

## When NOT to use

- Patient samples are unpaired (only one timepoint per patient) — a pre/post proportion-change comparison cannot be made
- A patient's sample has fewer than 30 CD8 T cells, making the per-patient EM-like proportion too noisy to include
- The goal is cell-state classification of non-CD8 populations (Treg, CD4T, NKT) rather than comparing paired CD8 state proportions across treatment timepoints

## Inputs

- scRNA-seq count matrices for CD8 T cells from patients sampled pre- and post-ICB (e.g. GSE120575 melanoma)
- SingleCellExperiment object containing the CD8 T-cell expression matrix
- Per-patient responder/non-responder labels
- Per-cell TILPRED predictedState labels and stateProbabilityMatrix

## Outputs

- Set of matched patients with paired pre/post samples and >=30 CD8 T cells per sample
- Per-patient EM-like proportions before and after ICB
- One-sided Wilcoxon rank-sum statistic and p-value (expected p = 0.0317 for GSE120575 responders vs non-responders)
- Per-cell predictions and test results saved to emlike_wilcoxon_results.csv

## How to apply

Assemble the patient CD8 T-cell count data (e.g. GSE120575 melanoma) into a SingleCellExperiment object and classify each cell with TILPRED via predictTilState(data=sce, human=TRUE, scoreThreshold=0.3), extracting predictedState labels and the stateProbabilityMatrix. Then filter to patients that have BOTH a pre- and a post-ICB sample and at least 30 CD8 T cells per sample — in GSE120575 this yields seven patients (responders P1, P7, P28; non-responders P2, P3, P12, P20). For each retained patient, compute the proportion of EM-like cells before and after ICB (non-CD8 calls like Treg, CD4T, NKT, Tcell_unknown or Non-Tcell are excluded from the CD8-state proportion). Finally, compare the change in EM-like proportion between responders and non-responders with a one-sided Wilcoxon rank-sum test; the rationale is that paired timepoints and a minimum cell-depth threshold prevent per-patient proportion estimates from being dominated by sampling noise.

## Related tools

- **TILPRED** (Logistic regression classifier that assigns each CD8 T cell a TIL state (exhausted, memory-like, naive, EM-like) and probability, providing the state labels whose per-patient proportions are matched pre/post ICB) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Container format for the CD8 T-cell expression matrix passed to predictTilState) — https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- **remotes** (Used to install TILPRED from its Git repository before classification) — https://cran.r-project.org/web/packages/remotes/index.html
- **AUCell** (Bioconductor dependency for the curated gene-signature enrichment scoring underlying TILPRED state assignment) — https://bioconductor.org/packages/release/bioc/html/AUCell.html

## Examples

```
preds <- predictTilState(data=sce, human=TRUE, scoreThreshold=0.3)
```

## Evaluation signals

- Exactly seven patients with paired pre/post ICB samples and >=30 CD8 T cells per sample are retained (responders P1, P7, P28; non-responders P2, P3, P12, P20)
- The reproduced one-sided Wilcoxon p-value matches the reported p = 0.0317
- Per-cell predicted states fall within the TILPRED label set: exhausted, memory-like, naive, EM-like, Treg, CD4T, NKT, Tcell_unknown, Non-Tcell
- Every per-patient EM-like proportion is within [0, 1] and computed only from CD8 T-cell state calls
- Output CSV emlike_wilcoxon_results.csv contains both per-cell predictions and the test statistic/p-value

## Limitations

- TILPRED is no longer maintained; the authors recommend using ProjecTILs instead
- Only seven matched patients met the paired >=30-cell filter, limiting statistical power of the Wilcoxon comparison
- Human CD8 TIL data carry large biological and technical variability, and antigen specificity is often unknown, complicating interpretation of state-proportion changes

## Evidence

- [results] responders had an increased proportion of EM-like cells upon therapy (p = 0.0317, one-sided Wilcoxon test): [evidence span withheld — anchor status: not_found]
