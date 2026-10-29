---
name: melanoma-axl-mitf-state-marker-analysis
description: Use when testing drug-resistance program mutual exclusivity in melanoma
  scRNA-seq, especially in proliferating samples where cell-cycle signal masks the
  AXL–MITF anticorrelation (e.g., reproducing Mel78 R = -0.02 → -0.43 shift).
when_to_use_negative:
- The sample is quiescent rather than actively proliferating — cell-cycle correction
  is not needed to unmask the AXL–MITF anticorrelation.
- The goal is to infer circular pseudo-time or cell-cycle stage itself, rather than
  to use Cyclum as a confounder-removal step before program-score correlation.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Autoencoder that projects gene expression onto a circular manifold to remove
    cell-cycle effects before AXL/MITF program-score correlation
  repo: https://github.com/KChen-lab/Cyclum
- name: ccRemover
  role: Linear cell-cycle regression used for the Supplementary Fig. 7 comparison
    of corrected vs uncorrected correlations
- name: Seurat
  role: Linear cell-cycle regression used for the Supplementary Fig. 7 comparison
    of corrected vs uncorrected correlations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:defea3ed19f815127fd0c82d2bdf289e721b7d1b99a3a986a45ce5035e0f8924
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Seurat
  - ccRemover
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# melanoma-axl-mitf-state-marker-analysis

## Summary

Computes AXL-high and MITF-high melanoma cell-state program scores (mean expression of published gene sets) on raw and cell-cycle-corrected scRNA-seq matrices, then tests whether Cyclum correction unmasks the expected mutual exclusivity (negative AXL–MITF correlation) in actively proliferating melanoma samples like Mel78.

## When to use

Use when testing drug-resistance program mutual exclusivity in melanoma scRNA-seq, especially in proliferating samples where cell-cycle signal masks the AXL–MITF anticorrelation (e.g., reproducing Mel78 R = -0.02 → -0.43 shift).

## When NOT to use

- The sample is quiescent rather than actively proliferating — cell-cycle correction is not needed to unmask the AXL–MITF anticorrelation.
- The goal is to infer circular pseudo-time or cell-cycle stage itself, rather than to use Cyclum as a confounder-removal step before program-score correlation.

## Inputs

- Preprocessed Mel78 scRNA-seq expression matrix (scaled variance = 1, centered, log2(x+1) transformed)
- Published AXL and MITF gene sets (program signatures)
- Cell-cycle-corrected expression matrices (Cyclum, ccRemover, Seurat variants)

## Outputs

- Per-cell AXL and MITF program scores (mean expression of each gene set per condition)
- mel78_axl_mitf_correlations.csv containing per-condition Pearson R and P values
- Confirmed correlation shifts: uncorrected R = -0.02 (P = 0.81); Cyclum-corrected R = -0.43 (P = 9e-7)

## How to apply

Load the preprocessed Mel78 expression matrix using the paper's recommended transform: scaled (variance = 1), centered, log2(x+1) data. Compute per-cell AXL and MITF program scores as the average expression of the published AXL and MITF gene sets on the uncorrected matrix, and confirm the uncorrected Pearson correlation matches R = -0.02 (P = 0.81). Then run Cyclum — an autoencoder that projects gene expression to a circular manifold rather than relying on marker genes — to remove cell-cycle effects, and repeat program-score computation on the corrected matrix; also compute ccRemover- and Seurat-corrected versions for the Supplementary Fig. 7 comparison. Finally compute Pearson correlation between AXL and MITF scores per condition, expecting the Cyclum-corrected correlation to reach R = -0.43 (P = 9e-7), consistent with mutual exclusivity of MITF-high and AXL-high resistance states.

## Related tools

- **Cyclum** (Autoencoder that projects gene expression onto a circular manifold to remove cell-cycle effects before AXL/MITF program-score correlation) — https://github.com/KChen-lab/Cyclum
- **ccRemover** (Linear cell-cycle regression used for the Supplementary Fig. 7 comparison of corrected vs uncorrected correlations)
- **Seurat** (Linear cell-cycle regression used for the Supplementary Fig. 7 comparison of corrected vs uncorrected correlations)

## Examples

```
R, P = scipy.stats.pearsonr(mel78_cyclum_corrected['AXL_score'], mel78_cyclum_corrected['MITF_score'])  # expect R=-0.43, P=9e-7
```

## Evaluation signals

- Uncorrected Mel78 AXL–MITF Pearson correlation reproduces R = -0.02 (P = 0.81) within numerical tolerance.
- Cyclum-corrected Mel78 correlation reproduces R = -0.43 (P = 9e-7) within numerical tolerance, matching Supplementary Fig. 7.
- Per-condition correlations and p-values are all written to mel78_axl_mitf_correlations.csv with one row per correction method (none, Cyclum, ccRemover, Seurat).
- Program scores are computed as mean expression of the published gene sets on the same transform (scaled, centered, log2(x+1)) for corrected and uncorrected matrices alike.
- Corrected correlation is clearly negative (R ≈ -0.43), consistent with the expected MITF-high/AXL-high mutual exclusivity.

## Limitations

- No version numbers for Python, Keras, or TensorFlow are stated in the paper; the README notes pip installs latest dependencies differing from tested versions (e.g., keras 2.2.4, tensorflow 1.14.0), and TensorFlow < 1.15.2 has a known security issue.
- Cyclum is designed for periodic processes fitting circular manifolds — it is a model-based approach whose benefit depends on the cell cycle actually being the confounding periodic process.
- Hardware/runtime only given informally (12,280 cells × 33,694 genes in 40 min on a laptop with GTX 960M GPU, 2 GB graphic memory), with details deferred to Supplementary Note 2.

## Evidence

- [methods] Preprocessing with the paper-recommended transform: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] Removing cell-cycle effects before downstream subpopulation analysis: "It provides methods to recover cell cycle information and remove cell cycle effects from the scRNA-seq data."
- [abstract] Cyclum improves subpopulation delineation, motivating correction before state-marker correlation: "Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations"
- [readme] Cyclum relies on circular manifolds, not marker genes, when correcting the matrix: [evidence span withheld — anchor status: external_doc]
