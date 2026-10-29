---
name: single-cell-rna-seq-preprocessing
description: Use when an expression matrix (scRNA-seq counts or qPCR values) must
  be normalized before fitting Cyclum or before running baseline methods (PCA, Slalom,
  Cyclone, reCAT, Oscope, Cyclops) on periodic-process data.
when_to_use_negative:
- Input is already log-transformed, centered, and scaled (e.g. loaded from a preprocessed
  .h5 matrix); re-applying the transform would distort the circular pattern
- The analysis target is linear trajectories rather than circular/periodic processes
  such as the cell cycle, where Cyclum's circular manifold assumption does not apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cyclum
  role: Target method whose sinusoidal autoencoder consumes the preprocessed matrix;
    preprocessing is the recommended transform in the Cyclum documentation
  repo: github.com/KChen-lab/cyclum
- name: PCA
  role: Linear baseline (PC1/PC2) run on the same preprocessed data for accuracy and
    GSEA NES comparison
- name: f-scLVM (Slalom)
  role: Baseline fit with known gene sets on the identically preprocessed matrix
- name: Cyclone
  role: Baseline (PCA plus relative expression of gene pairs) run on the same preprocessed
    data
- name: reCAT
  role: Model-free baseline (GMM pseudo-time) applied to the identically preprocessed
    matrix
- name: Oscope
  role: Baseline detecting pairwise circular gene patterns on the shared preprocessed
    data
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:960084aed2db3e21fd22a6cee6870fc166c17e95d26da9a5dd1e30af654438bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellOracle
  - Cyclone
  - Cyclum
  - MSigDB
  - Oscope
  - PCA
  - R
  - R 4.1.3
  - TSP
  - VEGA
  - VGAM
  - arrow
  - cluster
  - doParallel
  - doSNOW
  - f-scLVM
  - f-scLVM (Slalom)
  - foreach
  - mclust
  - reCAT
  - scGATE
  - scanpy
  - truncnorm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-rna-seq-preprocessing

## Summary

Preprocess scRNA-seq or qPCR expression matrices for Cyclum circular-manifold analysis using the log2(x+1) transform followed by centering (mean = 1) and scaling (variance = 1), which preserves the circular pattern of periodic processes such as the cell cycle.

## When to use

Use when an expression matrix (scRNA-seq counts or qPCR values) must be normalized before fitting Cyclum or before running baseline methods (PCA, Slalom, Cyclone, reCAT, Oscope, Cyclops) on periodic-process data.

## When NOT to use

- Input is already log-transformed, centered, and scaled (e.g. loaded from a preprocessed .h5 matrix); re-applying the transform would distort the circular pattern
- The analysis target is linear trajectories rather than circular/periodic processes such as the cell cycle, where Cyclum's circular manifold assumption does not apply

## Inputs

- raw scRNA-seq expression matrix (genes x cells) or single-cell qPCR dataset
- Hoechst-labeled datasets such as mESC, PC3, MDA-MB-231, H9 for benchmarking

## Outputs

- log2(x+1)-transformed, centered (mean = 1), scaled (variance = 1) expression matrix ready for Cyclum or baseline fitting
- downstream derived tables such as classification_accuracy_comparison.csv or gsea_nes_results.csv built from the shared preprocessed matrix

## How to apply

Apply a log2(x+1) transformation to the raw expression matrix, then center each feature to mean = 1 and scale to variance = 1, exactly as advised in the Cyclum documentation so the circular pattern of gene expression is preserved. Feed the transformed matrix into the Cyclum sinusoidal autoencoder (optionally using cyclum.tuning.CyclumAutoTune to pick the number of linear dimensions) to obtain circular pseudo-time, or into linear baselines such as PCA (PC1/PC2), NMF, and ICA for comparison. Use the same preprocessed matrix for every method being compared so accuracy or GSEA NES differences reflect the inference method, not the normalization. The rationale is that Cyclum projects high-dimensional gene expression onto a circular manifold rather than relying on marker genes, and this transform keeps the data on a scale compatible with that manifold.

## Related tools

- **Cyclum** (Target method whose sinusoidal autoencoder consumes the preprocessed matrix; preprocessing is the recommended transform in the Cyclum documentation) — github.com/KChen-lab/cyclum
- **PCA** (Linear baseline (PC1/PC2) run on the same preprocessed data for accuracy and GSEA NES comparison)
- **f-scLVM (Slalom)** (Baseline fit with known gene sets on the identically preprocessed matrix)
- **Cyclone** (Baseline (PCA plus relative expression of gene pairs) run on the same preprocessed data)
- **reCAT** (Model-free baseline (GMM pseudo-time) applied to the identically preprocessed matrix)
- **Oscope** (Baseline detecting pairwise circular gene patterns on the shared preprocessed data)

## Examples

```
git clone https://github.com/KChen-lab/Cyclum.git; cd Cyclum; pip install .
```

## Evaluation signals

- Verify the preprocessed matrix has per-feature variance = 1 and mean = 1 after log2(x+1) transformation
- When comparing methods, confirm every method (Cyclum, Slalom, Cyclone, reCAT, Oscope, Cyclops, PCA, NMF, ICA) received the identical preprocessed matrix
- Accuracy table values must lie in [0, 1] for all nine methods on all four datasets, with Cyclum exceeding every baseline on mESC, PC3, MDA-MB-231, and H9
- For GSEA on mESC, Cyclum-based gene ranking should yield NES = 1.57 for GO:0007049, above PC1 (1.07) and PC2 (1.06)
- CyclumAutoTune should select the number of linear dimensions that produces the 'most circular' manifold before pseudo-time inference

## Limitations

- Cyclum was tested with specific pinned versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0); pip installs latest dependencies which may differ, and TensorFlow < 1.15.2 has a known security issue
- Runtime/hardware are informal (12,280 cells x 33,694 genes in 40 min on a GTX 960M laptop); detailed benchmarking is only in Supplementary Note 2
- The paper does not state Python/Keras/TensorFlow version numbers in the article body, so reproducibility depends on the README's dependency table

## Evidence

- [methods] Recommended transform preserves circular pattern: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] Cyclum projects onto circular manifold without marker genes: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [readme] CyclumAutoTune selects linear dimensionality: [evidence span withheld — anchor status: external_doc]
- [abstract] Cyclum autoencoder identifies circular trajectories ab initio: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [readme] Tested dependency versions for the pipeline: [evidence span withheld — anchor status: external_doc]
