---
name: single-cell-rna-seq-normalization-and-scaling
description: Use when a scRNA-seq Seurat object has already been through a coarse
  doublet-removal (overkill) step and you need normalized, scaled, PCA-embedded data
  to build k-means clusters for artificial doublet simulation.
when_to_use_negative:
- Input data has not yet been through a coarse doublet-filtering (overkill) step,
  so simulated doublets would be trained on likely doublets
- Input is not droplet-based scRNA-seq count data in a Seurat object, e.g. already
  a feature table or a fully analysed expression matrix
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: Implements the normalization, scaling, PCA, and k-means clustering pipeline
    (via overkillDB2) that prepares the data for artificial doublet simulation
  repo: https://github.com/13308204545/Chord
- name: Seurat
  role: R framework providing NormalizeData, FindVariableFeatures, ScaleData, and
    PCA used to preprocess the prefiltered Seurat object
- name: DoubletFinder
  role: Installed dependency whose pN convention (default 0.25) informs the expected
    proportion of artificial doublets in the merged training set
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Dependency used within Chord's overkill/built-in methods applied before normalization-based
    training-set generation
  repo: https://github.com/kostkalab/scds
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:87eb68ddbdd7dc5b015a2aebff2bbff5a74d1856be0249eaebd9ebdf0991f090
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - Seurat
  - scds
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rna-seq-normalization-and-scaling

## Summary

Preprocess a prefiltered Seurat object (NormalizeData, FindVariableFeatures, ScaleData, PCA on PC1-30) so its cell embeddings can be partitioned by k-means (k=20) to simulate artificial doublets for Chord's GBM ensemble training set.

## When to use

Use when a scRNA-seq Seurat object has already been through a coarse doublet-removal (overkill) step and you need normalized, scaled, PCA-embedded data to build k-means clusters for artificial doublet simulation.

## When NOT to use

- Input data has not yet been through a coarse doublet-filtering (overkill) step, so simulated doublets would be trained on likely doublets
- Input is not droplet-based scRNA-seq count data in a Seurat object, e.g. already a feature table or a fully analysed expression matrix

## Inputs

- Prefiltered Seurat object (remaining quality singlet cells after the overkill step)
- SingleCellExperiment object derived via creatSCE(seu)

## Outputs

- Normalized, scaled, PCA-reduced (PC1-30) Seurat object partitioned into k-means clusters (k=20)
- Merged real-artificial labelled training set saved as overkilled.robj

## How to apply

Load the prefiltered Seurat object and normalise it with NormalizeData, then identify highly variable genes with FindVariableFeatures and scale the expression matrix with ScaleData. Reduce dimensionality with PCA retaining components PC1-30 on the scaled data. Partition the cells into clusters using k-means clustering with k=20 on the PCA embedding, implemented inside overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt). The rationale is that these normalized and PCA-reduced embeddings define the cell clusters from which cell pairs are sampled (at ratio doubletrate/(1-doubletrate)) to generate simulated doublet profiles used to fit Chord's GBM ensemble model.

## Related tools

- **Chord** (Implements the normalization, scaling, PCA, and k-means clustering pipeline (via overkillDB2) that prepares the data for artificial doublet simulation) — https://github.com/13308204545/Chord
- **Seurat** (R framework providing NormalizeData, FindVariableFeatures, ScaleData, and PCA used to preprocess the prefiltered Seurat object)
- **DoubletFinder** (Installed dependency whose pN convention (default 0.25) informs the expected proportion of artificial doublets in the merged training set) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Dependency used within Chord's overkill/built-in methods applied before normalization-based training-set generation) — https://github.com/kostkalab/scds

## Examples

```
chord(seu="input seurat object", doubletrate="estimated doubletrate", overkill=T, outname="the name you want")
```

## Evaluation signals

- Confirm the merged real-artificial object (overkilled.robj) contains artificial doublets at the expected proportion set by doubletrate (analogous to DoubletFinder pN, default 0.25 of merged data)
- Verify PCA was run retaining PC1-30 components on the scaled data
- Verify k-means clustering used k=20 clusters on the PCA embedding
- Confirm doublet barcodes can be exported from the output to CSV (outname_doublets.csv)

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods, and this preprocessing step is part of that multi-method pipeline
- The estimated doubletrate parameter directly controls how many artificial doublets are sampled per cluster; a wrong estimate skews the training set
- Other pre-processing steps, such as ambient mRNA removal for droplets, are needed to improve downstream accuracy but are not incorporated in Chord

## Evidence

- [other] Partition cells into clusters using k-means clustering with k=20 on the PCA embedding.: [evidence span withheld — anchor status: not_found]
- [other] PC1-30 retained on scaled data: [evidence span withheld — anchor status: not_found]
- [methods] overkillDB2 function signature: "overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt)"
- [discussion] Overkill step workflow description: "First, built-in methods are used to evaluate the data. Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [readme] DoubletFinder pN default in Chord README: "The number of generated artificial doublets, expressed as a proportion of the merged real-artificial data. Default is set to 0.25, based on observation that DoubletFinder performance is largely"
