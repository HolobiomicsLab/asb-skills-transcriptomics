---
name: kendall-rank-correlation-evaluation
description: Use when comparing pseudotime/trajectory orderings of QC-filtered scRNA-seq
  developmental datasets against known time labels, and you need POS and Kendall's
  rank correlation to judge whether imputation improved ordering fidelity.
when_to_use_negative:
- Datasets without known time labels or a reference ordering — POS and Kendall's correlation
  are computed against the true developmental ordering, so the skill cannot be applied
- Steady-state (non-developmental) scRNA-seq data where pseudotime ordering evaluation
  is not meaningful
- Inputs that are already normalized/transformed — Bfimpute's CPM + log10 (bias 1.01)
  step assumes raw counts (normalized=FALSE); feeding pre-normalized matrices changes
  the imputation and downstream scores
tools:
- name: Bfimpute
  role: Imputation method being evaluated; produces imputed matrices whose pseudotime
    orderings are scored against true time labels
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Competing imputation method included in the 4-method comparison feeding the
    POS/Kendall score table
- name: DrImpute
  role: Competing imputation method included in the 4-method comparison feeding the
    POS/Kendall score table
- name: TSCAN
  role: Pseudotime inference tool applied to raw and each imputed matrix before scoring
- name: Monocle
  role: Pseudotime inference tool applied to raw and each imputed matrix before scoring
- name: Spectrum
  role: Spectral clustering approach used inside Bfimpute to group cells (e.g., c(7,"Spectrum"))
    before imputation of the Chu data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:3a1ccf377c8e4fa9ca4cdfd86a024ee32f5832c248656d22824849b774552c6d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - Monocle
  - Spectrum
  - TSCAN
  - scImpute
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# kendall-rank-correlation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score pseudotime orderings inferred from raw and imputed scRNA-seq developmental datasets (Chu, Scialdone, Petropoulos) against true time labels using POS (percent of overlap) and Kendall's rank correlation, to quantify whether imputation (e.g., Bfimpute) improves trajectory recovery over raw data and competing methods (scImpute, DrImpute).

## When to use

Use when comparing pseudotime/trajectory orderings of QC-filtered scRNA-seq developmental datasets against known time labels, and you need POS and Kendall's rank correlation to judge whether imputation improved ordering fidelity.

## When NOT to use

- Datasets without known time labels or a reference ordering — POS and Kendall's correlation are computed against the true developmental ordering, so the skill cannot be applied
- Steady-state (non-developmental) scRNA-seq data where pseudotime ordering evaluation is not meaningful
- Inputs that are already normalized/transformed — Bfimpute's CPM + log10 (bias 1.01) step assumes raw counts (normalized=FALSE); feeding pre-normalized matrices changes the imputation and downstream scores

## Inputs

- QC-filtered scRNA-seq count matrices (genes x cells, CSV) from Chu GSE75748, Scialdone GSE74994, Petropoulos E-MTAB-3929
- Known developmental time labels / true ordering per dataset
- Pseudotime orderings inferred by TSCAN and Monocle on raw and imputed matrices

## Outputs

- POS (percent of overlap) scores per dataset/tool/imputation-method combination
- Kendall's rank correlation scores per dataset/tool/imputation-method combination
- 3-datasets x 2-tools x 4-methods score table comparable to Table 1

## How to apply

Download QC-filtered count matrices for Chu (GSE75748), Scialdone (GSE74994), and Petropoulos (E-MTAB-3929), then impute each with Bfimpute (CPM + log10 transform with bias 1.01, D=32, totalepoch=300, burnin=200, threshold=0.5, ncores=5, e.g. ccluster=c(7,'Spectrum') for the 7-cell-type Chu data), scImpute, and DrImpute. Infer pseudotime orderings on the raw matrix and each imputed matrix with TSCAN and Monocle (3 datasets x 2 tools x 4 methods). Score every ordering against the known developmental ordering using POS and Kendall's rank correlation, and assemble the score table. Judge success by reproducing Table 1 values, e.g., Chu TSCAN raw POS 0.610 / Kendall 0.317 improving to Bfimpute 1.000 / 0.902, with Bfimpute outperforming raw data and competing imputation methods.

## Related tools

- **Bfimpute** (Imputation method being evaluated; produces imputed matrices whose pseudotime orderings are scored against true time labels) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Competing imputation method included in the 4-method comparison feeding the POS/Kendall score table)
- **DrImpute** (Competing imputation method included in the 4-method comparison feeding the POS/Kendall score table)
- **TSCAN** (Pseudotime inference tool applied to raw and each imputed matrix before scoring)
- **Monocle** (Pseudotime inference tool applied to raw and each imputed matrix before scoring)
- **Spectrum** (Spectral clustering approach used inside Bfimpute to group cells (e.g., c(7,"Spectrum")) before imputation of the Chu data)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- Reproduced Chu TSCAN scores match Table 1: raw POS 0.610 / Kendall 0.317 improving to Bfimpute POS 1.000 / Kendall 0.902
- Bfimpute increases both POS and Kendall's rank correlation relative to raw data for TSCAN and Monocle on Chu, and outperforms scImpute and DrImpute across Chu, Scialdone, and Petropoulos
- Score table has full coverage: 3 datasets x 2 tools (TSCAN, Monocle) x 4 methods (raw, Bfimpute, scImpute, DrImpute) = 24 score pairs
- All imputed matrices were produced with identical Bfimpute defaults (D=32, totalepoch=300, burnin=200, threshold=0.5, ncores=5) and CPM + log10 transform with bias 1.01 so comparisons are confound-free
- Kendall's values fall within expected range for the benchmark (e.g., 0.317 to 0.902 on Chu TSCAN); POS within [0, 1]

## Limitations

- Generalizability of Bfimpute beyond the benchmarked datasets is unconfirmed — the paper states it is premature to conclude Bfimpute achieves better performance across all datasets
- No single best imputation method exists; performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses
- Imputation can increase false positives in downstream analysis; parameters must be tuned per application
- Larger D, totalepoch, and burnin increase imputation accuracy but increase runtime

## Evidence

- [other] Bfimpute outperforms across pseudotime: [evidence span withheld — anchor status: not_found]
- [other] Table 1 benchmark values: [evidence span withheld — anchor status: not_found]
