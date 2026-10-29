---
name: adjusted-rand-index-evaluation
description: Use when comparing corrected scRNAseq matrices from multiple confounder-removal
  methods (scPLS, RUV, LMM, PCA, scLVM) and you need a single clustering-accuracy
  number per method against known labels.
when_to_use_negative:
- You need to evaluate downstream analyses other than clustering (e.g. differential
  expression, lineage reconstruction) — the paper states evaluating other downstream
  analyses is beyond its scope
- No ground-truth or known labels exist for the cells, so no ARI against a reference
  partition can be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: scPLS
  role: Provides the corrected target-gene matrix whose clustering ARI is evaluated,
    via scPLS(Target, Control, ...)
  repo: http://chenmengjie.github.io/Citrus/
- name: SC3
  role: One of the four downstream clustering approaches whose SC3-based clustering
    is scored by ARI
- name: t-SNE
  role: t-SNE-embedded clustering is one of the four clustering approaches scored
    by ARI
- name: Citrus
  role: Package supplying scPLS, the CellCycleData dataset used for labels/controls,
    and simulation utilities for the ARI benchmark
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:43508e0e15eb0b73292f081f4f30c71712243da8c272a46c96b61e4e18993d5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - SC3
  - genlasso
  - scPLS
  - t-SNE
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# adjusted-rand-index-evaluation

## Summary

Evaluate confounding-effect-removal methods by clustering corrected single-cell RNA-seq data and scoring agreement with known cell labels using the Adjusted Rand Index (ARI, mclust implementation), as done in the scPLS paper across kmeans, PCA, t-SNE and SC3 clustering approaches.

## When to use

Use when comparing corrected scRNAseq matrices from multiple confounder-removal methods (scPLS, RUV, LMM, PCA, scLVM) and you need a single clustering-accuracy number per method against known labels.

## When NOT to use

- You need to evaluate downstream analyses other than clustering (e.g. differential expression, lineage reconstruction) — the paper states evaluating other downstream analyses is beyond its scope
- No ground-truth or known labels exist for the cells, so no ARI against a reference partition can be computed

## Inputs

- Confounder-corrected expression matrices from each method (scPLS, RUV, LMM, PCA, scLVM) plus the uncorrected matrix
- Ground-truth cell labels (e.g. known cell-cycle-stage labels for the 132 homogeneous mESC control cells, or true simulated cell labels)
- Choice of clustering approaches: k-means, PCA-based, t-SNE-embedded, SC3

## Outputs

- ARI value per (method, clustering-approach) pair computed with the mclust ARI implementation
- Methods-by-clustering-approach comparison table written to ari_table.csv
- For simulation studies: ARI across PDE levels (20–100%) and dropout conditions, plus PVE of estimated confounding factors in simulation_evaluation_results.csv

## How to apply

Cluster each confounder-corrected expression matrix (e.g. scPLS output vs RUV/LMM/PCA/scLVM outputs and the uncorrected data) using multiple downstream approaches — k-means, PCA-based clustering, t-SNE-embedded clustering, and SC3 — so method rankings are not an artifact of one clustering choice. Compute the Adjusted Rand Index of each clustering against ground-truth labels (in the mESC application, the known cell-cycle-stage labels of the 132 homogeneous control cells) using the mclust ARI implementation. Assemble the ARI values into a methods-by-clustering-approach comparison table (e.g. ari_table.csv) and judge a correction method successful when its ARI approaches or exceeds that of the uncorrected baseline (scPLS reached up to 100 vs 59 uncorrected, winning on kmeans, PCA and SC3 but not tSNE). Sweep simulation parameters such as proportion of differential expression (PDE 20–100%) and dropout condition when evaluating on simulated data.

## Related tools

- **scPLS** (Provides the corrected target-gene matrix whose clustering ARI is evaluated, via scPLS(Target, Control, ...)) — http://chenmengjie.github.io/Citrus/
- **SC3** (One of the four downstream clustering approaches whose SC3-based clustering is scored by ARI)
- **t-SNE** (t-SNE-embedded clustering is one of the four clustering approaches scored by ARI)
- **Citrus** (Package supplying scPLS, the CellCycleData dataset used for labels/controls, and simulation utilities for the ARI benchmark)

## Examples

```
library(mclust); ari <- adjustedRandIndex(clusters, labels); write.csv(ari_table, "ari_table.csv")
```

## Evaluation signals

- ARI values are non-negative or near 0 for random agreement and should be reported per (method, clustering approach) pair, not as a single number
- Corrected-data ARI should be compared against the uncorrected baseline (mESC: scPLS ~100 vs 59 uncorrected under kmeans)
- Method ranking must be reported across all four clustering approaches; scPLS loses under tSNE, so a single-approach table signals incomplete evaluation
- Simulation results table should cover the full PDE sweep 20–100% for both scenarios I and II, with and without dropout, and include PVE of estimated confounding factors
- Output file ari_table.csv (or simulation_evaluation_results.csv) exists and is a methods-by-clustering-approach matrix

## Limitations

- The 'best' confounding-effect-removal method may vary depending on the downstream analysis or question of interest; scPLS performance in other analysis settings has not been evaluated
- A method's ARI advantage can be clustering-dependent — scPLS outperformed RUV, LMM, PCA and scLVM on kmeans, PCA and SC3 but lost under tSNE on the mESC control cells
- ARI evaluation depends on known labels; scPLS can only infer biological effects, not biological factors, so labels must come from external knowledge

## Evidence

- [results] scPLS outperforms RUV, LMM, PCA and scLVM in three of the four clustering approaches (kmeans, PCA, SC3), with ARI values up to 100 (vs 59 uncorrected), losing only under tSNE.: [evidence span withheld — anchor status: not_found]
- [discussion] The 'best' confounding effects removing method may vary depending on the downstream analysis or question of interest; performance of scPLS in other analysis settings has not been evaluated: "the “best” confounding effects removing method may vary depending on the question of interest. Therefore, it would be important to evaluate the performance of scPLS in other analysis settings in"
- [other] Compute the adjusted Rand Index of each clustering against the known cell-cycle-stage labels using the mclust ARI implementation.: [evidence span withheld — anchor status: not_found]
- [abstract] With extensive simulations and comparisons with other methods, we demonstrate the effectiveness of scPLS.: "With extensive simulations and comparisons with other methods, we demonstrate the effectiveness of scPLS."
