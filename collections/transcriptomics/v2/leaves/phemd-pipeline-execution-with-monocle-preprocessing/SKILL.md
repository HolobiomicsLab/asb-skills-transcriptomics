---
name: phemd-pipeline-execution-with-monocle-preprocessing
description: Use when benchmarking sample-level single-cell distance methods on multi-sample
  scRNA-seq datasets (e.g., .h5ad patient cohorts) and you need the PhEMD baseline
  with its required Monocle 2 preprocessing and measured runtime.
when_to_use_negative:
- 'When the goal is unsupervised sample-level analysis rather than benchmarking: the
  paper notes the latter methods ''require labels of patients for their analysis and
  cannot be used in the unsupervised analysis'''
- When you only need patient-level distances and runtime matters — PILOT via POT's
  emd2 completed the 1.26M-cell Lupus dataset in 58.12 s vs PhEMD's 16.50 min
- When the data are already aggregated per sample (pseudo-bulk proportions); the pseudo-bulk
  baseline applies directly instead of the full PhEMD pipeline
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: PhEMD
  role: Computes earth moving distance (EMD) between specimens (single-cell samples)
    as the competing baseline being executed and timed
- name: Monocle 2
  role: 'Required preprocessing stage inside the PhEMD pipeline: normalization, dimension
    reduction, and clustering of cells'
- name: Seurat
  role: Used in the uniform pre-processing pipeline for the normalization and clustering
    of structures across benchmarked methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:2c327da696b605936e16165f80f8947bf8b0e51f2824065a9f8da65a496a71b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PILOT
  - POT library
  - PhEMD
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phemd-pipeline-execution-with-monocle-preprocessing

## Summary

Executes PhEMD end-to-end as a competing sample-level baseline: Monocle 2 performs normalization, dimension reduction, and cell clustering, after which PhEMD computes earth moving distances (EMD) between specimens. Used in the PILOT paper to benchmark wall-clock runtime against PILOT and a pseudo-bulk baseline on the ~1.26M-cell Lupus PBMC dataset.

## When to use

Use when benchmarking sample-level single-cell distance methods on multi-sample scRNA-seq datasets (e.g., .h5ad patient cohorts) and you need the PhEMD baseline with its required Monocle 2 preprocessing and measured runtime.

## When NOT to use

- When the goal is unsupervised sample-level analysis rather than benchmarking: the paper notes the latter methods 'require labels of patients for their analysis and cannot be used in the unsupervised analysis'
- When you only need patient-level distances and runtime matters — PILOT via POT's emd2 completed the 1.26M-cell Lupus dataset in 58.12 s vs PhEMD's 16.50 min
- When the data are already aggregated per sample (pseudo-bulk proportions); the pseudo-bulk baseline applies directly instead of the full PhEMD pipeline

## Inputs

- Multi-sample single-cell RNA-seq cohort in Anndata .h5ad format (e.g., GSE174188_CLUES1_adjusted.h5ad.gz from GEO accession GSE174188)
- Sample/patient labels mapping cells to specimens for pairwise EMD computation

## Outputs

- Specimen-level EMD distance matrix from PhEMD
- Measured PhEMD wall-clock runtime row for the benchmark table (e.g., lupus_runtime_benchmark.csv)

## How to apply

Run PhEMD end-to-end on the same input cohort as the competing methods — for the PILOT paper this is the Lupus PBMC Anndata file GSE174188_CLUES1_adjusted.h5ad.gz (~1,263,676 cells) loaded with scanpy. PhEMD requires Monocle 2 (Trapnell et al, 2014) to perform normalization, dimension reduction, and clustering of cells; wire Monocle 2 as the preprocessing stage before EMD computation. PhEMD then measures the earth moving distance (EMD) between specimens (single-cell samples), yielding a sample-by-sample distance matrix. Time the full run with Python's time module and record the wall-clock value alongside the PILOT run (which uses emd2 from the POT library) and the pseudo-bulk baseline (which aggregates cell-type proportions per patient before computing sample-level distances). Assemble the measured runtimes into a (method, runtime) table (e.g., lupus_runtime_benchmark.csv) and compare each entry to the values reported in the Results section.

## Related tools

- **PhEMD** (Computes earth moving distance (EMD) between specimens (single-cell samples) as the competing baseline being executed and timed)
- **Monocle 2** (Required preprocessing stage inside the PhEMD pipeline: normalization, dimension reduction, and clustering of cells)
- **Seurat** (Used in the uniform pre-processing pipeline for the normalization and clustering of structures across benchmarked methods)

## Evaluation signals

- The benchmark table contains one runtime row for each of PILOT, pseudo-bulk, and PhEMD for the 1.26M-cell Lupus dataset
- Measured PhEMD runtime falls within the same order of magnitude as the reported 16.50 min for the 1.26M-cell Lupus PBMC data
- Monocle 2 was demonstrably used for normalization, dimension reduction, and clustering of cells before EMD computation (a PhEMD requirement)
- The output is a pairwise sample-level distance matrix with one EMD entry per specimen pair
- Pseudo-bulk baseline time (reported 9.03 s) and PILOT time (reported 58.12 s) are recorded on the identical input for a fair comparison

## Limitations

- PhEMD scales poorly to millions of cells: 16.50 min vs PILOT's 58.12 s on the 1,263,676-cell Lupus PBMC dataset
- PhEMD depends on the Monocle 2 toolchain for normalization, dimension reduction, and clustering, adding preprocessing overhead
- Like other competing approaches (Ravindra, SCANCell, Multi-scale PHATE), methods in this family may require patient labels, limiting unsupervised analysis

## Evidence

- [discussion] PhEMD required the use of Monocle 2 (Trapnell et al, 2014) to perform normalization, dimension reduction, and clustering of cells.: "PhEMD required the use of Monocle 2 (Trapnell et al, 2014) to perform normalization, dimension reduction, and clustering of cells."
- [intro] PhEMD is based on earth moving distance (EMD) to measure the distance between specimens (single-cell samples): "PhEMD (Chen et al, 2020) is based on earth moving distance (EMD) to measure the distance between specimens (single-cell samples)"
- [intro] The latter methods (Kuchroo et al, 2022; Ravindra et al, 2020; Zhang et al, 2022) require labels of patients for their analysis and cannot be used in the unsupervised analysis: "The latter methods (Kuchroo et al, 2022; Ravindra et al, 2020; Zhang et al, 2022) require labels of patients for their analysis and cannot be used in the unsupervised analysis"
- [discussion] PILOT is based on the emd2 function from POT library (Flamary et al, 2021): "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] We employed a uniform pre-processing pipeline utilizing Seurat (Hao et al, 2021) for the normalization and clustering of structures.: "We employed a uniform pre-processing pipeline utilizing Seurat (Hao et al, 2021) for the normalization and clustering of structures."
