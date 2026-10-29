---
name: exon-skipping-event-count-filtering
description: Use when preparing exon inclusion/exclusion count matrices (e.g., from
  a SingleCellExperiment with raw_M1 and raw_M2 assays) for GEDI splicing-mode integration,
  to drop low-sequencing-depth cells and low-coverage events before model fitting.
when_to_use_negative:
- Input is gene-level single-cell expression data rather than paired exon inclusion/exclusion
  read counts — the cell/event thresholds are splicing-specific
- Data is already normalized PSI estimates rather than raw inclusion/exclusion counts
  — the method models the paired counts' log-ratio directly and a naïve PSI estimate
  performs worse
- Applying to spliced/unspliced stability data — analogous but distinct filtering
  would be needed for that modality
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GEDI
  role: Downstream model that consumes the filtered paired inclusion/exclusion count
    matrices in ratio (splicing) mode
  repo: github.com/csglab/GEDI
- name: Matrix
  role: dgCMatrix conversion of the filtered M1/M2 count matrices for efficient sparse
    computation
- name: HDF5Array
  role: HDF5-backed loading of the SingleCellExperiment assays raw_M1 and raw_M2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:a894bb17f481f6931be367312d51170671bdc7581fad042d35c133cc027a0e63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GEDI
  - HDF5Array
  - Matrix
  - R
  - ggplot2
  - uwot
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# exon-skipping-event-count-filtering

## Summary

Pre-GEDI quality filtering of paired exon inclusion (M1) and exclusion (M2) count matrices from Quantas-processed single-cell splicing data, removing low-content cells and poorly covered cassette-exon events so the paired binomial log-ratio model is fit on reliable counts.

## When to use

Use when preparing exon inclusion/exclusion count matrices (e.g., from a SingleCellExperiment with raw_M1 and raw_M2 assays) for GEDI splicing-mode integration, to drop low-sequencing-depth cells and low-coverage events before model fitting.

## When NOT to use

- Input is gene-level single-cell expression data rather than paired exon inclusion/exclusion read counts — the cell/event thresholds are splicing-specific
- Data is already normalized PSI estimates rather than raw inclusion/exclusion counts — the method models the paired counts' log-ratio directly and a naïve PSI estimate performs worse
- Applying to spliced/unspliced stability data — analogous but distinct filtering would be needed for that modality

## Inputs

- SingleCellExperiment with exon inclusion counts (raw_M1) and exclusion counts (raw_M2) from the Quantas pipeline, stored HDF5-backed (e.g., './my_h5_se')
- Exon inclusion count matrix M1 (events x cells)
- Exon exclusion count matrix M2 (events x cells)

## Outputs

- Filtered dgCMatrix sparse matrices M1 and M2 covering only cells with log10(colSums) above 4.5/4 and only events with rowSums above 50
- Filtered count matrices ready for GEDI ratio-mode input M = list(M1, M2)

## How to apply

Load the HDF5-backed SingleCellExperiment containing raw_M1 (inclusion counts) and raw_M2 (exclusion counts) processed with the Quantas pipeline. Filter cells with which(log10(colSums(M1)) > 4.5 & log10(colSums(M2)) > 4), retaining cells whose log10 total inclusion and exclusion counts exceed both thresholds. Then filter events with which(rowSums(M1) > 50 & rowSums(M2) > 50), keeping only cassette-exon events with more than 50 total inclusion reads and 50 total exclusion reads across the dataset. Convert the filtered matrices to dgCMatrix sparse matrices before supplying them to GEDI as M = list(M1, M2); the rationale is that the paired binomial model of the log-ratio requires sufficient observations per cell and per event for stable latent splicing factors.

## Related tools

- **GEDI** (Downstream model that consumes the filtered paired inclusion/exclusion count matrices in ratio (splicing) mode) — github.com/csglab/GEDI
- **Matrix** (dgCMatrix conversion of the filtered M1/M2 count matrices for efficient sparse computation)
- **HDF5Array** (HDF5-backed loading of the SingleCellExperiment assays raw_M1 and raw_M2)

## Examples

```
filter <- which( rowSums(M1) > 50 & rowSums(M2) > 50 ); M1 <- as(M1[filter,], 'dgCMatrix'); M2 <- as(M2[filter,], 'dgCMatrix')
```

## Evaluation signals

- After cell filtering, both log10(colSums(M1)) and log10(colSums(M2)) exceed 4.5 and 4 respectively for all retained cells
- After event filtering, all retained events have rowSums(M1) > 50 and rowSums(M2) > 50 across the dataset
- Filtered M1 and M2 have identical cell and event dimensions and remain paired row/column-wise before constructing M = list(M1, M2)
- The filtered matrices convert cleanly to dgCMatrix and GEDI's model convergence tracking (model$plotTracking() over o, Z, Bi, Qi, oi, si and MSE) proceeds without errors

## Limitations

- Thresholds (log10 total counts > 4.5/4 per cell; > 50 reads per event) were calibrated on the Tasic 2016 vs 2018 mouse cortex datasets and may need retuning for other splicing datasets
- GEDI currently supports only linear latent manifold learning for the integrated splicing space, so nonlinear structure in the splicing space is not captured

## Evidence

- [methods] Cell filtering thresholds: "filter<- which(log10(colSums(M1)) > 4.5 & log10(colSums(M2)) > 4)"
- [methods] Event filtering thresholds: "filter <- which( rowSums(M1) > 50 & rowSums(M2) > 50 ) # Filter events"
- [abstract] Ratio-based modality modeling: "joint analysis of exon inclusion/exclusion reads to model alternative cassette exon splicing, or spliced/unspliced reads to model the mRNA stability"
- [readme] Splicing vignette in GEDI README: [evidence span withheld — anchor status: external_doc]
