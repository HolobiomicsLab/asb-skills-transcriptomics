---
name: pca-linear-baseline-ordering
description: 'Use when benchmarking a circular/periodic pseudo-time method against
  a linear baseline: scRNA-seq count matrices where cells should be ordered along
  a latent process and you need PCA-derived PC1/PC2 orderings for comparison (e.g.
  GSEA NES ranking as in Fig. 4a).'
when_to_use_negative:
- If the process of interest is genuinely nonlinear/periodic — PCA's linear representation
  cannot accurately represent circular timings, which is precisely why Cyclum's sinusoidal
  autoencoder was developed
- If you need cell-cycle scoring from marker genes rather than ab initio orderings
  — Cyclone-style phase prediction or Seurat CellCycleScoring is a different analysis
- If the input is not a cells-by-genes expression matrix suitable for the log2(x+1),
  centered (mean = 1), scaled (variance = 1) preprocessing
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: Provides the recommended preprocessing (scaled, centered log2(x+1) data) and
    the GSEA export helpers (mat2txt, vec2cls, mat2cls) used to feed PC orderings
    into GSEA; the circular method the PCA baseline is compared against
  repo: https://github.com/KChen-lab/Cyclum
- name: reCAT
  role: GMM-based cell-cycle pseudo-time method benchmarked alongside the PC1/PC2
    linear baseline
- name: GSEA
  role: Gene set enrichment tool used to score each ordering (including PC1/PC2) against
    the GO cell-cycle gene set and produce NES values
- name: scikit-learn
  role: Tested Cyclum dependency whose PCA implementation can supply the PC1/PC2 orderings
  repo: https://github.com/KChen-lab/Cyclum
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:35e81fcee8c207787b5d679a2f06c27c06331b2b383d57dd101ecbec13d51e0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - GSEA
  - reCAT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# PCA linear baseline ordering

## Summary

Computes PC1 and PC2 orderings of cells as linear-representation baselines for pseudo-time inference, so that circular methods such as Cyclum (and the GMM-based reCAT) can be benchmarked against linear approaches using GSEA cell-cycle enrichment scores.

## When to use

Use when benchmarking a circular/periodic pseudo-time method against a linear baseline: scRNA-seq count matrices where cells should be ordered along a latent process and you need PCA-derived PC1/PC2 orderings for comparison (e.g. GSEA NES ranking as in Fig. 4a).

## When NOT to use

- If the process of interest is genuinely nonlinear/periodic — PCA's linear representation cannot accurately represent circular timings, which is precisely why Cyclum's sinusoidal autoencoder was developed
- If you need cell-cycle scoring from marker genes rather than ab initio orderings — Cyclone-style phase prediction or Seurat CellCycleScoring is a different analysis
- If the input is not a cells-by-genes expression matrix suitable for the log2(x+1), centered (mean = 1), scaled (variance = 1) preprocessing

## Inputs

- Preprocessed scRNA-seq expression matrices (log2(x+1) transformed, centered to mean = 1, scaled to variance = 1) for samples Mel78, Mel79, Mel80, Mel81, Mel88 (10X Chromium counts)
- GO cell-cycle gene set for GSEA

## Outputs

- PC1 and PC2 cell orderings per sample (continuous phenotype vectors, e.g. GSEA-compatible .cls via mat2cls)
- GSEA normalized enrichment scores (NES) for PC1 and PC2 per sample, contributing to gsea_nes_ranking.csv (Fig. 4a / Supplementary Table 1)

## How to apply

Run PCA on the same preprocessed matrices used for the other methods — the melanoma 10X Chromium counts preprocessed with a log2(x+1) transform followed by centering (mean = 1) and scaling (variance = 1), the normalization Cyclum recommends. Extract the first two principal components and use the cells' coordinates along PC1 and PC2 to define two linear orderings, treated as candidate pseudo-times. For each sample (Mel78, Mel79, Mel80, Mel81, Mel88) and each ordering, run GSEA against the GO cell-cycle gene set and record the normalized enrichment score (NES). Concatenate the NES values of PC1 and PC2 with those of Cyclum and reCAT to build the per-sample ranking table (gsea_nes_ranking.csv, mirroring Fig. 4a / Supplementary Table 1). The rationale is that existing linear representations cannot accurately represent circular timings, so PC1/PC2 orderings should score lower on cell-cycle GSEA enrichment than the circular pseudo-time.

## Related tools

- **Cyclum** (Provides the recommended preprocessing (scaled, centered log2(x+1) data) and the GSEA export helpers (mat2txt, vec2cls, mat2cls) used to feed PC orderings into GSEA; the circular method the PCA baseline is compared against) — https://github.com/KChen-lab/Cyclum
- **reCAT** (GMM-based cell-cycle pseudo-time method benchmarked alongside the PC1/PC2 linear baseline)
- **GSEA** (Gene set enrichment tool used to score each ordering (including PC1/PC2) against the GO cell-cycle gene set and produce NES values)
- **scikit-learn** (Tested Cyclum dependency whose PCA implementation can supply the PC1/PC2 orderings) — https://github.com/KChen-lab/Cyclum

## Examples

```
from sklearn.decomposition import PCA; pcs = PCA(n_components=2).fit_transform(X_preprocessed); pc_orders = {'PC1': pcs[:, 0], 'PC2': pcs[:, 1]}
```

## Evaluation signals

- PC1 and PC2 orderings are computed on the identical preprocessed (log2(x+1), centered mean=1, scaled variance=1) matrices as Cyclum/reCAT, so NES values are directly comparable
- In the expected result, Cyclum's circular pseudo-time attains the highest GSEA cell-cycle NES on all five melanoma samples, with PC1 and PC2 ranking below Cyclum (Fig. 4a / Supplementary Table 1)
- NES values are computed per method per sample across Mel78, Mel79, Mel80, Mel81, Mel88, including the low-cycling sample Mel79 where the ranking must still hold
- Ranking table gsea_nes_ranking.csv contains NES entries for all four methods (Cyclum, reCAT, PC1, PC2) per sample with a consistent rank column

## Limitations

- Linear PCA orderings are baselines: existing pseudo-time methods based on linear representations 'are not optimal for representing such nonlinear periodicity', so PC1/PC2 are not expected to recover circular cell-cycle structure
- Comparison depends on GSEA of a single cell-cycle gene set; results are enrichment scores (NES) rather than per-cell phase labels
- Cyclum's tested software stack is specific (python 3.7.4, tensorflow 1.14.0, scikit-learn 0.21.3); newer versions should work but untested combinations may fail

## Evidence

- [intro] Existing trajectory/pseudo-time inference methods are not optimal for representing such nonlinear periodicity: "Existing trajectory/pseudo-time inference methods are not optimal for representing such nonlinear periodicity8."
- [methods] scaled centered log2(x+1) preprocessing: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum outperformed reCAT: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [readme] mat2cls writes multiple sets of phenotypes for GSEA: [evidence span withheld — anchor status: not_found]
