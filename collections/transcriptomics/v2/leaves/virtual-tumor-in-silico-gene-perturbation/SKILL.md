---
name: virtual-tumor-in-silico-gene-perturbation
description: Use when benchmarking cell-cycle removal methods on scRNA-seq data where
  no true mixed-subclone tumor sample exists, by simulating perturbed gene expression
  in mESC cells and measuring subclone delineation accuracy.
when_to_use_negative:
- Input data already contains genuine mixed tumor subclones — use real subclone delineation
  rather than in-silico perturbation
- The perturbation effect of interest is not gene-set expression amplification; doubling
  (×2) expression only models upregulation-style perturbations
- The goal is cell-cycle phase scoring/pseudotime inference itself rather than evaluating
  removal methods via downstream subclone separability
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: Cyclum
  role: Removes cell-cycle effects from virtual tumor data by projecting expression
    to a circular manifold via a sinusoidal autoencoder
  repo: github.com/KChen-lab/cyclum
- name: Seurat
  role: Linear regression baseline for cell-cycle effect removal on the virtual tumor
    samples
- name: ccRemover
  role: Linear regression baseline for cell-cycle effect removal on the virtual tumor
    samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:31d51c742e6d45f2f810f1760ee975adc2592661662b5d7a6234f356735297c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - Seurat
  - ccRemover
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Virtual tumor in-silico gene perturbation for benchmarking cell-cycle correction

## Summary

Constructs virtual tumor samples from the mESC scRNA-seq dataset by doubling (×2) expression of perturbed gene sets (including >400 cell-cycle genes) across mESC cells, then benchmarking how well Cyclum vs. Seurat, ccRemover, and PC1–5 linear corrections restore subclone separability quantified by a two-component Gaussian mixture model.

## When to use

Use when benchmarking cell-cycle removal methods on scRNA-seq data where no true mixed-subclone tumor sample exists, by simulating perturbed gene expression in mESC cells and measuring subclone delineation accuracy.

## When NOT to use

- Input data already contains genuine mixed tumor subclones — use real subclone delineation rather than in-silico perturbation
- The perturbation effect of interest is not gene-set expression amplification; doubling (×2) expression only models upregulation-style perturbations
- The goal is cell-cycle phase scoring/pseudotime inference itself rather than evaluating removal methods via downstream subclone separability

## Inputs

- mESC scRNA-seq expression matrix (count data amenable to log2(x+1) transformation)
- Perturbed gene set definitions, including the >400-gene cell-cycle gene set
- Subclone membership labels for the two virtual subclones

## Outputs

- Per-method corrected expression matrices (Cyclum circular-manifold, Seurat, ccRemover, PC1–5, uncorrected)
- Two-component GMM subclone separability metrics per method and perturbation condition
- Comparison table saved as subclone_separability.csv

## How to apply

Load the mESC scRNA-seq dataset and preprocess with log2(x+1) transformation, then center to mean = 1 and scale to variance = 1, as recommended by the paper's methods. Generate virtual tumor samples by doubling (×2) the expression of perturbed gene sets — including the cell-cycle set of more than 400 genes — across mESC cells. Remove cell-cycle effects with Cyclum by projecting the data to a circular manifold using the sinusoidal autoencoder, and with the linear regression approaches implemented in Seurat and ccRemover, plus a PC1–5 correction baseline. For each corrected dataset, fit a two-component Gaussian mixture model (GMM) on the leading components and quantify separability of the two subclones. Aggregate GMM separability metrics across methods and save as subclone_separability.csv. The rationale is that linear representations cannot accurately represent circular cell-cycle timings, so the circular approach should yield improved subclone delineation over linear baselines under most perturbation conditions.

## Related tools

- **Cyclum** (Removes cell-cycle effects from virtual tumor data by projecting expression to a circular manifold via a sinusoidal autoencoder) — github.com/KChen-lab/cyclum
- **Seurat** (Linear regression baseline for cell-cycle effect removal on the virtual tumor samples)
- **ccRemover** (Linear regression baseline for cell-cycle effect removal on the virtual tumor samples)

## Examples

```
pip install . && jupyter notebook tests/notebooks/mESC.ipynb  # Cyclum circular-manifold correction on mESC virtual tumor data
```

## Evaluation signals

- Two-component GMM separability ranking across methods reproduces the published Fig. 3c / Source Data pattern, with the circular approach outperforming linear baselines
- Cyclum shows significantly higher subclone-separability accuracy than Seurat, ccRemover, and PC1–5 under almost all conditions, especially when more than 400 cell-cycle genes are perturbed
- Preprocessing invariant: data are scaled to variance = 1, centered to mean = 1 after log2(x+1) transformation before perturbation and correction
- Output table subclone_separability.csv contains separability metrics for all methods (Cyclum, Seurat, ccRemover, PC1–5, uncorrected) across the perturbation parameter grid

## Limitations

- Virtual tumor samples are simulated by ×2 gene-expression doubling, so results may not transfer to perturbation types other than expression amplification
- Cyclum requires TensorFlow 1.x; the README notes pip will install latest dependencies that differ from tested versions (tested: python 3.7.4, keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5)
- Linear baselines (Seurat, ccRemover, PC1–5) rely on linear representations that cannot accurately represent circular periodic timings, limiting fair comparison at extreme nonlinearity

## Evidence

- [methods] Preprocessing uses scaled, centered log-transformed data: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] Cyclum removes cell-cycle effects via circular manifold projection: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [abstract] Cyclum improves subpopulation delineation: "Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations"
- [intro] Linear approaches are limited for circular processes: "The cell cycle regression approaches implemented in scLVM17, Seurat18, and ccRemover19 are based on linear representations"
- [readme] Cyclum install and dependency guidance: [evidence span withheld — anchor status: external_doc]
