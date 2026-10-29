---
name: marker-gene-list-validation
description: Use when a dimensionality-reduction/feature-selection method produces
  a ranked gene list that must be checked for biological plausibility against known
  cell-type markers before downstream use.
when_to_use_negative:
- Input is simulated scRNA-seq data without known cell-type markers — the validation
  relies on curated marker references for real datasets only
- The gene list is a raw variable-feature list (e.g., Seurat FindVariableFeatures
  output) rather than the post-cutoff EDGE feature gene set — the μ − 1.5 × σ step
  must run first
- The dataset was restricted to fewer than 1000 top variable genes, which would change
  the candidate pool and invalidate the expected list sizes
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EDGE
  role: Produces the ranked feature gene list via ensemble weak learners and the μ
    − 1.5 × σ entropy cutoff that is then validated against markers
  repo: https://github.com/shawnstat/EDGE
- name: Seurat
  role: Used to compute the 1000 top variable genes that define the candidate gene
    pool for real datasets before EDGE runs
- name: R
  role: Runtime environment in which EDGE, Seurat, and RSpectra are executed for feature
    gene identification and validation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:119ecb6ff497f65aeb50aabde03f7b75e5db74335e801ec84a1ed26932c1005c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# Marker gene list validation

## Summary

Validates a computationally derived feature gene list (e.g., from EDGE's μ − 1.5 × σ entropy cutoff) by cross-referencing it against a curated marker reference and checking that known cell-type markers (e.g., PBMC markers PPBP, LYZ, CST3, NKG7, S100A9) and expected list sizes (17, 35, 43 genes for Jurkat, PBMC, mouse brain) are recovered.

## When to use

Use when a dimensionality-reduction/feature-selection method produces a ranked gene list that must be checked for biological plausibility against known cell-type markers before downstream use.

## When NOT to use

- Input is simulated scRNA-seq data without known cell-type markers — the validation relies on curated marker references for real datasets only
- The gene list is a raw variable-feature list (e.g., Seurat FindVariableFeatures output) rather than the post-cutoff EDGE feature gene set — the μ − 1.5 × σ step must run first
- The dataset was restricted to fewer than 1000 top variable genes, which would change the candidate pool and invalidate the expected list sizes

## Inputs

- Ranked feature gene list with averaged entropy contribution scores per gene (output of EDGE endr())
- Curated marker gene reference table (Supplementary Data 1)
- Log2-normalized cells-by-genes scRNA-seq matrices (Jurkat, PBMC, mouse brain) restricted to 1000 top variable genes

## Outputs

- Per-dataset feature gene count (expected: 17 Jurkat, 35 PBMC, 43 mouse brain)
- Marker gene recovery report mapping recovered markers to cell types (PPBP–platelet, LYZ–CD14+ monocyte, CST3–dendritic cell, NKG7–NK cell)
- Top-scoring gene call (S100A9 with highest importance score in PBMC)

## How to apply

First generate the feature gene list by ranking genes on averaged entropy contribution across EDGE's 1000 weak learners and applying the μ − 1.5 × σ cutoff; restrict real scRNA-seq inputs to the 1000 top variable genes beforehand (500 for simulated data). Then compare the ranked feature list against the curated marker reference (e.g., Supplementary Data 1) for each dataset, recording which known markers appear. For the PBMC dataset, verify recovery of PPBP (platelets), LYZ (CD14+ monocytes), CST3 (dendritic cells), and NKG7 (natural killer cells), and confirm S100A9 receives the highest importance score. Finally, check that list sizes match the published counts of 17, 35, and 43 feature genes for Jurkat, PBMC, and mouse brain respectively.

## Related tools

- **EDGE** (Produces the ranked feature gene list via ensemble weak learners and the μ − 1.5 × σ entropy cutoff that is then validated against markers) — https://github.com/shawnstat/EDGE
- **Seurat** (Used to compute the 1000 top variable genes that define the candidate gene pool for real datasets before EDGE runs)
- **R** (Runtime environment in which EDGE, Seurat, and RSpectra are executed for feature gene identification and validation)

## Examples

```
custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Feature gene counts must equal 17, 35, and 43 for the Jurkat, PBMC, and mouse brain datasets respectively
- The PBMC feature list must contain PPBP, LYZ, CST3, and NKG7, corresponding to platelet, CD14+ monocyte, dendritic cell, and NK cell markers
- S100A9 must appear in the PBMC list with the highest importance score
- Recovered markers and list membership must match the entries in Supplementary Data 1

## Limitations

- The hash table size in the runnable example (101,107) is reduced from the optimal 1,017,881 to save memory, which may slightly affect similarity learning and thus the entropy scores that determine the feature list
- Large hash table sizes increase memory requirements, constraining how faithfully the published feature gene sets can be reproduced on limited hardware
- Validation depends on the completeness of the external marker reference (Supplementary Data 1); markers absent from that reference cannot confirm sensitivity

## Evidence

- [discussion] cutoff definition: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [discussion] gene pool restriction: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
- [readme] hash table size caveat: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example."
- [abstract] ensemble scheme: "the proposed method implements an ensemble learning scheme that utilizes massive weak learners for an accurate similarity search"
