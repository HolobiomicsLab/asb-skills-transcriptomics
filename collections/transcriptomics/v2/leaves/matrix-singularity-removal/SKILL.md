---
name: matrix-singularity-removal
description: Use when supplying a gene-by-geneset or TF-gene prior matrix (e.g., 'C_dorothea_tf_human.refined.rds')
  to a model like GEDI that requires a full-rank C, and singularity must be eliminated
  before model fitting.
when_to_use_negative:
- The prior matrix is already full rank (QR rank equals column count) — truncation
  is a no-op and reordering may be unnecessary
- The downstream method accepts rank-deficient priors or uses regularization that
  handles collinearity without requiring full rank
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GEDI
  role: Downstream model whose fit requires the prior matrix C to be free of singular
    columns
  repo: https://github.com/csglab/GEDI
- name: DoRothEA
  role: Source of the human gene regulatory network whose refined TF-gene matrix is
    pruned for singularity
- name: R (base QR decomposition)
  role: Provides qr(), $pivot and $rank used to identify and drop singular columns
    of C
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:095b5ead506281d1c9fd89dc7e4aa3b8088a94d0f6cecd49289466cc0ac47330
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - DoRothEA
  - GEDI
  - HDF5Array
  - R (base QR decomposition)
  - ggplot2
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 21
---

# matrix-singularity-removal

## Summary

Removes linearly dependent (singular) columns from a prior knowledge matrix (e.g., the DoRothEA-derived TF-gene matrix C used in GEDI) via QR decomposition so downstream latent-variable models can be fitted without numerical failure.

## When to use

Use when supplying a gene-by-geneset or TF-gene prior matrix (e.g., 'C_dorothea_tf_human.refined.rds') to a model like GEDI that requires a full-rank C, and singularity must be eliminated before model fitting.

## When NOT to use

- The prior matrix is already full rank (QR rank equals column count) — truncation is a no-op and reordering may be unnecessary
- The downstream method accepts rank-deficient priors or uses regularization that handles collinearity without requiring full rank

## Inputs

- DoRothEA-derived TF-gene prior matrix C (RDS, e.g., 'C_dorothea_tf_human.refined.rds') subset to shared genes in the same order as the count matrix
- Raw count matrix (dgCMatrix, e.g., from a pre-processed PBMC SingleCellExperiment) used to determine shared genes

## Outputs

- Rank-truncated prior matrix C with singular columns removed, retaining QR$pivot[seq_len(QR$rank)] columns
- A GEDI model object fitted on the full-rank C (e.g., K = 40, mode = 'Bsphere') from which TF activities (getA.gedi, getADB.gedi) can be extracted

## How to apply

Align the prior matrix C and the count matrix to their shared genes in the same order, then run QR decomposition on C and retain only the pivot columns up to the rank: C <- C[, QR$pivot[seq_len(QR$rank)]]. This discards columns that would cause singularity. The pruned C is then passed to the GEDI model (e.g., K = 40 latent variables, mode = 'Bsphere', oi_shrinkage = 0.001, 500 iterations) so TF regulon activities can be projected reliably. Rationale: GEDI requires a full-rank prior, and redundant TF columns from regulatory networks like DoRothEA break the fit.

## Related tools

- **GEDI** (Downstream model whose fit requires the prior matrix C to be free of singular columns) — https://github.com/csglab/GEDI
- **DoRothEA** (Source of the human gene regulatory network whose refined TF-gene matrix is pruned for singularity)
- **R (base QR decomposition)** (Provides qr(), $pivot and $rank used to identify and drop singular columns of C)

## Examples

```
QR <- qr(C); C <- C[, QR$pivot[seq_len(QR$rank)]]
```

## Evaluation signals

- After pruning, ncol(C) equals QR$rank of the original matrix — verify length(QR$pivot[seq_len(QR$rank)]) matches retained columns
- The fitted GEDI model converges: model$plotTracking() shows stabilized components (o, Z, A, Bi, Qi, oi, si) over 500 iterations
- No linear-algebra errors ('singularity' / 'system is computationally singular') during model optimization
- Projected TF activities remain interpretable: e.g., 74 of 89 TFs show significant correlation between projected activity and decoded TF mRNA abundance, with PAX5 localized to B cells and TCF7 to CD4+ T cells

## Limitations

- Only applies when a full-rank prior is required; the pivot-based truncation may drop informative but redundant TF columns, slightly reducing prior coverage
- DoRothEA interactions must first be restricted to high-confidence sets A, B, and C, and C and the count matrix must be subset to shared genes in the same order before QR, otherwise the pruning is invalid

## Evidence

- [methods] Remove columns from C matrix that result in singularity: [evidence span withheld — anchor status: not_found]
- [methods] QR pivot truncation code: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [methods] DoRothEA prior source: "we downloaded the human [DoRothEA](https://saezlab.github.io/dorothea/) gene regulatory network"
- [readme] README notebook reference: [evidence span withheld — anchor status: external_doc]
