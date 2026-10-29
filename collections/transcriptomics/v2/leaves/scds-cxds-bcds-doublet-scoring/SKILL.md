---
name: scds-cxds-bcds-doublet-scoring
description: Use when a Seurat/SingleCellExperiment droplet dataset needs per-cell
  doublet scores from cxds and bcds, either for overkill pre-filtering or as GBM predictors
  in Chord.
when_to_use_negative:
- Data already scored or removed by experimental techniques (cell hashing, MULTI-seq,
  demuxlet) — these label doublets directly and scds scoring is redundant
- Python-based pipelines on h5ad/10x mtx where a deep-learning tool like solo is preferred
  over R/scds
- When scds alone is expected to be the gold standard — the article notes performance
  of individual computational methods varies greatly across datasets, which is why
  Chord ensembles them
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scds
  role: Provides the cxds and bcds doublet-scoring functions run on the SingleCellExperiment
    object with Chord-exposed parameters (cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat)
  repo: kostkalab/scds
- name: Chord
  role: Wraps scds via creatSCE and the scds(sce, ...) function; consumes the scores
    for overkill removal and GBM training
  repo: 13308204545/Chord
- name: R
  role: Runtime environment for scds, Seurat/SingleCellExperiment and the Chord workflow
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:02eec7101b07b237d10824e956b7c83d43f1b945267b53de018d8232125ac2d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kostkalab/scds
  tools:
  - Chord
  - R
  - creatSCE
  - gbm
  - overkillDB2
  - scds
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# scds-cxds-bcds-doublet-scoring

## Summary

Computes per-cell doublet scores for droplet scRNA-seq data using the scds package's two predictors — cxds (co-expression-based) and bcds (artificial-doublet kNN-based) — as standalone filtering or as predictor features for Chord's GBM ensemble.

## When to use

Use when a Seurat/SingleCellExperiment droplet dataset needs per-cell doublet scores from cxds and bcds, either for overkill pre-filtering or as GBM predictors in Chord.

## When NOT to use

- Data already scored or removed by experimental techniques (cell hashing, MULTI-seq, demuxlet) — these label doublets directly and scds scoring is redundant
- Python-based pipelines on h5ad/10x mtx where a deep-learning tool like solo is preferred over R/scds
- When scds alone is expected to be the gold standard — the article notes performance of individual computational methods varies greatly across datasets, which is why Chord ensembles them

## Inputs

- Seurat object (seu) of droplet scRNA-seq counts, converted via creatSCE(seu) to SingleCellExperiment
- Doublet rate estimate (doubletrate, e.g. ~0.9% per 1,000 cells for 10X) and overkillrate when used for overkill filtering
- Optional tuning: cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat

## Outputs

- Per-cell cxds and bcds doublet scores on the input SCE cells
- In overkill mode: prefiltered cells saved as overkilled.robj and union of removed barcodes to <outname>_doublets.csv
- Score columns for the simulated_doublet training set feeding the GBM predictors

## How to apply

Convert the input Seurat object to a SingleCellExperiment with Chord's creatSCE(seu) since scds operates on SCE objects. Run scds(sce, cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat) with the Chord defaults cxds.ntop=500, cxds.binThresh=0, bcds.ntop=500, bcds.srat=1; cxds ranks the top-variance genes for co-expression binarization (binThresh is the minimum counts to call a gene 'present'), while bcds simulates in-silico doublets at a cell:simulated-doublet ratio given by srat and scores via kNN. In Chord's first 'overkill' stage, rank cells by each method's score and remove the top doubletrate*overkillrate fraction of most-likely doublets per method (doubletrate ≈ 0.9% per 1,000 cells per the 10X guideline). In the second stage, the same cxds/bcds scores computed on the original data and on the simulated-doublet training set are assembled as predictors, labeled positives (simulated doublets) vs negatives (singlets), for the GBM classifier (1000 trees, shrinkage=0.01, cv.folds=5).

## Related tools

- **scds** (Provides the cxds and bcds doublet-scoring functions run on the SingleCellExperiment object with Chord-exposed parameters (cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat)) — kostkalab/scds
- **Chord** (Wraps scds via creatSCE and the scds(sce, ...) function; consumes the scores for overkill removal and GBM training) — 13308204545/Chord
- **R** (Runtime environment for scds, Seurat/SingleCellExperiment and the Chord workflow)

## Examples

```
sce <- creatSCE(seu); scores <- scds(sce, cxds.ntop=500, cxds.binThresh=0, bcds.ntop=500, bcds.srat=1)
```

## Evaluation signals

- Scores are returned for every cell of the input SCE object (one cxds and one bcds score per barcode)
- Parameter invariants: cxds.ntop=500, cxds.binThresh=0, bcds.ntop=500, bcds.srat=1 unless explicitly overridden
- In overkill mode, the fraction of cells removed per method equals doubletrate × overkillrate
- Removed-barcodes file <outname>_doublets.csv exists with the expected barcode column; overkilled.robj loads as a Seurat object containing only retained cells
- Scores computed on overkilled.robj and seu.robj correctly label simulated doublets as positives and singlets as negatives for GBM training

## Limitations

- Individual method performance varies greatly and no single method is a gold standard for each scenario — cxds/bcds scores alone are less stable than the ensemble
- In Chord's overkill stage, coarse removal may discard borderline cells before the training set is simulated
- Chord overall is not optimal in runtime because it integrates multiple methods' results

## Evidence

- [methods] scds(sce, cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat): "scds(sce, cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat)"
- [readme] cxds.ntop default: "cxds.ntop integer, indimessageing number of top variance genes to consider. Default: 500"
- [readme] bcds.srat default: "bcds.srat numeric, indicating ratio between orginal number of "cells" and simulated doublets; Default: 1"
- [discussion] overkill stage rationale: "First, built-in methods are used to evaluate the data. Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used"
- [methods] scds installation: "devtools::install_github('kostkalab/scds',ref="master")"
