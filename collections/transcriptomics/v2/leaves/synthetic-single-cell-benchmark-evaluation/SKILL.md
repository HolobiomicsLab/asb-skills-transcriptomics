---
name: synthetic-single-cell-benchmark-evaluation
description: Use when a scRNA-seq dataset has simulated/labelled doublets and known
  simulated-DEG labels, and you need to rank doublet-detection or removal methods
  by downstream DEG accuracy rather than doublet-call metrics alone.
when_to_use_negative:
- The dataset contains no labelled/simulated doublets or known DEG ground truth, so
  accuracy relative to clean data cannot be computed (use doublet-call metrics instead).
- The goal is doublet detection on real experimental data rather than benchmarking
  method accuracy on a synthetic contamination scenario.
- Input is ambient-RNA-contaminated droplet data where other pre-processing (e.g.
  ambient mRNA removal) is the dominant error source not addressed by this evaluation.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_6170
- http://edamontology.org/topic_0203
tools:
- name: Chord
  role: Ensemble GBM-based doublet detection run with overkill=T; its filtered dataset
    is compared against individual methods in the DEG-accuracy benchmark
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Individual competing doublet-detection method run with tuned pK; top-ranked
    doublets removed for comparison
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Provides cxds/bcds doublet scores used both individually and as components
    integrated by Chord's GBM ensemble
  repo: https://github.com/kostkalab/scds
- name: MAST
  role: Differential expression testing framework used alongside Wilcoxon rank-sum
    to produce DEG calls for accuracy evaluation
- name: gbm
  role: Gradient boosting machine algorithm Chord uses to integrate results of different
    detection methods
- name: adabag
  role: Alternative boosting method (adaboost) available in Chord via the method parameter
- name: remotes
  role: Used to install DoubletFinder and Chord from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:4d73c57dc512a51a3e49364f64fd5c5aef2db9ffef559d440ec5ed96672b36e0
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_6170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# synthetic-single-cell-benchmark-evaluation

## Summary

Benchmark doublet-removal methods on a synthetic doublet-contaminated scRNA-seq dataset (DataDEG, 40% labelled doublets) by measuring how well DEG detection accuracy after each method's filtering approaches the clean-data reference, using TPR, TNR, and accuracy against simulated-DEG ground truth.

## When to use

Use when a scRNA-seq dataset has simulated/labelled doublets and known simulated-DEG labels, and you need to rank doublet-detection or removal methods by downstream DEG accuracy rather than doublet-call metrics alone.

## When NOT to use

- The dataset contains no labelled/simulated doublets or known DEG ground truth, so accuracy relative to clean data cannot be computed (use doublet-call metrics instead).
- The goal is doublet detection on real experimental data rather than benchmarking method accuracy on a synthetic contamination scenario.
- Input is ambient-RNA-contaminated droplet data where other pre-processing (e.g. ambient mRNA removal) is the dominant error source not addressed by this evaluation.

## Inputs

- Seurat object of the synthetic DataDEG dataset (scDesign simulation, 1,667 cells, 40% labelled doublets) from the Xi & Li 2020 benchmarking deposit
- Known simulated-DEG labels (ground truth for DEG detection accuracy)
- Clean-data (no-doublet) reference DEG results

## Outputs

- Filtered SingleCellExperiment/Seurat objects per doublet-removal method
- Doublet barcode lists per method (e.g. outname_doublets.csv from Chord)
- Table of TPR, TNR, and accuracy of DEG calls per method (Wilcoxon rank-sum and MAST) versus the clean-data reference

## How to apply

Load the scDesign-simulated DataDEG dataset (1,667 cells, 40% labelled doublets) from the Xi & Li 2020 benchmarking deposit and convert the Seurat object to a SingleCellExperiment with creatSCE(). For each candidate method, remove predicted doublets: run chord() with overkill=T (GBM ensemble of scds cxds/bcds and DoubletFinder, default pN=0.25) and retrieve barcodes from outname_doublets.csv; for individual methods run DoubletFinder with tuned pK and scds cxds/bcds, removing the top 40% of cells ranked by each doublet score. Rerun differential expression on every filtered dataset with Wilcoxon rank-sum and MAST. Compute TPR, TNR, and accuracy of the DEG calls against the known simulated-DEG labels, and compare each method's values to the clean-data (no-doublet) reference. The rationale is that heterotypic doublets distort DE analysis, so a method that recovers DEG results closest to the clean reference is the most accurate remover.

## Related tools

- **Chord** (Ensemble GBM-based doublet detection run with overkill=T; its filtered dataset is compared against individual methods in the DEG-accuracy benchmark) — https://github.com/13308204545/Chord
- **DoubletFinder** (Individual competing doublet-detection method run with tuned pK; top-ranked doublets removed for comparison) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Provides cxds/bcds doublet scores used both individually and as components integrated by Chord's GBM ensemble) — https://github.com/kostkalab/scds
- **MAST** (Differential expression testing framework used alongside Wilcoxon rank-sum to produce DEG calls for accuracy evaluation)
- **gbm** (Gradient boosting machine algorithm Chord uses to integrate results of different detection methods)
- **adabag** (Alternative boosting method (adaboost) available in Chord via the method parameter)
- **remotes** (Used to install DoubletFinder and Chord from GitHub)

## Examples

```
chord(seu="input seurat object", doubletrate="estimated doubletrate", overkill=T, outname="DataDEG")
```

## Evaluation signals

- The output table reproduces the reported pattern: Chord's filtered dataset yields TPR/TNR/accuracy values closest to the clean-data (no-doublet) reference results.
- DEG analysis results after doublet removal are better than on the contaminated 40%-doublet data for every method evaluated.
- Chord, DoubletFinder, and DoubletDetection results are closer to the clean-data results than other individual approaches on DataDEG.
- Doublet barcodes file (outname_doublets.csv) is produced for each chord() run and matches the expected number of removed cells for the chosen doubletrate.
- DE testing is run with both Wilcoxon rank-sum and MAST, and both show concordant relative method rankings.

## Limitations

- Chord is not optimal in time efficiency because it must integrate the results of multiple methods.
- Performance of existing computational doublet detection methods varies greatly; no method is a gold standard for every scenario.
- Other pre-processing steps such as ambient mRNA removal for droplets should be considered to improve downstream analysis accuracy but are not incorporated here.
- The estimated doubletrate depends on cell number (10X guidance: ~0.9% per 1,000 cells), so mis-estimated doublet rates shift how many cells are removed.

## Evidence

- [abstract] Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data.: "Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data."
- [intro] heterotypic doublets impact differential expression: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories"
- [methods] chord function signature: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [readme] doublet barcodes output: "The doublets' barcodes are in the file "outname_doublets.csv""
- [readme] dbf.pN default rationale: "Default is set to 0.25, based on observation that DoubletFinder performance is largely pN-invariant (see McGinnis, Murrow and Gartner 2019, Cell Systems)."
