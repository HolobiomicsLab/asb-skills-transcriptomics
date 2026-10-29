---
name: rmt-eigenvector-based-feature-selection
description: Use when a highly sparse single-cell expression matrix (~95% sparse,
  e.g. 6,573 PBMC or 3,005 mouse cortex cells) must be denoised and signal-like genes
  selected before clustering or latent-space embedding.
when_to_use_negative:
- The input is not a highly sparse single-cell expression matrix — the RMT noise model
  assumes the ~95%-sparse regime where universality holds and sparsity dominates the
  spectrum.
- Genes are already pre-selected to a small curated panel, leaving too few features
  for eigenvalue/eigenvector statistics to be meaningful.
- The task requires supervised phenotype prediction with labels rather than unsupervised
  feature selection and denoising.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Python 3 library implementing the RMT-based eigenvalue/eigenvector feature
    selection and low-rank signal extraction
  repo: github.com/RabadanLab/randomly
- name: pip
  role: Installs the Randomly package from PyPI
  repo: github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime required by Randomly (Python 2 is not supported)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:46a3a2beb49d8d4ded2de703759fa3356fd3b4b293118414808b4d3a34ea6e65
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rmt-eigenvector-based-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A feature selection method that denoises sparse single-cell expression matrices using random matrix theory (RMT): genes responsible for eigenvector localization in a randomized (sparsity-only) version of the data are discarded, leaving a low-rank biological signal. It is the core of the Randomly library from Aparicio et al., Patterns 2020.

## When to use

Use when a highly sparse single-cell expression matrix (~95% sparse, e.g. 6,573 PBMC or 3,005 mouse cortex cells) must be denoised and signal-like genes selected before clustering or latent-space embedding.

## When NOT to use

- The input is not a highly sparse single-cell expression matrix — the RMT noise model assumes the ~95%-sparse regime where universality holds and sparsity dominates the spectrum.
- Genes are already pre-selected to a small curated panel, leaving too few features for eigenvalue/eigenvector statistics to be meaningful.
- The task requires supervised phenotype prediction with labels rather than unsupervised feature selection and denoising.

## Inputs

- Sparse cell-gene expression matrix (e.g. 6,573 human PBMCs from Kang et al. or 3,005 mouse cortex cells from Zeisel et al.)
- Phenotype/cell-population labels for downstream validation (13 PBMC phenotypes; 7, 15, or 26 mouse cortex groupings)
- Randomized (label-permuted per gene) version of the expression matrix

## Outputs

- Set of signal-like genes (e.g. top 1,500) surviving the sparsity/eigenvector-localization filter
- Low-rank denoised representation of the single-cell data
- Per-gene statistics on eigenvalue deviations from RMT universality and eigenvector localization
- Reduced latent-space embedding used for mean silhouette scoring

## How to apply

Model the cell-gene covariance matrix as a random matrix (noise) plus a low-rank perturbation (signal). First randomize the dataset by permuting cell labels independently for each gene so only sparsity effects remain, then identify genes responsible for localized eigenvectors in the randomized matrix and remove them — this suppresses the ~3% spurious sparsity-induced signal, since only the last ~2% of the information is true biological signal while ~95% matches RMT universality (TW/MP distributions). On the filtered matrix, detect departures from RMT universality in the eigenvalue spectrum and eigenvector localization statistics to flag biological signal, and extract a low-rank signal matrix. Select the most signal-like genes (e.g., the top 1,500 genes used in the silhouette benchmark against MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG and raw matrices). Project the denoised result into a latent space and verify cluster resolution with the mean silhouette coefficient of known phenotype labels.

## Related tools

- **Randomly** (Python 3 library implementing the RMT-based eigenvalue/eigenvector feature selection and low-rank signal extraction) — github.com/RabadanLab/randomly
- **pip** (Installs the Randomly package from PyPI) — github.com/luisaparicio/randomly
- **Python 3** (Runtime required by Randomly (Python 2 is not supported))

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- Every computed silhouette value lies in the valid range [-1, 1] (per the silhouette_scores.csv validation).
- RMT-selected genes (e.g. top 1,500) attain the maximum mean silhouette score per dataset and latent dimension compared against MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, and raw baselines (Figure 6A-6D).
- After the sparsity filter, localized eigenvectors present in the randomized matrix are no longer driven by the retained genes.
- The retained signal corresponds to deviations from TW/MP universality distributions in the covariance spectrum, consistent with a low-rank perturbation.
- Randomly runs only under Python 3 (not Python 2); verify the environment before executing.

## Limitations

- Roughly 95% of the information in single-cell data matches RMT noise predictions and ~3% is spurious sparsity-induced signal, so only a small fraction (~2%) of the data is retained as biological signal.
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data; this motivates the randomized-data filter but means unfiltered interpretations are unreliable.
- No version or release information is provided for the publicly available algorithm code.
- Predictive quantitative models of gene expression are lacking, so the correct expression distribution and i.i.d. assumptions are unclear.

## Evidence
