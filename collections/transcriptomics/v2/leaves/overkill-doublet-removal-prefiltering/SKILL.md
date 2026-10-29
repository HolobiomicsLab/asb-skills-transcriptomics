---
name: overkill-doublet-removal-prefiltering
description: Use when running Chord with overkill=T on a Seurat object and likely
  doublets must be removed before artificial-doublet simulation for training.
when_to_use_negative:
- Data already contains experimental doublet labels (cell hashing, MULTI-seq, demuxlet)
  — those methods require special operations and costs but directly label doublets,
  making computational prefiltering redundant
- Running downstream boosting with additional methods on seu.robj / overkilled.robj
  — that is the 'addmethods' stage, not the initial overkill prefilter
- 'Runtime is critical: Chord (including overkill) is not time-optimal because it
  integrates results of multiple methods'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_6170
- http://edamontology.org/topic_2269
tools:
- name: Chord
  role: Hosts the overkill stage; chord() is run with overkill=T, doubletrate, overkillrate,
    and outname; overkillDB2 implements the filtering
  repo: github.com/13308204545/Chord
- name: creatSCE
  role: Converts the input Seurat object to a SingleCellExperiment so scds functions
    can run
  repo: github.com/13308204545/Chord
- name: scds
  role: Provides the cxds (cxds.ntop=500, cxds.binThresh=0) and bcds (bcds.ntop=500)
    doublet scores used for overkill ranking
  repo: github.com/kostkalab/scds
- name: DoubletFinder
  role: Provides pANN-based doublet scores in no-ground-truth mode on merged real-artificial
    data with dbf.PCs=1:10, dbf.pN=0.25
  repo: github.com/chris-mcginnis-ucsf/DoubletFinder
- name: R
  role: Runtime environment for Chord and its dependencies (adabag, gbm)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:4568841db2bae1f051ebe88326513a7a1418de4abc015f746cdf93b2d88395b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - creatSCE
  - overkillDB2
  - scds
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_6170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# overkill-doublet-removal-prefiltering

## Summary

Chord's 'overkill' preliminary filtering stage that removes the most likely doublets from a droplet scRNA-seq dataset using its three built-in detectors (scds cxds, scds bcds, and DoubletFinder) before artificial doublets are simulated for training the GBM/AdaBoost classifier.

## When to use

Use when running Chord with overkill=T on a Seurat object and likely doublets must be removed before artificial-doublet simulation for training.

## When NOT to use

- Data already contains experimental doublet labels (cell hashing, MULTI-seq, demuxlet) — those methods require special operations and costs but directly label doublets, making computational prefiltering redundant
- Running downstream boosting with additional methods on seu.robj / overkilled.robj — that is the 'addmethods' stage, not the initial overkill prefilter
- Runtime is critical: Chord (including overkill) is not time-optimal because it integrates results of multiple methods

## Inputs

- Input Seurat object (seu) of droplet scRNA-seq data
- Estimated doublet rate (doubletrate, ~0.9% per 1,000 cells per 10X guideline)
- overkillrate (fraction 0-1, top % of doublet-like cells removed per method)
- Optional overkilllist: vector of cells to be removed in overkill

## Outputs

- overkilled.robj: Seurat object containing retained (prefiltered) cells
- <outname>_doublets.csv: CSV of union of removed doublet barcodes

## How to apply

Convert the input Seurat object to a SingleCellExperiment via Chord's creatSCE(seu) so the scds functions can operate. Compute cxds scores with cxds.ntop=500 and cxds.binThresh=0, and bcds scores with bcds.ntop=500 (bcds.srat=1) using Chord's scds wrapper; then compute DoubletFinder pANN scores in no-ground-truth mode on merged real-artificial data with dbf.PCs=1:10 and dbf.pN=0.25 (default justified by DoubletFinder's pN-invariance, McGinnis et al. 2019). Rank cells by each method's score and remove the top doubletrate × overkillrate fraction per method, using the 10X guideline of doubletrate ≈ 0.9% per 1,000 cells; the method combination can be overridden by supplying overkilllist, a vector of cells to remove. Save the retained cells as overkilled.robj and the union of removed barcodes to <outname>_doublets.csv; these prefiltered cells then serve as the basis for simulating artificial doublets for the training set.

## Related tools

- **Chord** (Hosts the overkill stage; chord() is run with overkill=T, doubletrate, overkillrate, and outname; overkillDB2 implements the filtering) — github.com/13308204545/Chord
- **creatSCE** (Converts the input Seurat object to a SingleCellExperiment so scds functions can run) — github.com/13308204545/Chord
- **scds** (Provides the cxds (cxds.ntop=500, cxds.binThresh=0) and bcds (bcds.ntop=500) doublet scores used for overkill ranking) — github.com/kostkalab/scds
- **DoubletFinder** (Provides pANN-based doublet scores in no-ground-truth mode on merged real-artificial data with dbf.PCs=1:10, dbf.pN=0.25) — github.com/chris-mcginnis-ucsf/DoubletFinder
- **R** (Runtime environment for Chord and its dependencies (adabag, gbm))

## Examples

```
chord(seu="input seurat object", doubletrate="estimated doubletrate", overkill=T, outname="the name you want")
```

## Evaluation signals

- The fraction of cells removed per method equals doubletrate × overkillrate
- <outname>_doublets.csv exists and contains the expected barcode column listing the union of removed barcodes
- overkilled.robj loads as a Seurat object containing only the retained cells
- DoubletFinder step ran with pN=0.25 (default, pN-invariant) and PCs 1:10 as configured
- Simulated doublets for training are generated from the remaining cells, confirming prefiltering preceded artificial-doublet generation

## Limitations

- Chord is not optimal in time efficiency because it must integrate results of multiple methods; refactoring was proposed but not implemented
- The claim that the overkill step might also improve the performance of other methods was unverified (no test reported)
- doubletrate must be estimated by the user (10X guideline: ~0.9% per 1,000 cells); misestimation shifts how many cells are removed

## Evidence

- [discussion] Overkill removes built-in-detected doublets before simulation: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a training set."
- [methods] overkilllist parameter definition: "You can choose any combination of methods for overkill by setting the overkilllist parameter (a vector of cells to be remove in overkill)"
- [readme] doubletrate estimation guideline: "It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [readme] overkillrate parameter semantics: "**overkillrate**  an integer,remove the top ?% doublet-liked cells of any methods' results.(0-1)"
- [readme] DoubletFinder pN invariance rationale: "Default is set to 0.25, based on observation that DoubletFinder performance is largely pN-invariant (see McGinnis, Murrow and Gartner 2019, Cell Systems)."
- [methods] doublet barcodes output file: "The doublets' barcodes are in the file "outname_doublets.csv""
