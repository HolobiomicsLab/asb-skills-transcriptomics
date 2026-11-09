---
name: silhouette-cluster-separation-evaluation
description: Use when comparing single-cell denoising or feature-selection methods
  and a ground-truth cell-phenotype labeling exists, so cluster separation can be
  quantified as mean silhouette score per method and latent dimension.
when_to_use_negative:
- No known phenotype or cluster labels exist for the cells — silhouette requires predefined
  groupings and cannot be computed from labels alone
- Cells have not yet been embedded into a reduced latent space (silhouette is computed
  on embedded coordinates, not the raw sparse count matrix)
- The goal is to discover novel clusters rather than evaluate recovery of known marked
  cell populations
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Randomly
  role: RMT-based denoising whose latent output is evaluated by silhouette score against
    baselines
  repo: https://github.com/luisaparicio/randomly
- name: MAGIC
  role: Imputation baseline whose denoised matrix is embedded and silhouette-scored
- name: scImpute
  role: Imputation baseline included in the silhouette comparison
- name: ZIFA
  role: Zero-inflated dimensionality-reduction baseline whose latent space is silhouette-scored
- name: ZIMB-WaVE
  role: Zero-inflated dimensionality-reduction baseline included in the comparison
- name: scVI
  role: Deep-learning baseline producing a latent embedding evaluated by silhouette
    score
- name: DCA
  role: Deep-learning baseline included in the silhouette comparison
- name: Seurat
  role: Baseline workflow whose reduced representation is silhouette-scored
- name: pip
  role: Installs the randomly package used to generate the RMT latent representation
- name: Python 3
  role: Runtime environment for computing silhouette scores (Randomly only works with
    Python 3)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:8ba859ad7d3b8048836875fc5c3334451302beb021127bce5efab5cf71f8554f
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
  - Seurat
  - ZIFA
  - ZIMB-WaVE
  - pip
  - scImpute
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# silhouette-cluster-separation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates whether a denoising or dimensionality-reduction method preserves known cell phenotypes by computing the mean silhouette coefficient of phenotype clusters in the reduced latent space, as used to benchmark RMT-based denoising against MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-HVG, and raw baselines.

## When to use

Use when comparing single-cell denoising or feature-selection methods and a ground-truth cell-phenotype labeling exists, so cluster separation can be quantified as mean silhouette score per method and latent dimension.

## When NOT to use

- No known phenotype or cluster labels exist for the cells — silhouette requires predefined groupings and cannot be computed from labels alone
- Cells have not yet been embedded into a reduced latent space (silhouette is computed on embedded coordinates, not the raw sparse count matrix)
- The goal is to discover novel clusters rather than evaluate recovery of known marked cell populations

## Inputs

- Reduced latent-space embedding (cell x dimension coordinate matrix) per denoising method and latent dimension
- Phenotype/cluster label vector aligned to cells (e.g., 13 PBMC phenotypes; 7, 15, or 26 mouse cortex groupings)
- Denoised expression matrices from each method: RMT (Randomly), MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI, DCA, Seurat, top-300 HVG, and raw

## Outputs

- Table of mean silhouette scores per method, per latent dimension, and per dataset/grouping (e.g., silhouette_scores.csv)
- Comparison verdict identifying the method with maximum mean silhouette per dataset and latent dimension

## How to apply

Project each denoised expression matrix (e.g., RMT-selected signal via Randomly, MAGIC-imputed, scVI/DCA latent embeddings) into a reduced latent space, sweeping over latent-space dimensions. For each embedding, compute the silhouette coefficient on the embedded cell coordinates using the known phenotype labels — 13 PBMC phenotypes for the 6,573-cell Kang et al. dataset, or 7/15/26-groupings for the 3,005-cell Zeisel et al. mouse cortex dataset. Average the per-cell silhouette values to obtain a mean silhouette score per method, per dataset/grouping, and per latent dimension. Tabulate all scores (e.g., silhouette_scores.csv) and identify the method attaining the maximum mean silhouette per dataset and latent dimension. The rationale is that silhouette score is label-aware and ranges over [-1, 1], giving a bounded, comparable measure of whether denoising improved resolution of known marked cell populations.

## Related tools

- **Randomly** (RMT-based denoising whose latent output is evaluated by silhouette score against baselines) — https://github.com/luisaparicio/randomly
- **MAGIC** (Imputation baseline whose denoised matrix is embedded and silhouette-scored)
- **scImpute** (Imputation baseline included in the silhouette comparison)
- **ZIFA** (Zero-inflated dimensionality-reduction baseline whose latent space is silhouette-scored)
- **ZIMB-WaVE** (Zero-inflated dimensionality-reduction baseline included in the comparison)
- **scVI** (Deep-learning baseline producing a latent embedding evaluated by silhouette score)
- **DCA** (Deep-learning baseline included in the silhouette comparison)
- **Seurat** (Baseline workflow whose reduced representation is silhouette-scored)
- **pip** (Installs the randomly package used to generate the RMT latent representation)
- **Python 3** (Runtime environment for computing silhouette scores (Randomly only works with Python 3))

## Examples

```
import randomly; model = randomly.KMeans(denoise=True); model.fit(X); from sklearn.metrics import silhouette_score; s = silhouette_score(embedding, labels)
```

## Evaluation signals

- silhouette_scores.csv exists and contains a mean silhouette score for every method x latent-dimension x dataset/grouping combination
- Every silhouette value lies in the valid range [-1, 1]
- RMT rows show the maximum mean silhouette score per dataset and latent dimension, as reported in Figure 6A-6D
- The advantage of RMT over baselines grows as the number of populations and latent-space dimensions increase

## Limitations

- Silhouette evaluation depends on the quality of the predefined phenotype labels; mislabeled or coarse groupings distort the comparison
- Results must be swept over latent-space dimensions, since method rankings can change with dimensionality
- No version or release information is provided for the publicly available algorithm code, complicating exact reproduction

## Evidence

- [results] RMT outperforms baselines in silhouette separation: [evidence span withheld — anchor status: not_found]
- [results] Compute mean silhouette in latent space: [evidence span withheld — anchor status: not_found]
- [results] Validation range: [evidence span withheld — anchor status: not_found]
- [readme] Randomly install via pip: [evidence span withheld — anchor status: not_found]
