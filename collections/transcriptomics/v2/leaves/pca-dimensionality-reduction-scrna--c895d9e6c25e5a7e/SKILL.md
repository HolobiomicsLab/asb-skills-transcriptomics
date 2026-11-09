---
name: pca-dimensionality-reduction-scrna
description: Use when a scRNA-seq preprocessing pipeline needs a compact PCA embedding
  (e.g., PC1-30) from scaled expression data before cell clustering, doublet simulation,
  or PC-count parameters for tools like DoubletFinder within Chord.
when_to_use_negative:
- Input is raw unnormalised UMI counts — PCA must be run on the normalised, variable-gene-scaled
  matrix, not raw counts
- The downstream tool requires full-dimensional data or its own embedding (e.g., scds's
  cxds/bcds operate on gene-level statistics, not a PCA embedding)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Chord
  role: Pipeline whose overkillDB2 training-set generation runs PCA (PC1-30) on scaled
    data before k-means clustering and cell-pair averaging
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Consumes a PC-based parameter (dbf.PCs, default 1:10) estimated from a PC
    elbow plot of the PCA embedding when generating artificial doublets within Chord
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: R
  role: Runtime environment (Seurat NormalizeData/FindVariableFeatures/ScaleData/PCA
    functions) executing the dimensionality reduction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:fdc2a475d1d5bbbc1a670882cbd1358de267d9ce7fc73123852cd64076e845be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# PCA dimensionality reduction for scRNA-seq

## Summary

Applies PCA to a scaled single-cell RNA-seq expression matrix to obtain a low-dimensional embedding (PC1-30 in the Chord workflow) that serves as the basis for downstream steps such as k-means clustering of cells for artificial doublet simulation, and as input to PC-based parameters like DoubletFinder's dbf.PCs.

## When to use

Use when a scRNA-seq preprocessing pipeline needs a compact PCA embedding (e.g., PC1-30) from scaled expression data before cell clustering, doublet simulation, or PC-count parameters for tools like DoubletFinder within Chord.

## When NOT to use

- Input is raw unnormalised UMI counts — PCA must be run on the normalised, variable-gene-scaled matrix, not raw counts
- The downstream tool requires full-dimensional data or its own embedding (e.g., scds's cxds/bcds operate on gene-level statistics, not a PCA embedding)

## Inputs

- Scaled gene-by-cell expression matrix (ScaleData output on a normalised Seurat object with highly variable genes identified)

## Outputs

- PCA embedding of cells (PC1-30 scores) used for k-means clustering with k=20 in the Chord artificial-doublet simulation step

## How to apply

In the Chord workflow (overkillDB2), after normalising the prefiltered Seurat object with NormalizeData and identifying variable genes with FindVariableFeatures, scale the expression matrix with ScaleData, then run PCA retaining components PC1-30 on the scaled data. This PCA embedding is then partitioned by k-means clustering (k=20) to sample cell pairs for generating artificial doublets labelled doubletrate/(1-doubletrate). Separately, when Chord invokes DoubletFinder, the number of statistically-significant principal components (dbf.PCs, default 1:10) should be estimated from a PC elbow plot and supplied as a parameter. Retaining ~30 PCs balances capturing dominant variance structure used for clustering against avoiding PC noise-driven cluster fragmentation.

## Related tools

- **Chord** (Pipeline whose overkillDB2 training-set generation runs PCA (PC1-30) on scaled data before k-means clustering and cell-pair averaging) — https://github.com/13308204545/Chord
- **DoubletFinder** (Consumes a PC-based parameter (dbf.PCs, default 1:10) estimated from a PC elbow plot of the PCA embedding when generating artificial doublets within Chord) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **R** (Runtime environment (Seurat NormalizeData/FindVariableFeatures/ScaleData/PCA functions) executing the dimensionality reduction)

## Examples

```
seu <- ScaleData(seu); seu <- RunPCA(seu, npcs = 30); overkillDB2(seu, sce, doubletrate, seed, k = 20, overkill = TRUE, overkillrate, overkilllist, adddoublt)
```

## Evaluation signals

- The PCA embedding is computed on the scaled matrix of the prefiltered Seurat object after NormalizeData and FindVariableFeatures, not on raw counts
- The retained components span PC1-30 as specified in the Chord training-set workflow, matching the k-means k=20 clustering input
- When DoubletFinder is invoked, dbf.PCs defaults to 1:10 and should correspond to statistically-significant PCs from a PC elbow plot
- Downstream check: cells partitioned by k-means on the PCA embedding yield artificial doublets at the expected proportion set by doubletrate (analogous to DoubletFinder pN, default 0.25 of merged data)

## Limitations

- PC retention (PC1-30) is fixed by the Chord workflow rather than adapted per dataset; an inappropriate PC count can distort the k-means clusters used for doublet simulation
- For DoubletFinder, pK should be adjusted per dataset (determined via mean-variance-normalized bimodality coefficient), implying PC-based defaults are not universally optimal

## Evidence

- [other] Reduce dimensionality with PCA retaining components PC1-30 on the scaled data. 3. Partition cells into clusters using k-means clustering with k=20 on the PCA embedding.: [evidence span withheld — anchor status: not_found]
- [readme] dbf.PCs Number of statistically-significant principal components (e.g., as estimated from PC elbow plot); Default: 1:10: "dbf.PCs Number of statistically-significant principal components (e.g., as estimated from PC elbow plot); Default: 1:10"
- [discussion] Overkill' step: evaluate data with built-in methods, remove filtered doublets, then simulate doublets from remaining cells to generate the training set: "First, built-in methods are used to evaluate the data. Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells"
- [other] ScaleData, identify highly variable genes with FindVariableFeatures, and scale the expression matrix with ScaleData: [evidence span withheld — anchor status: not_found]
