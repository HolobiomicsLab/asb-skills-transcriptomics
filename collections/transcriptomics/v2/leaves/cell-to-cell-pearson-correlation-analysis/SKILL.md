---
name: cell-to-cell-pearson-correlation-analysis
description: Use when you have imputed (or raw) gene-by-cell count matrices plus cell
  type labels and need to measure whether dropout recovery improves same-cell-type
  Pearson correlation and clustering separation without over-homogenizing cell types.
when_to_use_negative:
- Input is a bulk RNA-seq or already-aggregated expression profile — the same/different-cell-type
  Pearson comparison requires single-cell matrices with known cell labels
- No cell type labels or cluster assignments exist — silhouette scoring with true
  labels is undefined and only unsupervised ARI/Jaccard/NMI/purity workflows apply
- The goal is differential expression or pseudotime benchmarking rather than assessing
  cell-to-cell relationship recovery, which the article evaluates separately
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Primary imputation method whose labeled/spectral-clustering output matrices
    are evaluated via same/different-cell-type Pearson correlation and silhouette
    scores
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparator imputation method run on the same count matrix at default settings
    before correlation evaluation
- name: SAVER
  role: Comparator Bayesian imputation method in the unsupervised Pearson-correlation
    benchmark
- name: VIPER
  role: Comparator neighborhood-based imputation method in the unsupervised Pearson-correlation
    benchmark
- name: DrImpute
  role: Comparator multi-clustering imputation method evaluated in both labeled and
    unlabeled correlation benchmarks
- name: MAGIC
  role: Comparator graph-based imputation method; achieves the highest same-cell-type
    Pearson (~0.97) but is distinguished by poorer cross-type variation preservation
- name: PBLR
  role: Comparator low-rank ADMM imputation method in the labeled benchmark
- name: Spectrum
  role: Spectral clustering used inside Bfimpute (ccluster=c(7,"Spectrum")) and downstream
    clustering-accuracy evaluation
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:def3cf1bdbc849fda215da57994b5c6c12876d23cca464118765c816f272a57c
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
  - MAGIC
  - PBLR
  - SAVER
  - Spectrum
  - VIPER
  - devtools
  - scImpute
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cell-to-cell-pearson-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes same-cell-type versus different-cell-type Pearson correlations of imputed scRNA-seq expression matrices (plus silhouette scores with true labels) to quantify whether imputation recovers cell-to-cell relationships, as used to benchmark Bfimpute against scImpute, SAVER, VIPER, DrImpute, MAGIC, PBLR, ALRA, and mcImpute on Chu, Tang, Petropoulos, and Zheng datasets.

## When to use

Use when you have imputed (or raw) gene-by-cell count matrices plus cell type labels and need to measure whether dropout recovery improves same-cell-type Pearson correlation and clustering separation without over-homogenizing cell types.

## When NOT to use

- Input is a bulk RNA-seq or already-aggregated expression profile — the same/different-cell-type Pearson comparison requires single-cell matrices with known cell labels
- No cell type labels or cluster assignments exist — silhouette scoring with true labels is undefined and only unsupervised ARI/Jaccard/NMI/purity workflows apply
- The goal is differential expression or pseudotime benchmarking rather than assessing cell-to-cell relationship recovery, which the article evaluates separately

## Inputs

- Raw QC-filtered scRNA-seq count matrix (e.g. sc_qc.csv from Chu GSE75748 or Tang GSE100911), genes as rows, cells as columns
- Cell type label file (cell_type.csv) for the labeled-mode evaluation
- Imputed expression matrices from Bfimpute and comparator methods (scImpute, SAVER, VIPER, DrImpute, MAGIC, PBLR, ALRA, mcImpute)
- Optional bulk matrix (bulk_qc.csv) if using Bfimpute with S_G side information

## Outputs

- Per-method same-cell-type vs different-cell-type Pearson correlation values
- Per-method silhouette scores computed with true cell type labels
- Summary CSV tables of clustering metrics (ARI, Jaccard, NMI, purity) and correlation barplot data

## How to apply

After imputing each raw count matrix with Bfimpute (using either ccluster=c(7,"Spectrum") spectral clustering or ccluster="labeled" with true cell_type labels; defaults D=32, totalepoch=300, burnin=200, threshold=0.5, normalized=FALSE so Bfimpute performs CPM and log10 transform with bias 1.01) and with comparator methods at their defaults, compute same-cell-type and different-cell-type Pearson correlations of gene expression between cells for every matrix including raw counts. In parallel, compute silhouette scores using the true cell type labels. On the Chu dataset the expectation is same-cell-type Pearson rising from ~0.70 (raw) to ~0.87 for Bfimpute (~0.85 scImpute, ~0.72 SAVER, ~0.73 VIPER, ~0.78 DrImpute, ~0.97 MAGIC), so a high correlation alone is not sufficient — Bfimpute's advantage is best preserved cross-type variation alongside high within-type correlation. Aggregate per-method silhouette and correlation values into summary tables saved as CSV, mirroring the Figure 3 barplot data and Figure 6/Table S1 metrics.

## Related tools

- **Bfimpute** (Primary imputation method whose labeled/spectral-clustering output matrices are evaluated via same/different-cell-type Pearson correlation and silhouette scores) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparator imputation method run on the same count matrix at default settings before correlation evaluation)
- **SAVER** (Comparator Bayesian imputation method in the unsupervised Pearson-correlation benchmark)
- **VIPER** (Comparator neighborhood-based imputation method in the unsupervised Pearson-correlation benchmark)
- **DrImpute** (Comparator multi-clustering imputation method evaluated in both labeled and unlabeled correlation benchmarks)
- **MAGIC** (Comparator graph-based imputation method; achieves the highest same-cell-type Pearson (~0.97) but is distinguished by poorer cross-type variation preservation)
- **PBLR** (Comparator low-rank ADMM imputation method in the labeled benchmark)
- **Spectrum** (Spectral clustering used inside Bfimpute (ccluster=c(7,"Spectrum")) and downstream clustering-accuracy evaluation)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- On Chu, same-cell-type Pearson correlation should rise from ~0.70 (raw) to ~0.87 for Bfimpute and ~0.97 for MAGIC; a drop below the raw value indicates an imputation or normalization mismatch
- Bfimpute should attain the best silhouette score in all but one of the labeled benchmark datasets (Chu, Tang, Petropoulos, Zheng), matching the reported labeled comparison
- Cross-type (different-cell-type) Pearson correlation must remain low for Bfimpute despite high within-type correlation, distinguishing it from MAGIC's over-smoothing
- All methods must be evaluated on identically preprocessed matrices — Bfimpute applies CPM then log10 transform with bias 1.01 when normalized=FALSE, so raw counts are the expected input
- Per-method summary CSVs should mirror the Figure 3A-3C barplot values and Figure 6/Table S1 silhouette and correlation tables

## Limitations

- MAGIC achieves the highest same-cell-type Pearson correlation (~0.97 vs ~0.87 for Bfimpute), so high within-type correlation alone is not proof of better imputation — cross-type variation preservation must be checked
- The article's discussion notes there is no consensus on the best imputation method and it is premature to conclude Bfimpute outperforms across all datasets and downstream analyses
- Depending on the application, users may need to constrain parameters to maximize true positives or minimize false positives, and larger D, totalepoch, or burnin raise accuracy at higher compute cost

## Evidence

- [intro] Prior methods effective for point estimates but weaker at cell-to-cell recovery: "they were less effective in recovering cell-to-cell relationships resulting in less improvement in cell clustering and trajectory analysis"
- [abstract] Bfimpute imputation within cell groups: "to impute the final gene expression matrix within each cell group, with or without the aid of cell type labels or bulk data"
- [readme] Bfimpute normalization behavior on raw counts: "If `FALSE`, Bfimpute will perform CPM and log10 transform with bias 1.01."
- [readme] Labeled-mode invocation used before correlation evaluation: "counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)"
