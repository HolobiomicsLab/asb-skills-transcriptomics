---
name: summarizedexperiment-data-access
description: Use when a dataset is loaded as a SummarizedExperiment object and you
  need the gene-by-cell count matrix (assays(x)$count) plus per-cell metadata such
  as time.table or sample_alias to run temporal single-cell analyses like TCM.
when_to_use_negative:
- Input is already a plain matrix or data.frame of counts with a separate metadata
  table — accessors like assays()$count and colData()$time.table do not apply
- Data comes from sim.rnaseq.ts simulation — although it also exposes assays(sim)$count
  and colData(sim)$time.table, no scDatasets loading step is needed
- The analysis requires cell-level quantifications (e.g. TPM, normalized logcounts)
  rather than the raw count slot stored in assays()
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SummarizedExperiment
  role: Provides the assays() and colData() accessors used to pull count matrices
    and cell metadata out of scDatasets objects
- name: scDatasets
  role: Supplies the packaged SummarizedExperiment datasets (guo, guo2, loh, trapnell)
    loaded via data() before accessor extraction
- name: tcm
  role: Downstream consumer of the extracted count matrix and time.table for TCM model
    fitting
  repo: gongx030/tcm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:930218fa8878da06d84e10f532ec3f3d8614ee8ffb3493c01d779ca59b9af46a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - SummarizedExperiment
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# summarizedexperiment-data-access

## Summary

Extract count matrices and cell-level metadata (time index, sample labels) from scDatasets SummarizedExperiment objects (e.g. loh, guo, guo2, trapnell) using the assays() and colData() accessors before feeding them into the tcm pipeline.

## When to use

Use when a dataset is loaded as a SummarizedExperiment object and you need the gene-by-cell count matrix (assays(x)$count) plus per-cell metadata such as time.table or sample_alias to run temporal single-cell analyses like TCM.

## When NOT to use

- Input is already a plain matrix or data.frame of counts with a separate metadata table — accessors like assays()$count and colData()$time.table do not apply
- Data comes from sim.rnaseq.ts simulation — although it also exposes assays(sim)$count and colData(sim)$time.table, no scDatasets loading step is needed
- The analysis requires cell-level quantifications (e.g. TPM, normalized logcounts) rather than the raw count slot stored in assays()

## Inputs

- SummarizedExperiment object from the scDatasets package (e.g. loh, guo, guo2, trapnell)
- assays()$count slot: gene-by-cell raw read count matrix
- colData()$time.table slot: cell-by-time-point binary matrix
- colData() metadata columns such as sample_alias, lineage, time, source.name

## Outputs

- Gene-by-cell count matrix X ready for tcm::preprocess(X)
- Per-cell time index (time.table binary matrix) for tcm(X, time.table = time.table)
- Per-cell label/color vector derived from colData columns

## How to apply

Load the scDatasets package and the target data object (e.g. data(loh)); extract the gene-by-cell read count matrix with assays(loh)$count and the cell-by-time-point binary matrix with colData(loh)$time.table. For per-cell coloring labels, pull a metadata column via colData(loh)[['sample_alias']] and map it through a named vector (e.g. lab2name mapping H7hESC to hESC(D0), H7_derived_MPS to Mid PS(D1), H7_derived_D3GARPpCrdcM to Cardiac Mesoderm(D3-D4)). The time.table is a cell by time point binary matrix indicating the timestamp of each cell and is passed directly to tcm(X, time.table = time.table) after preprocessing. This accessor pattern is identical across the guo, guo2, loh, and trapnell datasets in the README case studies.

## Related tools

- **SummarizedExperiment** (Provides the assays() and colData() accessors used to pull count matrices and cell metadata out of scDatasets objects)
- **scDatasets** (Supplies the packaged SummarizedExperiment datasets (guo, guo2, loh, trapnell) loaded via data() before accessor extraction)
- **tcm** (Downstream consumer of the extracted count matrix and time.table for TCM model fitting) — gongx030/tcm

## Examples

```
library(scDatasets); library(SummarizedExperiment); data(loh); X <- assays(loh)$count; time.table <- colData(loh)$time.table
```

## Evaluation signals

- assays(loh)$count returns a genes-by-cells matrix whose columns match the rows of colData(loh)$time.table (one row per loh cell)
- colData(loh)$time.table is a binary cell-by-time-point matrix (e.g. 500 by 5 structure in the simulated case) covering every cell
- colData(loh)[['sample_alias']] values all map through the lab2name named vector with no NAs after factor(cnv, names(lab2name), lab2name)
- The extracted X passes tcm::preprocess(X) without dimension mismatch errors when paired with time.table

## Limitations

- The accessor pattern assumes the object's metadata column names (e.g. sample_alias, time.table) match those documented in scDatasets; other datasets may use different column names
- TCM requires the time index to be complete and correlated with the underlying dynamic expression pattern, so a malformed or partial time.table extracted via colData() will undermine the model

## Evidence

- [readme] X <- assays(loh)$count; time.table <- colData(loh)$time.table: "data(loh)
X <- assays(loh)$count
time.table <- colData(loh)$time.table"
- [readme] Note that X is the simulated gene by cell read count matrix (2,000 by 500), and time.table is a cell by time point binary matrix (500 by 5), indicating the timestamp of each cell.: "Note that X is the simulated gene by cell read count matrix (2,000 by 500), and time.table is a cell by time point binary matrix (500 by 5), indicating the timestamp of each cell."
- [methods] library(tcm); library(SummarizedExperiment): "library(tcm)
library(SummarizedExperiment)"
- [methods] library(scDatasets); data(guo): "library(scDatasets)
data(guo)"
