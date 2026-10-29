---
name: latent-dimensionality-sweep-analysis
description: Use when you have denoised single-cell expression matrices plus known
  phenotype labels and need to compare methods by cluster resolution, sweeping latent-space
  dimensions and reporting mean silhouette scores per method and dimension.
when_to_use_negative:
- The dataset has no known phenotype labels or marked cell populations — the mean
  silhouette score of known clusters cannot be defined
- You only need gene/feature selection (RMT sparsity filter) rather than a cross-method,
  cross-dimension cluster-resolution benchmark
- A single fixed latent dimensionality is already mandated by the downstream analysis,
  so the dimension sweep adds no decision value
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Randomly
  role: 'RMT-based denoising of the single-cell matrix: eigenvalue/eigenvector-based
    feature selection and low-rank signal extraction before the latent projection'
  repo: https://rabadan.c2b2.columbia.edu/html/randomly/
- name: pip
  role: Installation route for the Randomly library
- name: Python 3
  role: Required runtime — Randomly only works with Python 3
- name: MAGIC
  role: Imputation baseline denoising method to benchmark against RMT
- name: scImpute
  role: Imputation baseline denoising method to benchmark against RMT
- name: ZIFA
  role: Zero-inflated dimensionality-reduction baseline to benchmark against RMT
- name: ZIMB-WaVE
  role: Zero-inflated dimensionality-reduction baseline to benchmark against RMT
- name: scVI
  role: Non-linear neural-network baseline to benchmark against RMT
- name: DCA
  role: Non-linear neural-network baseline to benchmark against RMT
- name: Seurat
  role: Baseline processing pipeline to benchmark against RMT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:477633ed3ecbe6c960232b79d304c5318fe2d81fe92f03346a22cd2d3f49e122
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
  - Seurat
  - ZIFA
  - ZIMB-WaVE
  - pip
  - scImpute
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# latent-dimensionality-sweep-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that projects denoised single-cell matrices (RMT and baselines such as MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw) into latent spaces of increasing dimension and computes mean silhouette scores of known cell phenotypes to quantify which method best resolves marked cell populations.

## When to use

Use when you have denoised single-cell expression matrices plus known phenotype labels and need to compare methods by cluster resolution, sweeping latent-space dimensions and reporting mean silhouette scores per method and dimension.

## When NOT to use

- The dataset has no known phenotype labels or marked cell populations — the mean silhouette score of known clusters cannot be defined
- You only need gene/feature selection (RMT sparsity filter) rather than a cross-method, cross-dimension cluster-resolution benchmark
- A single fixed latent dimensionality is already mandated by the downstream analysis, so the dimension sweep adds no decision value

## Inputs

- Sparse single-cell cell-gene expression matrices with phenotype labels (e.g., 6,573-cell human PBMCs from Kang et al.; 3,005-cell mouse cortex cells from Zeisel et al.)
- Denoised versions of each matrix produced by Randomly (RMT, selecting ~1,500 signal-like genes) and by baselines: MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, and the raw unfiltered matrix
- Sweep specification of latent-space dimensions and phenotype groupings (13 PBMC phenotypes; 7, 15, and 26 mouse cortex groupings)

## Outputs

- silhouette_scores.csv with mean silhouette score per method, per latent dimension, and per dataset/grouping
- Comparison table/plot reproducing the Figure 6A-6D method-vs-dimension silhouette curves
- Per-dimension latent embeddings of the denoised matrices used for silhouette computation

## How to apply

Load a marked dataset (e.g., the 6,573-cell human PBMC matrix from Kang et al. or the 3,005-cell mouse cortex matrix from Zeisel et al.) together with its phenotype labels. Denoise the matrix with Randomly (installed with `pip install randomly`, Python 3 only), which suppresses sparsity-induced localized eigenvectors and extracts a low-rank signal via eigenvalue/eigenvector departures from RMT universality (TW/MP distributions); denoise parallel copies with each baseline (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, and the raw matrix). Project each denoised result into a reduced latent space while sweeping over the number of latent dimensions, then compute the mean silhouette coefficient of phenotype clusters (13 PBMC phenotypes; 7, 15, and 26 mouse cortex groupings) on the embedded cell coordinates at each dimension. Tabulate mean silhouette per method, per latent dimension, and per dataset/grouping into silhouette_scores.csv; the expected outcome is that the RMT-selected 1,500 most signal-like genes attain the highest silhouette, with the advantage growing as the number of populations and latent dimensions increase, reproducing the Figure 6A-6D comparison.

## Related tools

- **Randomly** (RMT-based denoising of the single-cell matrix: eigenvalue/eigenvector-based feature selection and low-rank signal extraction before the latent projection) — https://rabadan.c2b2.columbia.edu/html/randomly/
- **pip** (Installation route for the Randomly library)
- **Python 3** (Required runtime — Randomly only works with Python 3)
- **MAGIC** (Imputation baseline denoising method to benchmark against RMT)
- **scImpute** (Imputation baseline denoising method to benchmark against RMT)
- **ZIFA** (Zero-inflated dimensionality-reduction baseline to benchmark against RMT)
- **ZIMB-WaVE** (Zero-inflated dimensionality-reduction baseline to benchmark against RMT)
- **scVI** (Non-linear neural-network baseline to benchmark against RMT)
- **DCA** (Non-linear neural-network baseline to benchmark against RMT)
- **Seurat** (Baseline processing pipeline to benchmark against RMT)

## Examples

```
pip install randomly && python -c 'import randomly'
```

## Evaluation signals

- silhouette_scores.csv exists and every silhouette value falls in the valid range [-1, 1]
- For each dataset and latent dimension, the RMT row attains the maximum mean silhouette score among all methods, as reported in Figure 6A-6D
- All compared methods are present in the table (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, raw, RMT)
- Sweep coverage includes the 13 PBMC phenotypes and the 7, 15, and 26 mouse cortex phenotype groupings
- The RMT advantage over baselines grows with the number of populations and the number of latent-space dimensions

## Limitations

- Randomly only works with Python 3 (not Python 2)
- No version or release information is provided for the publicly available algorithm code
- The RMT advantage grows as the number of populations and latent-space dimensions increase, so results may be weaker for few populations or few dimensions

## Evidence
