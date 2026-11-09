---
name: imputation-benchmark-comparison
description: Use when a new or existing scRNA-seq imputation method needs head-to-head
  evaluation against published imputers on simulated or real developmental datasets,
  scored by downstream pseudotime, clustering, or correlation metrics.
when_to_use_negative:
- The goal is a single-dataset method application rather than a comparative benchmark
  — no competitor imputers or ground-truth labels/time orderings are available
- Input data lacks QC or cell-type/trajectory ground truth, making POS/Kendall or
  clustering-based evaluation meaningless
- Generalizing conclusions beyond benchmarked datasets is desired — the paper cautions
  it is premature to conclude Bfimpute will outperform across all datasets
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Bfimpute
  role: Target Bayesian factorization imputation method being benchmarked; supports
    gene/cell side information (S_G, S_C) and spectral or labeled clustering
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Competitor imputation method run on the same three datasets for comparison
- name: DrImpute
  role: Competitor imputation method that averages imputation from multiple clustering
    results
- name: TSCAN
  role: Pseudotime inference tool applied to raw and imputed matrices to produce orderings
    scored by POS and Kendall's tau
- name: Monocle
  role: Second pseudotime inference tool used for ordering evaluation across all datasets
- name: Spectrum
  role: Spectral clustering used inside Bfimpute (ccluster = c(7,"Spectrum")) to group
    cells before imputation
- name: SCRABBLE
  role: Bulk-data-constrained comparator imputation method
- name: netNMF-sc
  role: Gene-gene-interaction-based comparator imputation method
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:c2c52aa668e1b5acfec8b46c47a5a600e1dc3e71eced9606e5fd4bbe0400e08f
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
  - SCRABBLE
  - Spectrum
  - TSCAN
  - netNMF-sc
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# imputation-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a scRNA-seq imputation method (e.g., Bfimpute) against competing imputers (scImpute, DrImpute, SCRABBLE, netNMF-sc) on raw and imputed count matrices using downstream tasks such as pseudotime inference (POS, Kendall's tau), PCA clustering, and cell-to-cell correlation, as done in the Bfimpute paper (Cell Reports Methods 2022).

## When to use

Use when a new or existing scRNA-seq imputation method needs head-to-head evaluation against published imputers on simulated or real developmental datasets, scored by downstream pseudotime, clustering, or correlation metrics.

## When NOT to use

- The goal is a single-dataset method application rather than a comparative benchmark — no competitor imputers or ground-truth labels/time orderings are available
- Input data lacks QC or cell-type/trajectory ground truth, making POS/Kendall or clustering-based evaluation meaningless
- Generalizing conclusions beyond benchmarked datasets is desired — the paper cautions it is premature to conclude Bfimpute will outperform across all datasets

## Inputs

- QC-filtered scRNA-seq count matrices (e.g., Chu sc_qc.csv from GSE75748, Scialdone GSE74994, Petropoulos E-MTAB-3929)
- Cell type labels (cell_type.csv) or expected number of clusters (e.g., 7 for Chu)
- Optional bulk RNA-seq matrix (bulk_qc.csv) for gene side information S_G = t(bulk)
- Known developmental time labels for pseudotime scoring

## Outputs

- Imputed expression matrices from each method (Bfimpute.csv/txt/rds, scImpute, DrImpute, SCRABBLE, netNMF-sc outputs)
- Pseudotime orderings from TSCAN and Monocle on raw and imputed matrices
- 3-datasets x 2-tools x 4-methods table of POS and Kendall's rank correlation scores
- PCA/correlation comparison figures (e.g., Figures S1I-S1J pattern)

## How to apply

Run the target imputer (Bfimpute) and competitors (scImpute, DrImpute; optionally bulk-constrained SCRABBLE and netNMF-sc) on the same QC-filtered count matrices, e.g., Chu (GSE75748), Scialdone (GSE74994), and Petropoulos (E-MTAB-3929). Apply Bfimpute with defaults D=32, totalepoch=300, burnin=200, threshold=0.5, ncores=5, CPM normalization with log10 transform (bias 1.01), and spectral clustering via c(7,"Spectrum") for the 7-cell-type Chu data. Infer pseudotime orderings on raw and each imputed matrix with TSCAN and Monocle, then score against known developmental ordering using POS (percent of overlap) and Kendall's rank correlation. For side-information comparisons, supply S_G = t(bulk) so bulk RNA-seq columns align with count-matrix gene rows, and compare PCA/correlation profiles across methods. Verify reproductions against reported values such as Chu TSCAN raw POS 0.610/Kendall 0.317 improving to Bfimpute 1.000/0.902, and confirm no significant gain from bulk side information (Figures S1I-S1J).

## Related tools

- **Bfimpute** (Target Bayesian factorization imputation method being benchmarked; supports gene/cell side information (S_G, S_C) and spectral or labeled clustering) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Competitor imputation method run on the same three datasets for comparison)
- **DrImpute** (Competitor imputation method that averages imputation from multiple clustering results)
- **TSCAN** (Pseudotime inference tool applied to raw and imputed matrices to produce orderings scored by POS and Kendall's tau)
- **Monocle** (Second pseudotime inference tool used for ordering evaluation across all datasets)
- **Spectrum** (Spectral clustering used inside Bfimpute (ccluster = c(7,"Spectrum")) to group cells before imputation)
- **SCRABBLE** (Bulk-data-constrained comparator imputation method)
- **netNMF-sc** (Gene-gene-interaction-based comparator imputation method)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- Reproduced Chu TSCAN scores match Table 1: raw POS 0.610 / Kendall 0.317 improving to Bfimpute 1.000 / 0.902
- Bfimpute increases POS and Kendall's tau over raw data and competitor imputers across Chu, Scialdone, and Petropoulos datasets
- Adding bulk side information via S_G = t(bulk) yields no significant performance difference for Bfimpute (Figures S1I-S1J)
- SCRABBLE and netNMF-sc, despite bulk or gene-interaction constraints, do not outperform Bfimpute in PCA/correlation comparisons
- Imputed output files (Bfimpute.csv, Bfimpute.txt, Bfimpute.rds) are saved with default out_type = "all" and share gene/cell dimensions with the input count matrix

## Limitations

- No single best imputation method exists; performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses
- It is premature to conclude Bfimpute will outperform across all datasets or improve all downstream analyses
- Imputation may increase false positives in downstream analysis; parameters must be tuned per application to balance true vs. false positives
- Larger D, totalepoch, and burnin improve accuracy but increase runtime

## Evidence
