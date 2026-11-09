---
name: pseudotime-benchmarking-on-real-datasets
description: Use when a scRNA-seq dataset has a known number of branches and cell-type
  ordering (e.g., CMP GSE72857 with eight branches, HSMM with two, MEF with three)
  and you must choose which TI method to trust before pseudotime-based downstream
  analysis.
when_to_use_negative:
- The dataset has no independently known branch structure or cell-type ordering to
  benchmark against, so per-method comparison is meaningless
- A single TI method is already fixed downstream (scTITANS uses Monocle3 for pseudotime)
  and no method-selection decision is needed
- The input is bulk time-series data rather than single-cell RNA-seq, where trajectory
  inference per cell is not applicable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Monocle
  role: One of the ten benchmarked TI methods applied to the normalized CMP matrix
- name: Monocle2
  role: One of the ten benchmarked TI methods applied to the normalized CMP matrix
- name: Monocle3
  role: Benchmarked TI method; also the method scTITANS uses to construct trajectories
    and calculate per-cell pseudotime
- name: TSCAN
  role: One of the ten benchmarked TI methods
- name: SLICER
  role: One of the ten benchmarked TI methods
- name: GPfates
  role: One of the ten benchmarked TI methods (reconstructed HSMM branches but with
    a much less reasonable trajectory)
- name: Slingshot
  role: One of the ten benchmarked TI methods
- name: Destiny
  role: One of the ten benchmarked TI methods
- name: Mpath
  role: Benchmarked TI method; only method that accurately reconstructed the eight
    CMP branches, but provides no per-cell trajectory so is unusable for asynchrony
    correction
- name: STEMNET
  role: One of the ten benchmarked TI methods
- name: scran
  role: Normalizes the raw UMI matrix with the pool-and-deconvolute algorithm before
    TI benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:af8a51415ed2e5c14230614b6802c767330d211899ef0dcf7e78487ef8b8157f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Destiny
  - GPfates
  - Monocle
  - Monocle2
  - Monocle3
  - Mpath
  - SLICER
  - STEMNET
  - Slingshot
  - TSCAN
  - scTITANS
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pseudotime-benchmarking-on-real-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark ten trajectory inference (TI) methods (Monocle, Monocle2, Monocle3, TSCAN, SLICER, GPfates, Slingshot, Destiny, Mpath, STEMNET) on scRNA-seq datasets with known branching topology to select the method whose reconstructed branches best match the reference cell-type ordering.

## When to use

Use when a scRNA-seq dataset has a known number of branches and cell-type ordering (e.g., CMP GSE72857 with eight branches, HSMM with two, MEF with three) and you must choose which TI method to trust before pseudotime-based downstream analysis.

## When NOT to use

- The dataset has no independently known branch structure or cell-type ordering to benchmark against, so per-method comparison is meaningless
- A single TI method is already fixed downstream (scTITANS uses Monocle3 for pseudotime) and no method-selection decision is needed
- The input is bulk time-series data rather than single-cell RNA-seq, where trajectory inference per cell is not applicable

## Inputs

- Raw UMI gene-by-cell count matrix (e.g., GSE72857 CMP: 2730 cells, 19 cell types, eight branches)
- GEO accession / raw digital gene expression matrix (e.g., from Cell Ranger or MARS-seq)
- Known reference topology: number of branches and cell-type ordering per dataset

## Outputs

- Per-method trajectory reconstructions from the ten TI methods
- Comparison table of branch-reconstruction success/failure (e.g., cmp_ti_comparison.csv)

## How to apply

Download the dataset from NCBI GEO (e.g., GSE72857: 2730 mouse myeloid progenitor cells, 19 cell types, eight branches) and preprocess the raw UMI matrix: keep genes detected (two or more transcripts) in at least 10 cells, discard cells with gene counts outside the 5th and 95th percentile or with more than 10% mitochondrial UMIs, then normalize with the scran pool-and-deconvolute algorithm. Apply each of the ten TI methods to the same normalized matrix and compare the number and topology of reconstructed branches against the known branch structure and cell-type ordering. Record per-method success/failure in a comparison table (e.g., cmp_ti_comparison.csv) and cross-check against published figures (Figure S1a-c of the scTITANS paper). When selecting a method for time-series analysis, remember that accurate branch reconstruction alone is insufficient: Mpath best reconstructed the CMP branches but does not provide a per-cell trajectory, so it cannot correct single-cell asynchrony — a Monocle3-based pseudotime (as used by scTITANS) is needed instead.

## Related tools

- **Monocle** (One of the ten benchmarked TI methods applied to the normalized CMP matrix)
- **Monocle2** (One of the ten benchmarked TI methods applied to the normalized CMP matrix)
- **Monocle3** (Benchmarked TI method; also the method scTITANS uses to construct trajectories and calculate per-cell pseudotime)
- **TSCAN** (One of the ten benchmarked TI methods)
- **SLICER** (One of the ten benchmarked TI methods)
- **GPfates** (One of the ten benchmarked TI methods (reconstructed HSMM branches but with a much less reasonable trajectory))
- **Slingshot** (One of the ten benchmarked TI methods)
- **Destiny** (One of the ten benchmarked TI methods)
- **Mpath** (Benchmarked TI method; only method that accurately reconstructed the eight CMP branches, but provides no per-cell trajectory so is unusable for asynchrony correction)
- **STEMNET** (One of the ten benchmarked TI methods)
- **scran** (Normalizes the raw UMI matrix with the pool-and-deconvolute algorithm before TI benchmarking)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- The comparison table must show Mpath as the only method that accurately reconstructs all eight CMP branches (consistent with Figure S1c)
- For HSMM (GSE52529), the table should show only Monocle, Monocle3, GPfates, and Mpath reconstructing the two branches (Figure S1a)
- For MEF (GSE67310), the table should show only Monocle3 reconstructing the three branches (Figure S1b)
- Reconstructed branch counts and cell-type ordering per method match the known reference topology of each dataset
- Per-cell pseudotime values are available from the selected method (required for downstream asynchrony correction, disqualifying Mpath)

## Limitations

- Mpath accurately reconstructs branch topology but does not provide a trajectory for each single cell, reducing its value in correcting single-cell asynchrony for time-series analysis
- Software version numbers for Monocle3, scran, scTITANS and the other TI tools are not stated in the article, complicating reproducibility of the benchmark
- GPfates' HSMM trajectory was apparently much less reasonable than the other methods that reconstructed the two branches, so branch count alone is not a sufficient success criterion

## Evidence

- [results] Although Mpath best reconstructed the CMP branches, Mpath did not provide a trajectory for each single cell, which reduces its value in correcting the asynchrony: [evidence span withheld — anchor status: not_found]
