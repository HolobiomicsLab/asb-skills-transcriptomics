---
name: cell-clustering-ari-evaluation
description: Use when you have simulated or labeled scRNA-seq data with known cell-type
  labels and need to quantify how well a normalization/confounding-removal method
  (e.g., scPLS) preserves cluster structure via ARI.
when_to_use_negative:
- The data have no known ground-truth cell-type labels, so no external clustering
  validation (ARI) is possible
- The downstream question is not clustering (the paper notes the best confounding-removal
  method may vary by analysis; scPLS performance was evaluated for clustering, not
  differential expression or lineage reconstruction)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
tools:
- name: Citrus
  role: R/C++ toolkit providing the exported scPLS() function used to estimate confounding
    factors before clustering and ARI scoring
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Normalization method that removes unwanted variation using both control and
    target genes, producing the corrected data evaluated by ARI
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required dependency for the C++-backed Citrus installation
- name: RcppArmadillo
  role: Required dependency enabling the block-wise EM inference backend of Citrus/scPLS
- name: genlasso
  role: R package dependency of Citrus
- name: gplots
  role: R package dependency of Citrus used in the distributed version
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:3f3328dac8c62a1b3c62bb4a4b926c0a81ade39d842d2f909224999ce093bd49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - genlasso
  - gplots
  - scPLS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cell-clustering-ari-evaluation

## Summary

Evaluates clustering accuracy of confounding-corrected single-cell RNA-seq data by comparing inferred cell clusters against true labels using adjusted Rand index (ARI), as done in the scPLS scenario I sensitivity analysis across reduced control-gene set sizes.

## When to use

Use when you have simulated or labeled scRNA-seq data with known cell-type labels and need to quantify how well a normalization/confounding-removal method (e.g., scPLS) preserves cluster structure via ARI.

## When NOT to use

- The data have no known ground-truth cell-type labels, so no external clustering validation (ARI) is possible
- The downstream question is not clustering (the paper notes the best confounding-removal method may vary by analysis; scPLS performance was evaluated for clustering, not differential expression or lineage reconstruction)

## Inputs

- Confounding-corrected single-cell expression matrix (e.g., scenario I simulated data after scPLS correction)
- Vector of true cell-type labels for each cell
- Control-gene set size q values to compare (e.g., 10, 20, 30, 40, 50)

## Outputs

- ARI value per control-set size q, tabulated as ari_vs_control_set_size.csv
- ARI curve figure across q values (ari_curve_control_set_reduction.png)

## How to apply

Fit the confounding-removal method (e.g., scPLS from the Citrus package, backed by Rcpp/RcppArmadillo block-wise EM) on target and control gene sets to estimate confounding factors, then cluster the corrected expression data. Score the resulting cluster assignments against the true cell-type labels using adjusted Rand index (ARI). For sensitivity analyses, vary the control-gene subset size q (e.g., 10, 20, 30, 40 versus the full 50) and tabulate ARI versus q. The reproduction is judged correct when ARI values across reduced q remain close to the q = 50 baseline, matching the paper's scenario I trend of high ARI maintained down to small control-set sizes.

## Related tools

- **Citrus** (R/C++ toolkit providing the exported scPLS() function used to estimate confounding factors before clustering and ARI scoring) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Normalization method that removes unwanted variation using both control and target genes, producing the corrected data evaluated by ARI) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required dependency for the C++-backed Citrus installation)
- **RcppArmadillo** (Required dependency enabling the block-wise EM inference backend of Citrus/scPLS)
- **genlasso** (R package dependency of Citrus)
- **gplots** (R package dependency of Citrus used in the distributed version)

## Examples

```
Rscript -e 'library(mclust); ari <- adjustedRandIndex(pred_clusters, true_labels); write.csv(data.frame(q=q_vals, ARI=ari_vals), "ari_vs_control_set_size.csv")'
```

## Evaluation signals

- CSV (ari_vs_control_set_size.csv) contains an ARI row for every q value in {10, 20, 30, 40, 50}
- ARI at each reduced q remains close to the q = 50 baseline, matching the paper's scenario I trend of high ARI at small control-set sizes
- Figure file ari_curve_control_set_reduction.png exists and plots ARI versus q
- Random subsampling of q control genes is drawn from the full 50-gene control set, preserving comparability across q
- Clustering of corrected data is scored against true labels only after scPLS confounding-factor estimation and correction

## Limitations

- The paper evaluates scPLS performance for clustering; performance in other downstream analyses (differential expression, lineage reconstruction) was not evaluated
- The best confounding-effects-removal method may vary depending on the downstream question of interest
- scPLS infers biological effects but not the biological factors themselves due to model identifiability

## Evidence

- [other] Cluster the confounding-corrected expression data at each q and score clustering accuracy against the true labels using adjusted Rand index (ARI).: [evidence span withheld — anchor status: not_found]
- [intro] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors."
- [discussion] the “best” confounding effects removing method may vary depending on the question of interest. Therefore, it would be important to evaluate the performance of scPLS in other analysis settings: "the “best” confounding effects removing method may vary depending on the question of interest."
- [readme] Citrus is written in c++ with easy-to-use R wrappers. To use Citrus, the installation of Rcpp and RcppArmadillo is required.: "Citrus is written in c++ with easy-to-use R wrappers. To use Citrus, the installation of Rcpp and RcppArmadillo is required."
- [readme] Citrus relies on the following R packages: Rcpp, RcppArmadillo, genlasso, gplots.: "Citrus relies on the following R packages: Rcpp, RcppArmadillo, genlasso, gplots."
