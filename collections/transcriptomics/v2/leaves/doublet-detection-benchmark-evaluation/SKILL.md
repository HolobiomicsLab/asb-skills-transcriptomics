---
name: doublet-detection-benchmark-evaluation
description: Use when you need to compare doublet detectors on datasets with experimental
  doublet annotations (e.g., cell-hashed DM-A/DM-B/DM-C, HTO12/HTO8, multiplexed DM-2.1/DM-2.2),
  or when choosing between detectors where spurious trajectory branches matter.
when_to_use_negative:
- The input dataset already has experimental doublet labels from cell hashing, MULTI-seq,
  or demuxlet that can directly remove doublets — computational detection then adds
  cost without benefit
- The merged dataset combines samples with non-representative cell-type proportions
  — Scrublet is designed for technical doublets within a single sample and 'may perform
  poorly on merged datasets', so run it per sample instead
- Runtime is the dominant constraint — Chord 'is not optimal in terms of time efficiency'
  because it must integrate the results of multiple methods
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3172
tools:
- name: Chord
  role: Ensemble (GBM/AdaBoost) doublet detector run with overkill=TRUE on each dataset;
    reference method whose predictions are benchmarked
  repo: github.com/13308204545/Chord
- name: DoubletFinder
  role: Standalone detector run with pN=0.25 and per-dataset pK tuning via mean-variance-normalized
    bimodality coefficient
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Provides the cxds and bcds standalone detectors run on SingleCellExperiment
    objects with cxds.ntop=500, cxds.binThresh=0, bcds.ntop=500, bcds.srat=1
  repo: https://github.com/kostkalab/scds
- name: Scrublet
  role: Python standalone detector scoring cells via simulated doublets and a k-nearest-neighbor
    classifier; also used in the DataPSE trajectory comparison
  repo: https://github.com/AllonKleinLab/scrublet
- name: Solo
  role: Semi-supervised deep-learning doublet classifier benchmarked as one of the
    standalone detectors
  repo: https://github.com/calico/solo
- name: Slingshot
  role: Cluster-based MST pseudotime inference used to check for spurious branches
    after doublet removal
- name: monocle
  role: Reversed-graph-embedding trajectory inference used as the second downstream-integrity
    check
- name: adabag
  role: R package providing AdaBoost as an alternative boosting method for the Chord
    ensemble (method='adaboost')
- name: gbm
  role: R package providing the GBM boosting algorithm Chord uses to integrate method
    scores
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:40c72d7dd67c8a39835690a050b3e522ae58fa9622710b05da5ee097926c8379
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - Slingshot
  - Solo
  - adabag
  - doubletCells
  - gbm
  - monocle
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# doublet-detection-benchmark-evaluation

## Summary

Benchmark doublet detection methods (Chord, ChordP, DoubletFinder, scds bcds/cxds, Scrublet, Solo, doubletCells) against experimentally-annotated ground-truth scRNA-seq datasets, scoring predictions with AUC, AUPRC, and partial AUCs at fixed true-positive counts, and optionally testing downstream trajectory integrity.

## When to use

Use when you need to compare doublet detectors on datasets with experimental doublet annotations (e.g., cell-hashed DM-A/DM-B/DM-C, HTO12/HTO8, multiplexed DM-2.1/DM-2.2), or when choosing between detectors where spurious trajectory branches matter.

## When NOT to use

- The input dataset already has experimental doublet labels from cell hashing, MULTI-seq, or demuxlet that can directly remove doublets — computational detection then adds cost without benefit
- The merged dataset combines samples with non-representative cell-type proportions — Scrublet is designed for technical doublets within a single sample and 'may perform poorly on merged datasets', so run it per sample instead
- Runtime is the dominant constraint — Chord 'is not optimal in terms of time efficiency' because it must integrate the results of multiple methods

## Inputs

- Seurat object of an annotated ground-truth scRNA-seq dataset (DM-A, DM-B, DM-C, HTO12, HTO8, DM-2.1, DM-2.2) with experimental doublet labels
- SingleCellExperiment object created via creatSCE(seu) for scds-based methods
- Synthetic pseudotime dataset (DataPSE: 600 cells, bifurcating trajectory, 20% labelled doublets)
- Estimated doublet rate from cell count (~0.9% per 1,000 cells for 10X data)
- Raw UMI counts matrix for Scrublet (cells as rows, genes as columns)

## Outputs

- outname_doublets.csv with doublet barcodes per method/dataset
- metrics_summary.csv of AUC, AUPRC, and pAUC800/900/950/975 across 9 methods x 7 datasets
- Average rank table comparable to Table 1 (e.g., ChordP average AUC 0.8132, Chord 0.801214286, average rank 2.285)
- trajectory_comparison.png showing per-method Slingshot/monocle branch structure versus the clean-data reference
- overkilled.robj, seu.robj, simulated_data.scores.csv, real_data.scores.csv when using the ChordP modular port

## How to apply

Load each annotated dataset as a Seurat object and convert with creatSCE for SingleCellExperiment-based methods. Run the individual detectors — scds' cxds (cxds.ntop=500, cxds.binThresh=0) and bcds (bcds.ntop=500, bcds.srat=1), DoubletFinder with pN fixed at 0.25 and pK tuned per dataset via the mean-variance-normalized bimodality coefficient, plus Scrublet, Solo, and doubletCells — and run Chord via chord(seu, doubletrate, overkill=TRUE, outname) with the doublet rate estimated as ~0.9% per 1,000 cells for 10X data; retrieve barcodes from outname_doublets.csv. Run ChordP through the modular port by supplying extra method scores as addmethods1 (real_data.scores.csv) and addmethods2 (simulated_data.scores.csv). Score every method's predictions against the experimental annotations computing AUC, AUPRC, and partial AUC at 800/900/950/975 true-positive counts, assemble a 9-methods x 7-datasets metrics_summary.csv, and compare average ranks (reference: ChordP average AUC 0.8132, average rank 2.285). For downstream-impact benchmarking, follow the DataPSE protocol: compute each method's scores on the 600-cell synthetic bifurcating trajectory with 20% labelled doublets, remove exactly the top 20% of cells by score so all methods are compared at equal stringency, then re-infer trajectories with Slingshot and monocle and check for spurious extra branches versus the clean reference.

## Related tools

- **Chord** (Ensemble (GBM/AdaBoost) doublet detector run with overkill=TRUE on each dataset; reference method whose predictions are benchmarked) — github.com/13308204545/Chord
- **DoubletFinder** (Standalone detector run with pN=0.25 and per-dataset pK tuning via mean-variance-normalized bimodality coefficient) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Provides the cxds and bcds standalone detectors run on SingleCellExperiment objects with cxds.ntop=500, cxds.binThresh=0, bcds.ntop=500, bcds.srat=1) — https://github.com/kostkalab/scds
- **Scrublet** (Python standalone detector scoring cells via simulated doublets and a k-nearest-neighbor classifier; also used in the DataPSE trajectory comparison) — https://github.com/AllonKleinLab/scrublet
- **Solo** (Semi-supervised deep-learning doublet classifier benchmarked as one of the standalone detectors) — https://github.com/calico/solo
- **Slingshot** (Cluster-based MST pseudotime inference used to check for spurious branches after doublet removal)
- **monocle** (Reversed-graph-embedding trajectory inference used as the second downstream-integrity check)
- **adabag** (R package providing AdaBoost as an alternative boosting method for the Chord ensemble (method='adaboost'))
- **gbm** (R package providing the GBM boosting algorithm Chord uses to integrate method scores)

## Examples

```
chord(seu=seu, doubletrate=0.009, overkill=TRUE, outname="groundtruth_run")
```

## Evaluation signals

- metrics_summary.csv assembles all 9 methods x 7 datasets with AUC, AUPRC, and partial AUCs at 800/900/950/975 true-positive counts
- Reproduction matches Table 1 reference values: ChordP average AUC 0.8132, Chord average AUC 0.801214286, ChordP highest average rank 2.285
- Doublet barcodes for each run are retrievable from outname_doublets.csv and are non-empty at the estimated doublet rate (~0.9% per 1,000 cells)
- On DataPSE, after removing exactly the top 20% scored cells (equal stringency across methods), Chord- and Scrublet-filtered data yield Slingshot/monocle trajectories with no extra branches versus the clean reference
- bcds-, cxds- and doubletCells-filtered DataPSE data should reproduce spurious additional branches in Slingshot and/or monocle

## Limitations

- Chord is not time-optimal because it must integrate results of multiple methods; refactoring the integrated methods is proposed but not implemented
- DoubletFinder pK has no default and must be adjusted for each dataset via the mean-variance-normalized bimodality coefficient
- Scrublet may perform poorly on merged multi-sample datasets with unrepresentative cell-type proportions; run per sample
- The claim that the 'overkill' step might also improve other methods' performance is unverified in the paper
- Computational methods cannot label doublets from different samples and do not help existing scRNA-seq data the way cell hashing, MULTI-seq, or demuxlet can

## Evidence

- [abstract] Chord implements ensemble integration benchmarking: "We here propose Chord which implements a machine learning algorithm that integrates multiple doublet detection methods to address these issues."
- [methods] Run Chord with Seurat input, estimated doubletrate, overkill setting, output name: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [methods] ChordP modular port re-runs chord with added method scores: "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" )"
- [readme] DoubletFinder pN default 0.25 rationale and pK per-dataset tuning: "Default is set to 0.25, based on observation that DoubletFinder performance is largely pN-invariant"
- [intro] Motivation for benchmarking: variable performance across datasets: "the performance of these computational methods varies greatly"
- [discussion] Overkill step removes filtered doublets before simulating training doublets: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [readme] Scrublet score interpretation best practices: [evidence span withheld — anchor status: external_doc]
