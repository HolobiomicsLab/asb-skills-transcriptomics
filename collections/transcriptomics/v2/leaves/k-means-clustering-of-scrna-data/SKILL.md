---
name: k-means-clustering-of-scrna-data
description: Use when simulating artificial doublets from quality singlets after an
  overkill/filter step, and cells must be grouped so sampled pairs come from transcriptionally
  similar cells to make realistic training doublets.
when_to_use_negative:
- The input is raw droplet data that has not yet passed Chord's overkill step — clustering
  should be run on the prefiltered/remaining quality singlet data, not before doublet
  filtering.
- The goal is biological cell-type annotation rather than simulating doublet training
  profiles — k=20 here is a simulation-granularity parameter, not a cell-type resolution
  choice.
- The data are not cell-level expression profiles (e.g. already a feature table or
  aggregated counts), since within-cluster cell-pair averaging of expression profiles
  is the core operation.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Chord
  role: Implements the k-means clustering step inside overkillDB2(seu, sce, doubletrate,
    seed, k, overkill, overkillrate, overkilllist, adddoublt) to build the simulated-doublet
    training set
  repo: https://github.com/13308204545/Chord
- name: R
  role: Runtime environment; Seurat PCA and k-means clustering are performed in R
- name: DoubletFinder
  role: Reference for the pN convention (default 0.25 of merged real-artificial data)
    used to set the artificial doublet proportion
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:d047f7625174b26e5e43963e91968f2dd39a2e3e4746173ea02e7cda5aa0b5dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# k-means-clustering-of-scrna-data

## Summary

Partition cells of a prefiltered Seurat object into k-means clusters (k=20) on a PC1-30 embedding so that artificial doublets can be simulated from within-cluster cell-pair averages when building Chord's GBM training set.

## When to use

Use when simulating artificial doublets from quality singlets after an overkill/filter step, and cells must be grouped so sampled pairs come from transcriptionally similar cells to make realistic training doublets.

## When NOT to use

- The input is raw droplet data that has not yet passed Chord's overkill step — clustering should be run on the prefiltered/remaining quality singlet data, not before doublet filtering.
- The goal is biological cell-type annotation rather than simulating doublet training profiles — k=20 here is a simulation-granularity parameter, not a cell-type resolution choice.
- The data are not cell-level expression profiles (e.g. already a feature table or aggregated counts), since within-cluster cell-pair averaging of expression profiles is the core operation.

## Inputs

- Prefiltered Seurat object (post-overkill) of quality singlets
- Normalized and scaled expression matrix (NormalizeData, FindVariableFeatures, ScaleData)
- PCA embedding retaining components PC1-30
- k-means parameter k (default k=20 in Chord's overkillDB2)
- Estimated doublet rate (doubletrate) and random seed

## Outputs

- Cell-to-cluster assignments on the PC1-30 embedding (k=20 clusters)
- Within-cluster cell-pair samples used as artificial doublet profiles
- Merged labelled real-artificial training set saved to overkilled.robj

## How to apply

Load the prefiltered Seurat object, run NormalizeData, FindVariableFeatures, and ScaleData, then reduce dimensionality with PCA retaining PC1-30. Partition cells with k-means using k=20 on the PCA embedding (k is exposed as the Chord/overkillDB2 parameter 'k', an integer k-means param). Within each cluster, sample cell pairs at a ratio of doubletrate/(1-doubletrate) relative to the real cells, weighting each member's expression profile with an independent N(1,0.1) biological weighting factor. Average the weighted profiles of each sampled pair to create artificial doublet profiles, label them as doublets, and append them to the remaining real cells to form the merged real-artificial training set saved as overkilled.robj. The rationale is that clustering before pair sampling keeps simulated doublets within biologically plausible cell states, analogous to the DoubletFinder pN convention (default 0.25 of merged data).

## Related tools

- **Chord** (Implements the k-means clustering step inside overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt) to build the simulated-doublet training set) — https://github.com/13308204545/Chord
- **R** (Runtime environment; Seurat PCA and k-means clustering are performed in R)
- **DoubletFinder** (Reference for the pN convention (default 0.25 of merged real-artificial data) used to set the artificial doublet proportion) — https://github.com/chris-mcginnis-ucsf/DoubletFinder

## Examples

```
chord(seu=seu, sce=sce, doubletrate=0.05, k=20, overkill=T, outname="run1")
```

## Evaluation signals

- The merged real-artificial object contains artificial doublets at the expected proportion set by doubletrate (analogous to the DoubletFinder pN convention, default 0.25 of the merged data).
- Clustering is performed on PC1-30 of the scaled data with k=20, matching the workflow's stated parameters.
- Each artificial doublet profile is the average of two real cells' expression profiles, each weighted by an independent N(1,0.1) factor.
- The merged labelled training set is written to overkilled.robj via overkillDB2.
- Doublet barcodes from the resulting training set can be exported to CSV (outname_doublets.csv).

## Limitations

- Chord is not optimal in runtime because it integrates results of multiple methods, so the clustering/simulation step adds overhead.
- The k-means parameter k and doubletrate must be chosen per dataset; doubletrate depends on cell number (10X: ~0.9% per 1,000 cells).
- Simulated doublets are homotypic-heavy by construction (pairs sampled within clusters), while heterotypic doublets are the ones with most impact on downstream analyses.

## Evidence

- [methods] Partition cells into clusters using k-means clustering with k=20 on the PCA embedding.: [evidence span withheld — anchor status: not_found]
- [methods] Within each cluster, sample cell pairs at a ratio of doubletrate/(1-doubletrate) relative to the real cells: [evidence span withheld — anchor status: not_found]
- [methods] Reduce dimensionality with PCA retaining components PC1-30 on the scaled data.: [evidence span withheld — anchor status: not_found]
- [readme] k an integer,k-means param k: "k an integer,k-means param k"
