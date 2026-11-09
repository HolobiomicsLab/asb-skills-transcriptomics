---
name: doubletfinder-artificial-nearest-neighbor-scoring
description: Use when a droplet-based scRNA-seq Seurat/SingleCellExperiment object
  needs DoubletFinder pANN doublet scores — either to prefilter likely doublets before
  Chord simulates its artificial-doublet training set, or as one of three predictors
  (with cxds and bcds) for the GBM ensemble.
when_to_use_negative:
- Input is not droplet-based scRNA-seq in a Seurat/SingleCellExperiment object, or
  the goal is experimental doublet labeling across samples (cell hashing, MULTI-seq,
  demuxlet are used instead)
- A single gold-standard doublet method is already known to work well for the dataset
  — Chord's rationale is that individual methods vary greatly in performance and no
  method is a gold standard for each scenario
- Runtime is the binding constraint — Chord integrates multiple methods' scores (including
  DoubletFinder's), so it is not optimal in terms of time efficiency
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DoubletFinder
  role: Provides the pANN artificial-nearest-neighbor doublet scores used for overkill
    prefiltering and as GBM predictors
  repo: chris-mcginnis-ucsf/DoubletFinder
- name: Chord
  role: Orchestrating ensemble that calls DoubletFinder via the overkillDB2/scds wrappers
    and integrates pANN scores with gbm
  repo: 13308204545/Chord
- name: gbm
  role: Fits the gradient-boosting classifier on predictor score matrices that include
    DoubletFinder pANN scores
- name: scds
  role: Complementary built-in method (cxds, bcds) whose scores are integrated alongside
    DoubletFinder's
  repo: kostkalab/scds
- name: R
  role: Runtime environment; DoubletFinder installed via remotes::install_github
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:63d6b5f8a8a2ca249f6d4d02eb684a60e1a6c9f2f9a9021b3807b968acc3cad4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chris-mcginnis-ucsf/DoubletFinder
  tools:
  - Chord
  - DoubletFinder
  - R
  - creatSCE
  - gbm
  - overkillDB2
  - scds
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# doubletfinder-artificial-nearest-neighbor-scoring

## Summary

Run DoubletFinder's pANN-based artificial-nearest-neighbor scoring to assign per-cell doublet scores in the Chord ensemble, both on the merged real-artificial data during the overkill prefilter and as a predictor feature for the GBM training model.

## When to use

Use when a droplet-based scRNA-seq Seurat/SingleCellExperiment object needs DoubletFinder pANN doublet scores — either to prefilter likely doublets before Chord simulates its artificial-doublet training set, or as one of three predictors (with cxds and bcds) for the GBM ensemble.

## When NOT to use

- Input is not droplet-based scRNA-seq in a Seurat/SingleCellExperiment object, or the goal is experimental doublet labeling across samples (cell hashing, MULTI-seq, demuxlet are used instead)
- A single gold-standard doublet method is already known to work well for the dataset — Chord's rationale is that individual methods vary greatly in performance and no method is a gold standard for each scenario
- Runtime is the binding constraint — Chord integrates multiple methods' scores (including DoubletFinder's), so it is not optimal in terms of time efficiency

## Inputs

- Seurat object (e.g. seu.robj) of droplet-based scRNA-seq counts
- SingleCellExperiment object converted via Chord's creatSCE(seu) when scds is run alongside
- overkilled.robj (prefiltered real-artificial merged data) for training-stage scoring
- dbf.PCs (default 1:10), dbf.pN (0.25), dataset-tuned dbf.pK

## Outputs

- Per-cell pANN doublet scores for all cells of the input object
- Top-scoring barcodes removed during overkill (contributing to <outname>_doublets.csv)
- Score table contributing to real_data.scores.csv / simulated_data.scores.csv for GBM predictor assembly

## How to apply

Convert the input Seurat object to a SingleCellExperiment where scds compatibility is needed (creatSCE), then run DoubletFinder over dbf.PCs principal components (default 1:10). Fix the artificial-doublet proportion at dbf.pN=0.25 — justified by DoubletFinder's observed pN-invariance (McGinnis, Murrow and Gartner 2019) — and tune dbf.pK per dataset via the mean-variance-normalized bimodality coefficient, since no default pK is set. In Chord's no-ground-truth overkill mode, rank cells by pANN score and remove the top doubletrate × overkillrate fraction of most-likely doublets (10X guideline: doubletrate ≈ 0.9% per 1,000 cells). For ensemble training, score both the original data (seu.robj) and the simulated-doublet training data (overkilled.robj) so pANN scores can serve as GBM predictors (1000 trees, shrinkage 0.01, cv.folds 5), with simulated doublets labeled positive and singlets negative.

## Related tools

- **DoubletFinder** (Provides the pANN artificial-nearest-neighbor doublet scores used for overkill prefiltering and as GBM predictors) — chris-mcginnis-ucsf/DoubletFinder
- **Chord** (Orchestrating ensemble that calls DoubletFinder via the overkillDB2/scds wrappers and integrates pANN scores with gbm) — 13308204545/Chord
- **gbm** (Fits the gradient-boosting classifier on predictor score matrices that include DoubletFinder pANN scores)
- **scds** (Complementary built-in method (cxds, bcds) whose scores are integrated alongside DoubletFinder's) — kostkalab/scds
- **R** (Runtime environment; DoubletFinder installed via remotes::install_github)

## Examples

```
chord(seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want")
```

## Evaluation signals

- Fraction of cells removed per method during overkill equals doubletrate × overkillrate
- real_data.scores.csv exists and contains exactly one doublet score per cell of the input Seurat object
- Removed-barcodes CSV (<outname>_doublets.csv) exists with the expected barcode column
- dbf.pK was tuned per dataset (no default pK is set; optimal pK determined via mean-variance-normalized bimodality coefficient)
- overkilled.robj loads as a Seurat object containing only the retained cells after prefiltering

## Limitations

- Chord is not optimal in time efficiency because it integrates results of multiple methods; refactoring the integrated methods is proposed but not implemented
- dbf.pK has no default and must be adjusted for each scRNA-seq dataset
- Performance of individual doublet methods varies greatly across datasets, which is the motivation for ensembling rather than relying on DoubletFinder alone
- Other pre-processing such as ambient mRNA removal is not incorporated and may still be needed for downstream accuracy

## Evidence

- [methods] dbf.pN fixed at pN-invariant default: "dbf.pN  The number of generated artificial doublets, expressed as a proportion of the merged real-artificial data. Default is set to 0.25, based on observation that DoubletFinder performance is"
- [readme] dbf.pK must be tuned per dataset: "dbf.pK  The PC neighborhood size used to compute pANN, expressed as a proportion of the merged real-artificial data. No default is set, as pK should be adjusted for each scRNA-seq dataset."
- [discussion] overkill removes filtered doublets before simulating training set: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [discussion] GBM integrates method scores: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [methods] doublet barcodes output file: "The doublets' barcodes are in the file "outname_doublets.csv""
