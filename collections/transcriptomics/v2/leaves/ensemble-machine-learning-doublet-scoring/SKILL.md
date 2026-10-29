---
name: ensemble-machine-learning-doublet-scoring
description: Use when raw Seurat/SCE droplet scRNA-seq data contains suspected technical
  doublets and you need stable, accurate doublet calls (AUC-based) that outperform
  any single tool like DoubletFinder or scds.
when_to_use_negative:
- Inputs already have experimental doublet labels (cell hashing, MULTI-seq, demuxlet)
  — these label doublets directly, though demuxlet-type methods only label doublets
  from different samples
- 'Runtime-critical pipelines: Chord integrates results of multiple methods, so it
  is not optimal in terms of time efficiency'
- Non-droplet or merged multi-sample datasets where Scrublet-style simulation assumptions
  break down (Scrublet should be run per sample)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: 'Ensemble driver: integrates built-in doublet detectors via GBM/AdaBoost boosting;
    supports modular addition of new method scores'
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Built-in detector integrated by Chord; pN fixed at 0.25, pK tuned per dataset
    via mean-variance-normalized bimodality coefficient
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Built-in cxds/bcds detectors run on a SingleCellExperiment converted with
    creatSCE; parameters cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat
  repo: https://github.com/kostkalab/scds
- name: gbm
  role: Boosting backend used by Chord to integrate results of multiple doublet detection
    methods
- name: adabag
  role: Alternative boosting backend (adaboost, mfinal=40 default iterations) selectable
    via method='adaboost'
- name: Scrublet
  role: Standalone comparison method (Python, kNN classifier on simulated doublets)
    benchmarked against the ensemble
  repo: https://github.com/AllonKleinLab/scrublet
- name: Solo
  role: Standalone semi-supervised deep learning doublet classifier used as a comparison
    method in benchmarking
  repo: https://github.com/calico/solo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:a9da747c681a004138b0e514430f91dedfc06889f2801aa76807557e573def5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - Solo
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# ensemble-machine-learning-doublet-scoring

## Summary

Identify doublets in droplet scRNA-seq by running an ensemble (Chord) that integrates scds, DoubletFinder and other detection methods via GBM/AdaBoost boosting, with an 'overkill' pre-filter and simulated-doublet training set, to achieve more accurate and stable doublet calls than any single detector.

## When to use

Use when raw Seurat/SCE droplet scRNA-seq data contains suspected technical doublets and you need stable, accurate doublet calls (AUC-based) that outperform any single tool like DoubletFinder or scds.

## When NOT to use

- Inputs already have experimental doublet labels (cell hashing, MULTI-seq, demuxlet) — these label doublets directly, though demuxlet-type methods only label doublets from different samples
- Runtime-critical pipelines: Chord integrates results of multiple methods, so it is not optimal in terms of time efficiency
- Non-droplet or merged multi-sample datasets where Scrublet-style simulation assumptions break down (Scrublet should be run per sample)

## Inputs

- Seurat object (seu) of a droplet-based scRNA-seq dataset
- SingleCellExperiment object (sce, converted via creatSCE) for scds-based methods
- Estimated doublet rate (e.g., ~0.9% per 1,000 cells for 10X data)
- Optional method score tables: real_data.scores.csv and simulated_data.scores.csv for the modular (ChordP) port

## Outputs

- outname_doublets.csv — barcodes of predicted doublets to remove
- overkilled.robj and seu.robj intermediate objects for adding more methods
- simulated_data.scores.csv / real_data.scores.csv method score tables
- Per-cell doublet scores from each integrated method and the boosted ensemble

## How to apply

Install DoubletFinder, scds, adabag, gbm and Chord via remotes/devtools. Run chord() on the Seurat object with an estimated doubletrate (~0.9% per 1,000 cells for 10X data) and overkill=TRUE: built-in methods first remove filtered doublets, then simulated doublets built from the remaining cells form the GBM training set. Optionally extend the ensemble through the modular port by scoring 'overkilled.robj' and 'seu.robj' with any extra method and adding the scores to simulated_data.scores.csv and real_data.scores.csv, then re-run chord with addmethods1/addmethods2 (ChordP). Tune DoubletFinder's dbf.pK per dataset using the mean-variance-normalized bimodality coefficient (dbf.pN stays at its default 0.25), and set scds parameters cxds.ntop/cxds.binThresh/bcds.ntop/bcds.srat as needed. Read doublet barcodes from the output 'outname_doublets.csv' and remove those cells before downstream clustering.

## Related tools

- **Chord** (Ensemble driver: integrates built-in doublet detectors via GBM/AdaBoost boosting; supports modular addition of new method scores) — https://github.com/13308204545/Chord
- **DoubletFinder** (Built-in detector integrated by Chord; pN fixed at 0.25, pK tuned per dataset via mean-variance-normalized bimodality coefficient) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Built-in cxds/bcds detectors run on a SingleCellExperiment converted with creatSCE; parameters cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat) — https://github.com/kostkalab/scds
- **gbm** (Boosting backend used by Chord to integrate results of multiple doublet detection methods)
- **adabag** (Alternative boosting backend (adaboost, mfinal=40 default iterations) selectable via method='adaboost')
- **Scrublet** (Standalone comparison method (Python, kNN classifier on simulated doublets) benchmarked against the ensemble) — https://github.com/AllonKleinLab/scrublet
- **Solo** (Standalone semi-supervised deep learning doublet classifier used as a comparison method in benchmarking) — https://github.com/calico/solo

## Examples

```
remotes::install_github('chris-mcginnis-ucsf/DoubletFinder'); devtools::install_github('kostkalab/scds',ref="master"); install.packages("gbm"); devtools::install_github("13308204545/Chord"); chord(seu=seu, doubletrate=0.009, overkill=T, outname="run1")
```

## Evaluation signals

- ChordP achieved the highest average rank (2.285) across seven ground-truth datasets, with highest average AUC, pAUC800, pAUC900, pAUC950, pAUC975; ChordP average AUC 0.8132 and Chord average AUC 0.801214286 serve as reference values
- outname_doublets.csv exists, contains only cell barcodes, and predicted doublets co-localize in a 2-D UMAP/t-SNE embedding
- overkill step completes: filtered doublets are removed before simulating doublets from the remaining cells for the training set
- Modular re-run with addmethods1/addmethods2 produces a second output (outname2_doublets.csv) whose scores changed relative to the first run
- DoubletFinder pK was re-tuned per dataset (not left at default) via the mean-variance-normalized bimodality coefficient

## Limitations

- Chord requires integrating results of multiple methods, so it is not optimal in terms of time efficiency
- The overkill step's benefit for other methods was proposed but not tested; refactoring to address runtime was proposed but not implemented
- Existing experimental techniques (cell hashing, MULTI-seq, demuxlet) only label doublets from different samples and cannot retroactively annotate existing datasets

## Evidence

- [discussion] Ensemble integration via GBM: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [discussion] Overkill training-set construction: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a training set."
- [readme] Core invocation and output file: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [readme] Doublet rate heuristic: "A:It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [readme] Modular re-run with added method scores: "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 ="simulated_data.scores.csv" )"
- [results] Benchmark result on ground-truth datasets: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime limitation: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
