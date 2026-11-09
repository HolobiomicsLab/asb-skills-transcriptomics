---
name: cell-cycle-effect-removal-method-benchmarking
description: Use when scRNA-seq data contain cell-cycle effects confounding subclone
  or subpopulation delineation, and the goal is to quantify which correction approach
  (circular vs linear) maximizes subclone separability.
when_to_use_negative:
- When the goal is predicting discrete cell-cycle phases (G1/S/G2M labels) rather
  than removing cell-cycle effects — marker-gene scoring (e.g., Seurat::CellCycleScoring)
  is the appropriate tool instead
- When the periodic process of interest is the analysis target (pseudo-time inference)
  rather than a confounder to be removed
- When the input data are not scRNA-seq expression matrices — the linear regression
  baselines and circular-manifold projection are defined on gene-by-cell expression
  data
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Cyclum
  role: 'Nonlinear correction baseline: sinusoidal autoencoder projecting data onto
    a circular manifold to remove cell-cycle effects; the method expected to win the
    separability benchmark'
  repo: https://github.com/KChen-lab/cyclum
- name: Seurat
  role: Linear regression baseline for cell-cycle effect removal used in the method
    comparison
- name: ccRemover
  role: Linear regression baseline for cell-cycle effect removal used in the method
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:59c01ed14bf82accd08c628256965ead4397e74cd0283160876e7a4f0043578c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/cyclum
  tools:
  - Cyclum
  - Seurat
  - ccRemover
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-cycle-effect-removal-method-benchmarking

## Summary

A benchmarking workflow comparing nonlinear circular cell-cycle correction (Cyclum, a sinusoidal autoencoder) against linear regression approaches (Seurat, ccRemover, PC1–5) for how well each restores subclone delineation in scRNA-seq data with perturbed cell-cycle effects.

## When to use

Use when scRNA-seq data contain cell-cycle effects confounding subclone or subpopulation delineation, and the goal is to quantify which correction approach (circular vs linear) maximizes subclone separability.

## When NOT to use

- When the goal is predicting discrete cell-cycle phases (G1/S/G2M labels) rather than removing cell-cycle effects — marker-gene scoring (e.g., Seurat::CellCycleScoring) is the appropriate tool instead
- When the periodic process of interest is the analysis target (pseudo-time inference) rather than a confounder to be removed
- When the input data are not scRNA-seq expression matrices — the linear regression baselines and circular-manifold projection are defined on gene-by-cell expression data

## Inputs

- mESC scRNA-seq gene expression count matrix
- Perturbation parameter specification: gene sets to double (×2), including the >400-gene cell-cycle gene set
- Cell-cycle gene set annotations used to define perturbed genes

## Outputs

- subclone_separability.csv — aggregated two-component GMM separability metrics per method (Cyclum, Seurat, ccRemover, PC1–5, uncorrected) and condition
- Cell-cycle-corrected expression matrices from each method (circular manifold projection and linear regression variants)
- Per-condition GMM subclone-separability accuracy values

## How to apply

Load the mESC scRNA-seq dataset and preprocess with a log2(x+1) transformation, centering to mean = 1 and scaling to variance = 1, which the paper reports works well. Generate virtual tumor samples by doubling (×2) the expression of perturbed gene sets — including the >400-gene cell-cycle set — across mESC cells to create two subclones with controlled perturbation parameters. Remove cell-cycle effects with each method: Cyclum by projecting data onto a circular manifold with its sinusoidal autoencoder, and the linear baselines via regression in Seurat and ccRemover, plus a PC1–5 correction as an additional linear control. For each corrected dataset, fit a two-component Gaussian mixture model (GMM) on the leading components and quantify two-subclone separability. Aggregate the GMM separability metrics across all methods and conditions into subclone_separability.csv. Correct application reproduces the published Fig. 3c pattern: Cyclum achieves significantly higher separability than the linear baselines, particularly when more than 400 cell-cycle genes are perturbed.

## Related tools

- **Cyclum** (Nonlinear correction baseline: sinusoidal autoencoder projecting data onto a circular manifold to remove cell-cycle effects; the method expected to win the separability benchmark) — https://github.com/KChen-lab/cyclum
- **Seurat** (Linear regression baseline for cell-cycle effect removal used in the method comparison)
- **ccRemover** (Linear regression baseline for cell-cycle effect removal used in the method comparison)

## Examples

```
conda create -n cyclum python=3.7 pip && conda activate cyclum && git clone https://github.com/KChen-lab/Cyclum.git && cd Cyclum && pip install .
```

## Evaluation signals

- Preprocessing invariant: the input matrix is log2(x+1) transformed, centered to mean = 1, and scaled to variance = 1 before correction
- Each corrected dataset yields a converged two-component GMM fit on its leading components with a finite subclone separability score
- Ranking check: Cyclum's separability accuracy exceeds Seurat, ccRemover, and PC1–5 under almost all perturbation conditions, most strongly when >400 cell-cycle genes are perturbed
- Output artifact subclone_separability.csv exists and reproduces the published Fig. 3c / Source Data ranking pattern
- Uncorrected (no-correction) control is included as a baseline row in the comparison table

## Limitations

- The linear regression baselines (Seurat, ccRemover, PC1–5) rely on linear representations and cannot represent the circular manifold, which is why they lose separability when many cell-cycle genes (>400) are perturbed
- Cyclum is model-based and fits data to predefined circular manifolds — appropriate for circular/periodic processes but an assumption that may not hold for other confounders
- Runtime and dependencies: the Cyclum package was tested with Python 3.7, Keras 2.2.4, and TensorFlow 1.14.0 (TensorFlow 1.x required); 12,280 cells × 33,694 genes took ~40 min on a GTX 960M GPU with 2 GB graphic memory
- No version numbers for Python/Keras/TensorFlow are stated in the paper's discussion section; only the README dependency table supplies tested versions

## Evidence

- [abstract] Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations: "Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations"
- [intro] The cell cycle regression approaches implemented in scLVM17, Seurat18, and ccRemover19 are based on linear representations: "The cell cycle regression approaches implemented in scLVM17, Seurat18, and ccRemover19 are based on linear representations"
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed log2(x+1) data work well: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes.: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
- [readme] It provides methods to recover cell cycle information and remove cell cycle factor from the scRNA-seq data.: [evidence span withheld — anchor status: external_doc]
