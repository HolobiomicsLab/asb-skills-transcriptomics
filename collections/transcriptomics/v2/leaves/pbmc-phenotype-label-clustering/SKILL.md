---
name: pbmc-phenotype-label-clustering
description: Use when you have a denoised or imputed single-cell expression matrix
  with known phenotype labels and need a quantitative cluster-separation metric (mean
  silhouette score in [-1, 1]) to rank denoising methods.
when_to_use_negative:
- The input is already a latent embedding with no underlying expression matrix and
  no phenotype labels, so silhouette scores against known populations cannot be computed
- You only need the RMT feature-selection step itself (discarding genes causing localized
  eigenvectors in the randomized data) rather than a benchmark comparison of denoising
  methods
- The dataset lacks known marked cell populations, so mean silhouette against ground-truth
  labels would be uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: 'RMT-based denoising library: eigenvalue/eigenvector-based feature selection
    selecting the 1,500 most signal-like genes and extracting a low-rank signal representation'
  repo: https://github.com/luisaparicio/randomly
- name: pip
  role: Installs the randomly package (Python 3 only)
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime for Randomly and for computing silhouette scores in the latent space
- name: MAGIC
  role: Imputation baseline for the silhouette comparison
- name: scImpute
  role: Imputation baseline for the silhouette comparison
- name: ZIFA
  role: Zero-inflated dimensionality-reduction baseline
- name: ZIMB-WaVE
  role: Zero-inflated dimensionality-reduction baseline
- name: scVI
  role: Non-linear neural-network baseline
- name: DCA
  role: Non-linear neural-network baseline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:1da842531e87d5f7243f35bcd39b3d0be13f2fd7b606958e9b890b521e0eedef
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  repo_url: https://github.com/luisaparicio/randomly
  tools:
  - DCA
  - MAGIC
  - Python 3
  - Randomly
  - ZIFA
  - ZIMB-WaVE
  - pip
  - scImpute
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pbmc-phenotype-label-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates denoised single-cell expression matrices by projecting them into a reduced latent space and computing the mean silhouette score of known cell-phenotype labels (13 PBMC phenotypes from Kang et al.; 7/15/26 mouse cortex groupings from Zeisel et al.), so that RMT-based denoising can be compared against MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, and raw baselines.

## When to use

Use when you have a denoised or imputed single-cell expression matrix with known phenotype labels and need a quantitative cluster-separation metric (mean silhouette score in [-1, 1]) to rank denoising methods.

## When NOT to use

- The input is already a latent embedding with no underlying expression matrix and no phenotype labels, so silhouette scores against known populations cannot be computed
- You only need the RMT feature-selection step itself (discarding genes causing localized eigenvectors in the randomized data) rather than a benchmark comparison of denoising methods
- The dataset lacks known marked cell populations, so mean silhouette against ground-truth labels would be uninformative

## Inputs

- 6,573-cell human PBMC expression matrix (Kang et al.), ~95% sparse, with 13 phenotype labels
- 3,005-cell mouse cortex expression matrix (Zeisel et al.) with 7, 15, and 26 phenotype groupings
- Denoised/imputed expression matrices from Randomly (RMT, 1,500 signal-like genes), MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, and the raw unfiltered matrix
- Sweep of latent-space dimensions for the reduced embedding

## Outputs

- silhouette_scores.csv with mean silhouette score per method, per latent dimension, and per dataset/phenotype grouping
- Figure 6A-6D-style comparison table showing RMT attaining the maximum silhouette score per dataset and latent dimension

## How to apply

Load the 6,573-cell human PBMC matrix (Kang et al.) or the 3,005-cell mouse cortex matrix (Zeisel et al.) together with phenotype labels, and denoise each with the Randomly package (RMT eigenvalue/eigenvector-based feature selection producing a 1,500-gene signal-like subset) plus each baseline (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw). Project every denoised result into a reduced latent space, sweeping over latent-space dimensions. Compute the mean silhouette coefficient on the embedded cell coordinates using the phenotype labels (13 PBMC phenotypes; 7, 15, and 26 mouse cortex groupings). Tabulate mean silhouette score per method, per latent dimension and per dataset/grouping, and write the results to silhouette_scores.csv. The decision point is the Figure 6A-6D comparison: RMT should attain the maximum mean silhouette score per dataset and latent dimension, with the advantage growing as the number of populations and latent dimensions increase.

## Related tools

- **Randomly** (RMT-based denoising library: eigenvalue/eigenvector-based feature selection selecting the 1,500 most signal-like genes and extracting a low-rank signal representation) — https://github.com/luisaparicio/randomly
- **pip** (Installs the randomly package (Python 3 only)) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime for Randomly and for computing silhouette scores in the latent space)
- **MAGIC** (Imputation baseline for the silhouette comparison)
- **scImpute** (Imputation baseline for the silhouette comparison)
- **ZIFA** (Zero-inflated dimensionality-reduction baseline)
- **ZIMB-WaVE** (Zero-inflated dimensionality-reduction baseline)
- **scVI** (Non-linear neural-network baseline)
- **DCA** (Non-linear neural-network baseline)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- silhouette_scores.csv exists and every silhouette value lies in the valid range [-1, 1]
- The RMT rows show the maximum mean silhouette score per dataset and latent dimension, as reported in Figure 6A-6D
- RMT's silhouette advantage grows as the number of populations and latent-space dimensions increase
- All 10 methods (RMT, MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw) are represented for each dataset and dimension sweep point

## Limitations

- Randomly only works with Python 3 (not Python 2)
- No version or release information is provided for the publicly available algorithm code
- Lack of complete quantitative understanding of the different sorts of noise that arise in single-cell measurements; the correct distribution and i.i.d. assumption are unclear
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data, which the sparsity filter must remove before scoring

## Evidence

- [results] RMT outperforms baselines on silhouette: [evidence span withheld — anchor status: not_found]
