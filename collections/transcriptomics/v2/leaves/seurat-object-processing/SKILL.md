---
name: seurat-object-processing
description: Use when a scRNA-seq dataset is stored as a Seurat object but the downstream
  doublet-detection tool (Chord/scds) requires a SingleCellExperiment object, or when
  preparing per-sample Seurat objects for Chord.
when_to_use_negative:
- Input data is already a SingleCellExperiment object — creatSCE() conversion is unnecessary
- Data comes from a non-R pipeline (e.g. Python scanpy/AnnData .h5ad) that cannot
  be loaded directly as a Seurat object without an intermediate conversion
- The goal is doublet detection itself rather than preparing objects — use the Chord
  ensemble detection skill instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: Provides the creatSCE() conversion function and consumes the converted Seurat/sce
    objects in chord() and overkillDB2() calls
  repo: https://github.com/13308204545/Chord
- name: scds
  role: Built-in method that evaluates the converted SingleCellExperiment object during
    the overkill step
  repo: https://github.com/kostkalab/scds
- name: DoubletFinder
  role: Built-in method operating on the Seurat/SingleCellExperiment data during the
    overkill evaluation step
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: R
  role: Runtime environment in which Seurat objects are loaded, converted, and passed
    to Chord
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:0566235cea87045182eb825acbfdcd7dad4bf3bb6336e0b08d2661163f060398
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# seurat-object-processing

## Summary

Processing Seurat single-cell objects into the SingleCellExperiment format required by the Chord doublet-detection pipeline, via the creatSCE() conversion function, before running ensemble doublet detection on scRNA-seq data.

## When to use

Use when a scRNA-seq dataset is stored as a Seurat object but the downstream doublet-detection tool (Chord/scds) requires a SingleCellExperiment object, or when preparing per-sample Seurat objects for Chord.

## When NOT to use

- Input data is already a SingleCellExperiment object — creatSCE() conversion is unnecessary
- Data comes from a non-R pipeline (e.g. Python scanpy/AnnData .h5ad) that cannot be loaded directly as a Seurat object without an intermediate conversion
- The goal is doublet detection itself rather than preparing objects — use the Chord ensemble detection skill instead

## Inputs

- Seurat object containing a single scRNA-seq sample (e.g. one of 5 NSCLC tumour samples, samples 11, 13, 17, 18, 22; 24,280 cells total)
- Cell-type annotations within the Seurat object for downstream per-cell-type doublet tabulation

## Outputs

- SingleCellExperiment (sce) object produced by creatSCE(seu)
- Chord output files: 'outname_doublets.csv' (predicted doublet barcodes), 'seu.robj' and 'sce.robj' (saved objects), 'simulated_data.scores.csv' (per-method scores)
- Per-sample and per-cell-type doublet proportion tables

## How to apply

Load the Seurat object for the sample of interest (e.g. the NSCLC tumour samples from the KU Leuven deposit) and convert it with the creatSCE() function to obtain a SingleCellExperiment ('sce') object, since Chord's chord() call accepts both a 'seu' and a 'sce' argument and its internal overkillDB2() step operates on both. Then pass the converted object to chord() with the estimated doublet rate (~0.9% per 1,000 cells, the 10X rule of thumb), overkill=T, and an output name; Chord evaluates the data with built-in methods (scds, DoubletFinder), removes filtered doublets, and simulates doublets from remaining cells to build the GBM training set. Doublet barcodes are retrieved from 'outname_doublets.csv'. Chord also writes 'seu.robj' and 'sce.robj' files, so the converted objects can be reloaded to boost additional methods via addmethods1/addmethods2.

## Related tools

- **Chord** (Provides the creatSCE() conversion function and consumes the converted Seurat/sce objects in chord() and overkillDB2() calls) — https://github.com/13308204545/Chord
- **scds** (Built-in method that evaluates the converted SingleCellExperiment object during the overkill step) — https://github.com/kostkalab/scds
- **DoubletFinder** (Built-in method operating on the Seurat/SingleCellExperiment data during the overkill evaluation step) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **R** (Runtime environment in which Seurat objects are loaded, converted, and passed to Chord)

## Examples

```
library(Chord); sce <- creatSCE(seu); chord(seu=seu, sce=sce, doubletrate=0.044, overkill=T, outname="sample11")
```

## Evaluation signals

- creatSCE(seu) returns a valid SingleCellExperiment object that Chord's chord(seu=..., sce=...) call accepts without error
- Chord writes the expected companion files ('seu.robj', 'sce.robj', 'outname_doublets.csv') in the working directory
- Cell count in the converted object matches the source Seurat object (e.g. sample totals summing to 24,280 cells for the 5 NSCLC samples)
- Per-sample doublet rates from downstream chord() runs fall in the reported 3.81–4.74% range for the NSCLC samples
- Per-cell-type doublet proportions reproduce reported values (myeloid 8.15%, fibroblasts 1.69%)

## Limitations

- Expected doublet rate must be estimated from cell count (10X rule: ~0.9% per 1,000 cells); incorrect estimation propagates into doublet calls
- When multiple samples are involved, each sample's Seurat object must be processed separately, since doublet simulation depends on sample-specific cell-type composition
- Chord is not optimal in runtime because it must integrate the results of multiple methods

## Evidence

- [discussion] Chord integrates the results of different methods through the GBM algorithm.: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [methods] chord() signature with seu and sce inputs: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [methods] overkill function operates on both seu and sce objects: "overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt)"
- [readme] doublet barcodes output file: "The doublets' barcodes are in the file "outname_doublets.csv""
- [readme] doublet rate estimation rule: "10X can be referred：doubletrate = ~0.9% per 1,000 cells"
- [readme] reload saved objects to boost more methods: "load("seu.robj")
load("sce.robj")"
