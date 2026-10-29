---
name: clustering-consistency-metric-evaluation
description: Use when an imputation method (e.g., Bfimpute, scImpute, MAGIC, SAVER,
  VIPER, PBLR, DrImpute) must be benchmarked on data with known cell type labels,
  such as splatter-simulated matrices or labeled 10x datasets, to quantify clustering
  recovery.
when_to_use_negative:
- No ground-truth cell labels or simulation-known identities exist, making ARI/Jaccard/NMI/purity
  against truth undefined — use unsupervised metrics instead
- The goal is differential expression or gene-level recovery accuracy rather than
  cell-to-cell relationship recovery, where the paper notes prior methods 'were less
  effective in recovering cell-to-cell relationships'
- Evaluating pseudotime/trajectory quality alone, where TSCAN POS and Kendall correlation
  are the appropriate scores rather than label-agreement metrics
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Imputes the count matrix (spectral clustering with c(7,"Spectrum"), D=32,
    totalepoch=300, burnin=200, threshold=0.5) whose clustered output is scored against
    truth labels
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparison imputer run with default settings on the same matrices before metric
    evaluation
- name: DrImpute
  role: Comparison imputer averaging imputation over multiple clustering results
- name: MAGIC
  role: Markov affinity-based graph imputer used as a baseline in the clustering benchmark
- name: SAVER
  role: Bayesian prior-based comparison imputer
- name: VIPER
  role: Non-negative sparse regression neighborhood imputer used as a baseline
- name: PBLR
  role: Low-rank alternating-direction matrix-completion baseline imputer
- name: splatter
  role: R package used to simulate scRNA-seq count matrices with 5, 6, 7, and 8 cell
    types (20,000 genes, 500-800 cells) in 'experiment' dropout mode, providing ground-truth
    labels
- name: Spectrum
  role: Spectral clustering engine invoked through Bfimpute's ccluster parameter to
    group cells before metric computation
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:0aa79400a4f1c57ea3ab6ea6e66c248ea5afb163a717e993e19dbe595a0bb478
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
  - scImpute
  - splatter
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# clustering-consistency-metric-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate how well imputed or raw scRNA-seq expression matrices recover known cell identities by clustering cells and scoring agreement against ground-truth labels with adjusted Rand index, Jaccard index, normalized mutual information, and purity, repeated over random seeds.

## When to use

Use when an imputation method (e.g., Bfimpute, scImpute, MAGIC, SAVER, VIPER, PBLR, DrImpute) must be benchmarked on data with known cell type labels, such as splatter-simulated matrices or labeled 10x datasets, to quantify clustering recovery.

## When NOT to use

- No ground-truth cell labels or simulation-known identities exist, making ARI/Jaccard/NMI/purity against truth undefined — use unsupervised metrics instead
- The goal is differential expression or gene-level recovery accuracy rather than cell-to-cell relationship recovery, where the paper notes prior methods 'were less effective in recovering cell-to-cell relationships'
- Evaluating pseudotime/trajectory quality alone, where TSCAN POS and Kendall correlation are the appropriate scores rather than label-agreement metrics

## Inputs

- Imputed or raw gene-by-cell count matrix (e.g., Bfimpute output R_calculate, scImpute/DrImpute/MAGIC/SAVER/VIPER/PBLR outputs)
- Ground-truth cell type labels (simulated splatter labels or provided cell_type.csv)
- Number of cell types / clusters (e.g., 5-8 for splatter simulations, 7 for the Chu et al 2016 dataset)

## Outputs

- Table of mean +/- STD ARI, Jaccard index, NMI, and purity per method per cell-type count (comparable to Figure 2B)
- t-SNE embeddings per imputed matrix showing cluster separation and compactness
- For large-scale benchmarks: TSCAN POS and Kendall scores plus runtime table (e.g., vladoiu_imputation_benchmark_results.csv)

## How to apply

Cluster the expression matrix (rows = genes, columns = cells) using Bfimpute's spectral clustering (ccluster = c(7,"Spectrum")) or, when cell labels are known, supply them via ccluster="labeled" with the label parameter. Compute four agreement metrics — adjusted Rand index, Jaccard index, normalized mutual information, and purity — between predicted clusters and true simulated or provided labels, repeating over 10 random seeds and reporting mean +/- STD per method. Complement the scalar metrics with t-SNE embeddings to visually confirm cluster compactness and separation, as in Figure 2B. For very large labeled datasets (e.g., the 62,893-cell Vladoiu cerebellum set with 91.9% dropout), additionally score with TSCAN pseudotime POS and Kendall's rank correlation. All four clustering metrics lie in [0,1] by construction, providing built-in schema validation.

## Related tools

- **Bfimpute** (Imputes the count matrix (spectral clustering with c(7,"Spectrum"), D=32, totalepoch=300, burnin=200, threshold=0.5) whose clustered output is scored against truth labels) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparison imputer run with default settings on the same matrices before metric evaluation)
- **DrImpute** (Comparison imputer averaging imputation over multiple clustering results)
- **MAGIC** (Markov affinity-based graph imputer used as a baseline in the clustering benchmark)
- **SAVER** (Bayesian prior-based comparison imputer)
- **VIPER** (Non-negative sparse regression neighborhood imputer used as a baseline)
- **PBLR** (Low-rank alternating-direction matrix-completion baseline imputer)
- **splatter** (R package used to simulate scRNA-seq count matrices with 5, 6, 7, and 8 cell types (20,000 genes, 500-800 cells) in 'experiment' dropout mode, providing ground-truth labels)
- **Spectrum** (Spectral clustering engine invoked through Bfimpute's ccluster parameter to group cells before metric computation)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), D = 32, totalepoch = 300, burnin = 200, threshold = 0.5, ncores = 5); ari <- mclust::adjustedRandIndex(cluster, true_labels)
```

## Evaluation signals

- All four metrics (ARI, Jaccard, NMI, purity) fall within [0,1] for every method and cell-type count
- Reproduced table of mean +/- STD per method per cell-type count is directly comparable to Figure 2B, with Bfimpute attaining the best scores on 5-8 cell-type splatter simulations
- Results are stable when repeated over 10 random seeds (low STD)
- t-SNE embeddings of Bfimpute-imputed matrices show the most compact, well-separated clusters versus raw data and other imputers
- On the Vladoiu cerebellum benchmark, Bfimpute attains the best TSCAN POS/Kendall and clustering scores and the runtime ordering reproduces MAGIC < PBLR < Bfimpute < scImpute < SAVER

## Limitations

- The paper cautions that 'There is no consensus about which imputation method is the best' — performance varies across datasets, protocols, and downstream analyses
- 'it is premature to conclude that Bfimpute will achieve better performance across all datasets' — generalizability beyond benchmarked data is unconfirmed
- Imputation can increase false positives in downstream analysis; parameters must be tuned per application to balance true and false positives

## Evidence

- [results] On simulated datasets with 5-8 cell types, Bfimpute achieved the best clustering scores (adjusted Rand index, Jaccard index, normalized mutual information, purity): [evidence span withheld — anchor status: not_found]
- [methods] Simulated matrices use splatter with experiment dropout mode and specific Bfimpute defaults: [evidence span withheld — anchor status: not_found]
- [results] Metrics repeated over random seeds against true simulated labels: [evidence span withheld — anchor status: not_found]
- [results] Large-scale benchmark finding on Vladoiu dataset: [evidence span withheld — anchor status: not_found]
