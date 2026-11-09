---
name: scrnaseq-doublet-detection-ensemble-analysis
description: Use when processing droplet-based scRNA-seq data (Seurat/SingleCellExperiment
  objects) that lacks experimental doublet labels (no cell hashing, MULTI-seq, or
  demuxlet), and doublets must be called computationally per sample before downstream
  clustering or trajectory analyses.
when_to_use_negative:
- When experimental doublet labels are available (cell hashing, MULTI-seq, demuxlet)
  — these directly label doublets and Chord's computational inference is unnecessary
- When runtime is critical — Chord must integrate results of multiple methods, so
  it is 'not optimal in terms of time efficiency'
- When the input is not a droplet-based scRNA-seq dataset with co-encapsulated-cell
  artefacts — doublet removal addresses a droplet-microfluidics-specific problem
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: Ensemble method integrating built-in doublet detectors via GBM/AdaBoost; produces
    final doublet calls and the 'outname_doublets.csv' barcode file
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Built-in doublet detection method evaluated during the overkill step; its
    pK parameter should be tuned per dataset
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Built-in method (cxds/bcds) providing doublet scores that feed Chord's GBM
    ensemble; parameters cxds.ntop (default 500), cxds.binThresh (default 0), bcds.ntop
    (default 500), bcds.srat (default 1)
  repo: https://github.com/kostkalab/scds
- name: adabag
  role: Provides the AdaBoost boosting option for integrating per-method scores
- name: gbm
  role: Gradient boosting machine algorithm used to integrate results of different
    methods into a final singlet/doublet classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:c595562b455f9dd57bc884330e2af7a67ab35b307242359649c1aa2e27b125a8
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# scrnaseq-doublet-detection-ensemble-analysis

## Summary

Detect and remove doublets in droplet scRNA-seq data using Chord, an ensemble machine learning method that integrates multiple doublet detection tools (scds, DoubletFinder) via GBM/AdaBoost, with an 'overkill' step that removes consensus doublets before simulating a training set. Validated on NSCLC tumour samples where it estimated per-sample doublet rates of 3.81–4.74%.

## When to use

Use when processing droplet-based scRNA-seq data (Seurat/SingleCellExperiment objects) that lacks experimental doublet labels (no cell hashing, MULTI-seq, or demuxlet), and doublets must be called computationally per sample before downstream clustering or trajectory analyses.

## When NOT to use

- When experimental doublet labels are available (cell hashing, MULTI-seq, demuxlet) — these directly label doublets and Chord's computational inference is unnecessary
- When runtime is critical — Chord must integrate results of multiple methods, so it is 'not optimal in terms of time efficiency'
- When the input is not a droplet-based scRNA-seq dataset with co-encapsulated-cell artefacts — doublet removal addresses a droplet-microfluidics-specific problem

## Inputs

- Seurat object (converted to SingleCellExperiment via creatSCE()) for each droplet scRNA-seq sample
- Estimated doublet rate per sample (~0.9% per 1,000 cells per the 10X rule of thumb)
- Optional: additional method score tables (real_data.scores.csv, simulated_data.scores.csv) for boosting extra methods

## Outputs

- CSV file 'outname_doublets.csv' listing barcodes of predicted doublets
- Per-sample doublet rate estimates (e.g., 3.81%, 4.68%, 4.74%, 4.20%, 4.42% for NSCLC samples 11, 13, 17, 18, 22)
- Per-cell-type doublet proportion cross-tabulation (e.g., myeloid 8.15%, fibroblasts 1.69%)

## How to apply

Install DoubletFinder, scds, adabag, gbm and Chord from their GitHub sources (13308204545/Chord). Estimate the doublet rate using the 10X rule of thumb of ~0.9% per 1,000 cells, then convert the input Seurat object with creatSCE() and call chord() with overkill=T: built-in methods evaluate the data, consensus 'overkilled' doublets are removed, and doublets simulated from the remaining cells generate the training set that the GBM algorithm (adabag/gbm) classifies real cells from. The overkilllist parameter controls which methods contribute to the overkill combination, and new methods can be added later by scoring the saved 'seu.robj' and 'overkilled.robj' datasets and re-running chord() with addmethods1/addmethods2 pointing to 'real_data.scores.csv' and 'simulated_data.scores.csv'. Run Chord on each sample separately, then extract doublet barcodes from 'outname_doublets.csv' and cross-tabulate them against cell-type annotations to compute per-sample and per-cell-type doublet proportions.

## Related tools

- **Chord** (Ensemble method integrating built-in doublet detectors via GBM/AdaBoost; produces final doublet calls and the 'outname_doublets.csv' barcode file) — https://github.com/13308204545/Chord
- **DoubletFinder** (Built-in doublet detection method evaluated during the overkill step; its pK parameter should be tuned per dataset) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Built-in method (cxds/bcds) providing doublet scores that feed Chord's GBM ensemble; parameters cxds.ntop (default 500), cxds.binThresh (default 0), bcds.ntop (default 500), bcds.srat (default 1)) — https://github.com/kostkalab/scds
- **adabag** (Provides the AdaBoost boosting option for integrating per-method scores)
- **gbm** (Gradient boosting machine algorithm used to integrate results of different methods into a final singlet/doublet classification)

## Examples

```
chord(seu="input seurat object", doubletrate=0.009, overkill=T, outname="sample11")
```

## Evaluation signals

- Doublet barcodes file 'outname_doublets.csv' exists and contains cell barcodes matching the input Seurat object's cell IDs
- Per-sample estimated doublet rates fall within the reported NSCLC range of 3.81–4.74% when using doubletrate ≈ 0.9% per 1,000 cells
- Per-cell-type doublet proportions reproduce reported values: myeloid cells highest at 8.15%, fibroblasts lowest at 1.69%
- Re-running chord() with addmethods1/addmethods2 score tables produces a refined 'outname2_doublets.csv' with consistent barcode counts
- Predicted doublets, when projected onto UMAP/t-SNE, should show enrichment in expected clusters (e.g., cluster-10 doublet enrichment in the NSCLC data)

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods; refactoring the integrated methods was proposed but not implemented
- Performance of individual doublet detection methods varies greatly across datasets; no method is a gold standard for every scenario
- Other pre-processing steps, such as ambient mRNA removal for droplets, are needed to improve downstream accuracy but are not incorporated into Chord
- The 'overkill' step's benefit to other methods' performance is an unverified claim with no test reported

## Evidence

- [abstract] Chord integrates multiple doublet detection methods using a machine learning algorithm: "We here propose Chord which implements a machine learning algorithm that integrates multiple doublet detection methods to address these issues."
- [discussion] Overkill workflow generates the training set: "First, built-in methods are used to evaluate the data. Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [readme] Chord invocation and doublet rate estimation: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [readme] Doublet rate rule of thumb: "10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [methods] Output file containing doublet barcodes: "The doublets' barcodes are in the file "outname_doublets.csv""
