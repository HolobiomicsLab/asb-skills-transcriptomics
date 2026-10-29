---
name: doublet-detection-ensemble-classification
description: Use when processing droplet-based scRNA-seq count data (Seurat object
  or SingleCellExperiment) where doublets are a suspected technical artefact and a
  stable doublet set is needed before downstream clustering or DEG analysis.
when_to_use_negative:
- 'Data were generated with experimental doublet labelling (cell hashing, MULTI-seq,
  demuxlet): these methods already label doublets and Chord adds computational inference
  rather than replacing genetic/sample-tag demultiplexing'
- 'Analysis is strictly time-critical: Chord integrates results of multiple methods,
  so it is not optimal in terms of time efficiency'
- Input is not droplet-based scRNA-seq with putative doublets (e.g. already demultiplexed
  or plate-based data without doublet artefacts)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Chord
  role: Ensemble GBM/Adaboost classifier that integrates built-in doublet detectors
    and outputs doublet barcodes
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Built-in individual detector whose artificial-doublet pANN scores feed the
    ensemble; pN default 0.25, pK tuned per dataset
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Built-in detector providing cxds and bcds doublet scores (cxds.ntop 500, bcds.ntop
    500 defaults) integrated by Chord
  repo: https://github.com/kostkalab/scds
- name: gbm
  role: Gradient boosting machine used by Chord to integrate results of different
    methods
- name: adabag
  role: R package providing the alternative adaboost integration method
- name: remotes
  role: Used to install DoubletFinder and Chord from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:4721400cdccf76d7eb91fb9751eefdf818f014a3eeca21c5f1be2dcdfb0805e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - MAST
  - R
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# doublet-detection-ensemble-classification

## Summary

Identify and remove doublets from droplet-based scRNA-seq data by integrating multiple doublet-detection methods (scds cxds/bcds, DoubletFinder) into a single ensemble call using Chord's GBM/AdaBoost machine-learning classifier, yielding more stable and accurate doublet calls than any individual method.

## When to use

Use when processing droplet-based scRNA-seq count data (Seurat object or SingleCellExperiment) where doublets are a suspected technical artefact and a stable doublet set is needed before downstream clustering or DEG analysis.

## When NOT to use

- Data were generated with experimental doublet labelling (cell hashing, MULTI-seq, demuxlet): these methods already label doublets and Chord adds computational inference rather than replacing genetic/sample-tag demultiplexing
- Analysis is strictly time-critical: Chord integrates results of multiple methods, so it is not optimal in terms of time efficiency
- Input is not droplet-based scRNA-seq with putative doublets (e.g. already demultiplexed or plate-based data without doublet artefacts)

## Inputs

- Seurat object (seu) of raw droplet scRNA-seq counts
- SingleCellExperiment object (sce) converted via creatSCE(seu)
- Estimated doublet rate for the library (e.g. ~0.9% per 1,000 cells for 10X)
- Optional: score tables from additional detectors (real_data.scores.csv, simulated_data.scores.csv)

## Outputs

- outname_doublets.csv listing predicted doublet barcodes to remove
- Doublet-filtered single-cell dataset for downstream clustering/DEG analysis
- Intermediate artefacts: overkilled.robj, seu.robj, simulated_data.scores.csv
- Optional refined doublet calls (outname2_doublets.csv) after boosting extra methods

## How to apply

Install DoubletFinder, scds, adabag, gbm, and Chord from their GitHub/package sources. Convert the input Seurat object with creatSCE(seu), then run chord(seu, sce, doubletrate, overkill=T, outname=...) — the overkill step first removes filtered doublets identified by built-in methods, then simulates doublets from the remaining cells to build the GBM training set; estimated doublet rate is ~0.9% per 1,000 cells for 10X data. Retrieve doublet barcodes from outname_doublets.csv and remove those cells. Optionally set dbf.pN (default 0.25, DoubletFinder is pN-invariant), tune dbf.pK per dataset via mean-variance-normalized bimodality coefficient, and customise the overkill method set with overkilllist. To incorporate additional detectors, score the overkilled.robj and seu.robj datasets, write scores to real_data.scores.csv / simulated_data.scores.csv, and re-run chord with addmethods1/addmethods2; the boosting engine itself is selected via the method parameter ('gbm' or 'adaboost').

## Related tools

- **Chord** (Ensemble GBM/Adaboost classifier that integrates built-in doublet detectors and outputs doublet barcodes) — https://github.com/13308204545/Chord
- **DoubletFinder** (Built-in individual detector whose artificial-doublet pANN scores feed the ensemble; pN default 0.25, pK tuned per dataset) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Built-in detector providing cxds and bcds doublet scores (cxds.ntop 500, bcds.ntop 500 defaults) integrated by Chord) — https://github.com/kostkalab/scds
- **gbm** (Gradient boosting machine used by Chord to integrate results of different methods)
- **adabag** (R package providing the alternative adaboost integration method)
- **remotes** (Used to install DoubletFinder and Chord from GitHub)

## Examples

```
chord(seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want")
```

## Evaluation signals

- Doublet barcodes are written to outname_doublets.csv and the number of removed cells matches the estimated doubletrate applied
- On the synthetic DataDEG dataset (40% labelled doublets), TPR/TNR/accuracy of DEG calls after Chord-based filtering should lie closest to the clean (no-doublet) reference compared with filtering by DoubletFinder or scds alone
- Re-running chord with addmethods1/addmethods2 produces outname2_doublets.csv with refined calls relative to the first run
- Overkill behaviour: simulated doublets are generated only from cells remaining after removing built-in-method doublets, so the training set size shrinks accordingly
- Setting overkilllist to a custom cell vector changes exactly that set of cells removed during overkill

## Limitations

- Chord requires integration of results of multiple methods, so it is not optimal in terms of time efficiency
- dbf.pK has no default and must be tuned for each scRNA-seq dataset (e.g. via mean-variance-normalized bimodality coefficient)
- Other pre-processing steps, such as ambient mRNA removal for droplets, are not incorporated and may be needed to improve downstream analysis accuracy
- The claim that the overkill step might also improve the performance of other methods was proposed but not tested in the paper

## Evidence

- [discussion] Ensemble GBM integration: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [discussion] Overkill training-set construction: "First, built-in methods are used to evaluate the data. Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells"
- [abstract] Accuracy and stability: "Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data."
- [methods] chord invocation: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [readme] Doublet barcode output: "A:The doublets' barcodes are in the file "outname_doublets.csv""
- [readme] DoubletFinder pN default: "Default is set to 0.25, based on observation that DoubletFinder performance is largely pN-invariant (see McGinnis, Murrow and Gartner 2019, Cell Systems)."
- [discussion] Time-efficiency limitation: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
