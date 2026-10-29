---
name: scrnaseq-data-input-format-preparation
description: Use when scRNA-seq data with cell-type and time-point metadata must be
  formatted into the expression/cell-type/time-point files that dsCellNet requires
  before running datainput and downstream network inference.
when_to_use_negative:
- The analysis goal is single-time-point cell–cell communication without developmental
  series information — other methods (CellPhoneDB, NATMI, SingleCellSignalR) may suffice
- Input data lacks cell-type labels or time-point information needed by dsCellNet's
  three-file input interface
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: dsCellNet
  role: R package whose datainput function ingests the three formatted input files
    and returns expression, cell-type, and time-point data frames for network inference
  repo: https://github.com/songzh523/dsCellNet
- name: R
  role: Runtime environment (>=4.0.3) in which dsCellNet is installed from source
    and input preparation is performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:d3c498f46c64fb92fb4210aa229928ce7a8330616bb5725efe86a77c01ff4dc4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  repo_url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# scrnaseq-data-input-format-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare scRNA-seq input files (expression matrix, cell-type labels, and time-point information) in the three-file layout required by dsCellNet's datainput interface so developmental-series cell–cell interaction networks can be inferred.

## When to use

Use when scRNA-seq data with cell-type and time-point metadata must be formatted into the expression/cell-type/time-point files that dsCellNet requires before running datainput and downstream network inference.

## When NOT to use

- The analysis goal is single-time-point cell–cell communication without developmental series information — other methods (CellPhoneDB, NATMI, SingleCellSignalR) may suffice
- Input data lacks cell-type labels or time-point information needed by dsCellNet's three-file input interface

## Inputs

- gene expression matrix file (e.g., scRNA-seq counts from GSE153164 E15.5 mouse embryonic cerebral cortex)
- cell-type label file matching cells in the expression matrix
- time-point file matching cells in the expression matrix (optional input)
- datloc: directory path of the input files

## Outputs

- inpexp: expression data frame returned by datainput
- inptyp: cell-type data frame with column Type
- inptim: time-point data frame with column Time

## How to apply

Assemble three tabular files: a gene expression matrix, a cell-type label file, and a time-point file, then point the datainput function at their directory (datloc) and filenames (expnam, typnam, timnam). Load the returned objects as data frames — inpexp, inptyp, and inptim — preserving row names and column names as shown in the README example, since downstream functions like definecolo and singlenet consume inptyp$Type and inptim$Time. If cells need to be reduced to a fixed size for benchmarking (as in the E15.5 runtime test), randomly subsample to a target count (e.g., 10,000 cells from the 11,670-cell GSE153164 dataset) with a fixed seed for reproducibility. Verify that expression, cell-type, and time-point files are aligned cell-by-cell before invoking datainput.

## Related tools

- **dsCellNet** (R package whose datainput function ingests the three formatted input files and returns expression, cell-type, and time-point data frames for network inference) — https://github.com/songzh523/dsCellNet
- **R** (Runtime environment (>=4.0.3) in which dsCellNet is installed from source and input preparation is performed)

## Examples

```
out<-datainput(datloc,expnam,typnam,timnam,"FALSE",resloc); inpexp<-as.data.frame(out$exp); inptyp<-as.data.frame(out$typ); inptim<-as.data.frame(out$tim)
```

## Evaluation signals

- datainput runs without error and returns non-empty inpexp, inptyp, and inptim data frames
- Row names of inpexp, inptyp, and inptim are cell-aligned across the three files
- inptyp has a 'Type' column and inptim has a 'Time' column usable by definecolo and singlenet
- If subsampling, each replicate contains exactly the target cell count (e.g., 10,000 cells) and is reproducible from the fixed seed

## Limitations

- The time-point file is optional, so single-time-point datasets may not exploit dsCellNet's developmental-series DTW and fuzzy clustering features
- dsCellNet's default L-R gene list size (5,744 interactions) limits detection of some interactions; users can supply supplementary L-R genes
- dsCellNet version number is not stated in the article

## Evidence
