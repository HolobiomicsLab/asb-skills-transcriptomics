---
name: seurat-single-cell-object-handling
description: 'Use when running Chord or its built-in detectors (scds, DoubletFinder)
  on scRNA-seq data: the input must be a Seurat object, and scds additionally requires
  an sce obtained via creatSCE(seu).'
when_to_use_negative:
- Input is not a droplet-based scRNA-seq dataset for which doublets must be removed
  (e.g. data already demultiplexed/labelled by cell hashing, MULTI-seq or demuxlet)
- You need a stand-alone method score only (e.g. running Scrublet or Solo directly
  on a counts matrix) without the Seurat/SCE object plumbing of the Chord ensemble
- Seurat and scds/DoubletFinder packages cannot be installed (this skill depends on
  remotes::install_github of DoubletFinder, scds and Chord)
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Chord
  role: Consumes the Seurat object as seu (and sce from creatSCE) and writes doublet
    barcodes to outname_doublets.csv
  repo: https://github.com/13308204545/Chord
- name: scds
  role: SingleCellExperiment-based built-in detector (cxds/bcds) run on the sce converted
    from the Seurat object
  repo: https://github.com/kostkalab/scds
- name: DoubletFinder
  role: R/Seurat dependency installed alongside Chord; dbf.pN default 0.25 and dbf.pK
    tuned per dataset operate on the same Seurat object
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: R
  role: Language in which the Seurat/SCE objects and the chord() call are handled
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:b6555295be71cd532a3b52a3f22f2d409a859248dd3ea5fb981ff1b86f297930
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# seurat-single-cell-object-handling

## Summary

Prepare and pass Seurat/SingleCellExperiment single-cell objects through the Chord doublet-detection workflow: load annotated datasets as Seurat objects, convert them with creatSCE for SCE-based methods like scds, and supply them (or persisted seu.robj/sce.robj files) as the seu/sce parameters of chord().

## When to use

Use when running Chord or its built-in detectors (scds, DoubletFinder) on scRNA-seq data: the input must be a Seurat object, and scds additionally requires an sce obtained via creatSCE(seu).

## When NOT to use

- Input is not a droplet-based scRNA-seq dataset for which doublets must be removed (e.g. data already demultiplexed/labelled by cell hashing, MULTI-seq or demuxlet)
- You need a stand-alone method score only (e.g. running Scrublet or Solo directly on a counts matrix) without the Seurat/SCE object plumbing of the Chord ensemble
- Seurat and scds/DoubletFinder packages cannot be installed (this skill depends on remotes::install_github of DoubletFinder, scds and Chord)

## Inputs

- Seurat object (seu) holding the scRNA-seq count matrix for one sample
- SingleCellExperiment object (sce) produced by creatSCE(seu) for scds-based methods
- Persisted workflow files from a first Chord run: seu.robj, sce.robj, overkilled.robj
- Method score tables to merge: real_data.scores.csv (addmethods1) and simulated_data.scores.csv (addmethods2)

## Outputs

- Doublet barcode list written to 'outname_doublets.csv' (or 'outname2_doublets.csv' after boosting)
- Persisted R objects seu.robj, sce.robj, overkilled.robj for later method augmentation
- Updated simulated_data.scores.csv / real_data.scores.csv with added method scores

## How to apply

Load each dataset (e.g. the ground-truth sets DM-A, DM-B, DM-C, HTO12, HTO8, DM-2.1, DM-2.2) as a Seurat object, then call creatSCE(seu) to build the SingleCellExperiment that scds-based functions (cxds/bcds with cxds.ntop=500, cxds.binThresh=0, bcds.ntop=500, bcds.srat=1) require. Pass the Seurat object as the seu argument to chord() along with doubletrate estimated from cell count (~0.9% per 1,000 cells for 10X data), overkill=TRUE, and an outname prefix. During the overkill step Chord persists the data as seu.robj, sce.robj, and overkilled.robj; when boosting with additional methods, load('seu.robj') and load('sce.robj') in the same directory and re-run chord with addmethods1='real_data.scores.csv' and addmethods2='simulated_data.scores.csv'. Doublet barcodes are then retrieved from 'outname_doublets.csv' (or 'outname2_doublets.csv' for the boosted run).

## Related tools

- **Chord** (Consumes the Seurat object as seu (and sce from creatSCE) and writes doublet barcodes to outname_doublets.csv) — https://github.com/13308204545/Chord
- **scds** (SingleCellExperiment-based built-in detector (cxds/bcds) run on the sce converted from the Seurat object) — https://github.com/kostkalab/scds
- **DoubletFinder** (R/Seurat dependency installed alongside Chord; dbf.pN default 0.25 and dbf.pK tuned per dataset operate on the same Seurat object) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **R** (Language in which the Seurat/SCE objects and the chord() call are handled)

## Examples

```
chord(seu = seu, sce = creatSCE(seu), doubletrate = 0.009, overkill = TRUE, outname = "sample1")
```

## Evaluation signals

- creatSCE(seu) returns a valid SingleCellExperiment on which scds(sce, cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat) runs without error
- A first chord() run produces the expected side files: seu.robj, sce.robj, overkilled.robj and 'outname_doublets.csv'
- Barcodes in outname_doublets.csv are a subset of the Seurat object's cell barcodes and their number is consistent with the estimated doubletrate (~0.9% per 1,000 cells for 10X)
- After boosting, re-running chord with addmethods1='real_data.scores.csv' and addmethods2='simulated_data.scores.csv' yields outname2_doublets.csv without object-loading errors
- On the annotated benchmark datasets, downstream AUC/AUPRC scoring against experimental doublet labels is computable from the returned barcodes (e.g. ChordP average AUC ≈ 0.8132)

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods
- The doublet rate must be estimated manually from the number of cells in the sample (10X: ~0.9% per 1,000 cells); a wrong rate mis-sizes the simulated training set
- dbf.pK has no default and must be adjusted for each dataset, e.g. via the mean-variance-normalized bimodality coefficient
- Other pre-processing steps, such as ambient mRNA removal for droplets, are not incorporated by the workflow

## Evidence

- [methods] Run Chord with input Seurat object, estimated doublet rate, overkill setting, and output name: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [readme] Boosted run reloads persisted objects: "load("seu.robj")
load("sce.robj")
chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1"
- [readme] Overkill persistence files: "Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv"."
- [readme] Doublet rate estimation rule: "It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [readme] Output barcode file: "The doublets' barcodes are in the file "outname_doublets.csv""
