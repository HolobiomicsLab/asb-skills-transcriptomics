---
name: scrna-seq-dropout-imputation-benchmarking
description: Use when a QC-filtered scRNA-seq count matrix has high dropout (40%-90%
  missing values) and the goal is to compare imputation methods' downstream effects
  on clustering, visualization, DE analysis, or pseudotime.
when_to_use_negative:
- Input counts are already normalized and log-transformed — Bfimpute's CPM + log10(bias
  1.01) step assumes raw counts (set normalized=TRUE only if you will log-transform
  yourself)
- The dataset is a bulk RNA-seq or non-single-cell expression matrix, since the dropout
  phenomenon motivating imputation (40%-90% missing) is specific to scRNA-seq
- You need a guaranteed-best method for a new dataset — the article's discussion notes
  no consensus best imputer exists and Bfimpute's generalizability beyond benchmarked
  datasets is unconfirmed
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Bfimpute
  role: 'Primary imputation method: Bayesian factorization of the count matrix into
    gene-specific (G) and cell-specific (C) latent matrices via Gibbs-sampling MCMC;
    the imputed matrix is R_calculate = G^T C'
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparator imputation method run with recommended default settings on the
    same raw matrices
- name: SAVER
  role: Comparator imputation method run with recommended default settings
- name: VIPER
  role: Comparator imputation method relying on non-negative sparse regression over
    local neighborhood cells
- name: DrImpute
  role: Comparator imputation method averaging imputations from multiple clustering
    results
- name: MAGIC
  role: Comparator imputation method building a Markov affinity-based graph for cell-to-cell
    interaction smoothing
- name: PBLR
  role: Comparator low-rank matrix-factorization imputation using an alternating direction
    method of multipliers algorithm
- name: Spectrum
  role: Spectral clustering used by Bfimpute to group cells when labels are unavailable
    (ccluster = c(7,"Spectrum"))
- name: splatter
  role: Simulates scRNA-seq count matrices with 5-8 cell types in 'experiment' dropout
    mode for ground-truth clustering benchmarks
- name: DESeq2
  role: Identifies the top-200 bulk DE genes as reference and computes scRNA-seq DE
    calls for bulk-concordance AUC analysis
merged_aliases:
- scrna-seq-dropout-imputation
merged_alias_records:
- alias: scrna-seq-dropout-imputation
  slug: scrna-seq-dropout-imputation
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
- alias: scrna-seq-imputation-benchmarking
  slug: scrna-seq-imputation-benchmarking
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
- alias: scrnaseq-dropout-imputation-benchmarking
  slug: scrnaseq-dropout-imputation-benchmarking
  cosine_score: 0.9274
  method: embedding-cosine
  decision: auto
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_008
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:54b8900e4e1481dfd32fa35f1460ee0da7eb59168126c2c2ac624ee783e35497
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - BISCUIT
  - Bfimpute
  - DCA
  - DESeq2
  - DeepImpute
  - DrImpute
  - MAGIC
  - PBLR
  - R stats
  - SAVER
  - Spectrum
  - VIPER
  - devtools
  - kernlab
  - mnormt
  - parallel
  - rsvd
  - scISR
  - scImpute
  - scScope
  - splatter
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# scrna-seq-dropout-imputation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq dropout imputation methods (Bfimpute, scImpute, SAVER, VIPER, DrImpute, MAGIC, PBLR, ALRA, mcImpute) on simulated (Splatter) and real datasets (Chu, Tang, Petropoulos, Zheng, Vladoiu) using clustering accuracy, silhouette, Pearson correlation, DE-concordance AUC, pseudotime, and runtime metrics.

## When to use

Use when a QC-filtered scRNA-seq count matrix has high dropout (40%-90% missing values) and the goal is to compare imputation methods' downstream effects on clustering, visualization, DE analysis, or pseudotime.

## When NOT to use

- Input counts are already normalized and log-transformed — Bfimpute's CPM + log10(bias 1.01) step assumes raw counts (set normalized=TRUE only if you will log-transform yourself)
- The dataset is a bulk RNA-seq or non-single-cell expression matrix, since the dropout phenomenon motivating imputation (40%-90% missing) is specific to scRNA-seq
- You need a guaranteed-best method for a new dataset — the article's discussion notes no consensus best imputer exists and Bfimpute's generalizability beyond benchmarked datasets is unconfirmed

## Inputs

- QC-filtered scRNA-seq count matrix (genes x cells CSV, e.g., sc_qc.csv)
- Bulk RNA-seq count matrix (e.g., bulk_qc.csv) for gene side-information (S_G) and DE reference
- Cell type label vector/data.frame (e.g., cell_type.csv) for labeled-mode imputation
- Splatter-simulated count matrices with known ground-truth labels ('experiment' dropout mode)

## Outputs

- Per-method clustering metric tables (ARI, Jaccard index, NMI, purity) per dataset/seed
- Silhouette score and same/different-cell-type Pearson correlation summary tables (e.g., CSV)
- DE-concordance AUC table vs bulk top-200 DE genes across FDR thresholds (de_concordance_auc.csv)
- TSCAN pseudotime scores (POS, Kendall correlation) and runtime table (e.g., vladoiu_imputation_benchmark_results.csv)
- t-SNE embeddings of raw and imputed matrices for cluster-separation assessment

## How to apply

Load the QC'd counts, bulk matrix, and cell-type labels (e.g., sc_qc.csv, bulk_qc.csv, cell_type.csv from Chu et al. 2016 via read.csv with row.names=1). Impute with Bfimpute using either spectral clustering with the known number of cell types (e.g., ccluster = c(7,"Spectrum")) or labels (ccluster = "labeled", label = cell_type), letting Bfimpute perform CPM and log10 transform with bias 1.01 (normalized=FALSE), and defaults D=32, totalepoch=300, burnin=200, sn_max=10, sn_init=1, threshold=0.5, ncores=5. Run comparators (scImpute, SAVER, VIPER, DrImpute, MAGIC, PBLR, ALRA, mcImpute) with their recommended defaults on the same matrix. Evaluate each imputed matrix downstream: spectral clustering scored by ARI, Jaccard index, NMI, and purity; silhouette scores with true labels; same- vs different-cell-type Pearson correlations; t-SNE visualization; DESeq2 DE concordance with the top-200 bulk DE genes swept over FDR thresholds (AUC); TSCAN pseudotime (POS and Kendall's rank correlation); and wall-clock runtime. For simulation studies, generate matrices with Splatter's 'experiment' dropout mode across 5-8 cell types (~20,000 genes, 500-800 cells) and repeat clustering metrics over 10 random seeds, aggregating mean +/- STD per method.

## Related tools

- **Bfimpute** (Primary imputation method: Bayesian factorization of the count matrix into gene-specific (G) and cell-specific (C) latent matrices via Gibbs-sampling MCMC; the imputed matrix is R_calculate = G^T C) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparator imputation method run with recommended default settings on the same raw matrices)
- **SAVER** (Comparator imputation method run with recommended default settings)
- **VIPER** (Comparator imputation method relying on non-negative sparse regression over local neighborhood cells)
- **DrImpute** (Comparator imputation method averaging imputations from multiple clustering results)
- **MAGIC** (Comparator imputation method building a Markov affinity-based graph for cell-to-cell interaction smoothing)
- **PBLR** (Comparator low-rank matrix-factorization imputation using an alternating direction method of multipliers algorithm)
- **Spectrum** (Spectral clustering used by Bfimpute to group cells when labels are unavailable (ccluster = c(7,"Spectrum")))
- **splatter** (Simulates scRNA-seq count matrices with 5-8 cell types in 'experiment' dropout mode for ground-truth clustering benchmarks)
- **DESeq2** (Identifies the top-200 bulk DE genes as reference and computes scRNA-seq DE calls for bulk-concordance AUC analysis)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), D = 32, totalepoch = 300, burnin = 200, threshold = 0.5, ncores = 5)
```

## Evaluation signals

- Same-cell-type Pearson correlation on Chu rises from ~0.70 (raw) toward ~0.87 for Bfimpute (0.85 scImpute, 0.72 SAVER, 0.73 VIPER, 0.78 DrImpute, 0.97 MAGIC) while Bfimpute best preserves cross-type variation
- Bfimpute attains the best silhouette score in all but one of the labeled benchmark datasets, and best ARI/Jaccard/NMI/purity on Splatter simulations with 5-8 cell types
- DE-concordance AUC on Chu: raw 0.568; Bfimpute 0.670; scImpute 0.665; SAVER 0.624; VIPER 0.639; DrImpute 0.657; MAGIC 0.668
- Vladoiu 10x benchmark (62,893 cells, 91.9% dropout): Bfimpute achieves best TSCAN POS/Kendall and clustering scores; runtime ordering reproduces MAGIC < PBLR < Bfimpute < scImpute < SAVER (DrImpute and VIPER fail)
- All simulated-clustering metric values fall within [0,1] and metric tables contain the required ARI/Jaccard/nmi/purity columns aggregated as mean +/- STD over 10 seeds

## Limitations

- No single best imputation method exists; performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses
- It is premature to conclude Bfimpute will outperform across all datasets; generalizability beyond the benchmarked datasets is unconfirmed
- Imputation may increase false positives in downstream analysis; users must tune parameters per application to maximize true positives or minimize false positives
- Larger D, totalepoch, and burnin increase accuracy but proportionally increase compute time; DrImpute and VIPER failed to run on the 62,893-cell Vladoiu dataset

## Evidence
