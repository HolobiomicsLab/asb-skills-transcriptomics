---
name: simulated-scrnaseq-cell-type-benchmarking
description: Use when a simulated or pre-labeled scRNA-seq count matrix plus cell-type
  marker list are available and the goal is to quantitatively rank multiple cell-type
  identification methods by accuracy and F1-score.
when_to_use_negative:
- The input is raw sequencing data (FASTQ) without a count matrix — this skill operates
  on count matrices and pre-computed labels, not alignment/preprocessing.
- Cells have no known or simulated cell-type labels and no marker signatures are available
  — accuracy and F1 cannot be computed, and marker-based methods like SCINA cannot
  run.
- The goal is annotating a single new dataset with one chosen method rather than comparing
  multiple methods' performance.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper used to run eager (eagersupervised), lazy (lazysupervised) and marker
    (markersupervised) cell-type identification methods uniformly on the same train/test
    inputs
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Unsupervised baseline pipeline for clustering and finding cell type specific
    markers used to annotate clusters
- name: SCINA
  role: Marker-learning baseline run on the same simulated counts with the same marker
    signatures via markersupervised(test,marker,method='SCINA')
- name: scmap-cluster
  role: Eager-learning baseline for cluster-level cell-type prediction compared against
    marker-based methods
  repo: https://github.com/hemberg-lab/scmap
- name: R
  role: Implementation environment for the benchmarking workflow and metric computation
    (with the caret package for data splits)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:09f17e05a16b549e7f410c190f87c5f9c20a64c4a0469bae8ce8efb04ba1668a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  repo_url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  tools:
  - AutomaticCellTypeIdentification
  - CellAssign
  - R
  - SCINA
  - Seurat
  - scmap-cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# simulated-scrnaseq-cell-type-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark automatic cell-type identification methods (eager/lazy supervised, marker-based) on simulated scRNA-seq count matrices by running each method on the same counts and markers, then comparing accuracy and mean per-cell-type F1-score in a ranked table.

## When to use

Use when a simulated or pre-labeled scRNA-seq count matrix plus cell-type marker list are available and the goal is to quantitatively rank multiple cell-type identification methods by accuracy and F1-score.

## When NOT to use

- The input is raw sequencing data (FASTQ) without a count matrix — this skill operates on count matrices and pre-computed labels, not alignment/preprocessing.
- Cells have no known or simulated cell-type labels and no marker signatures are available — accuracy and F1 cannot be computed, and marker-based methods like SCINA cannot run.
- The goal is annotating a single new dataset with one chosen method rather than comparing multiple methods' performance.

## Inputs

- simulated scRNA-seq count matrix (rows = genes, columns = cells)
- ground-truth cell-type labels for each cell
- cell-type marker list / marker signatures (for marker-based methods such as SCINA)
- training and testing count matrices plus training labels for eager/lazy supervised methods

## Outputs

- per-method comparison table (e.g. cellassign_comparison_results.csv) with accuracy and mean per-cell-type F1-score
- ranked method ordering by accuracy and F1-score
- optional per-method metrics: unlabeling rate, running time, sensitivity, specificity

## How to apply

Prepare the count matrix (rows = genes, columns = cells) with known labels and remove cells lacking cell-type information; the article's preprocessing also filters genes not expressed in any cells and cells with <200 expressed genes, >2 SD above the mean expressed genes, or >20% mitochondrial expression. Run each candidate method on the identical input: unsupervised baselines (e.g. Seurat clustering) with marker-based cluster annotation, eager-learning methods (e.g. scmap-cluster, correlation-based prediction), and marker-learning methods such as SCINA given the same marker signatures; the AutomaticCellTypeIdentification R package wraps these as eagersupervised(train,test,label_train,method=...), lazysupervised(...) and markersupervised(test,marker,method=...). For cross-validated splits, use caret's createDataPartition and createFolds (four subsets for training, one for testing, five rounds). Compute accuracy as correctly predicted cells divided by total labeled cells (excluding unlabeled) and mean per-cell-type F1-score, plus unlabeling rate and running time as in the Overall Performance Score. Rank methods and write the comparison table (e.g. cellassign_comparison_results.csv), validating that every method has numeric accuracy and F1-score columns and that the ordering matches the study's reported ranking.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper used to run eager (eagersupervised), lazy (lazysupervised) and marker (markersupervised) cell-type identification methods uniformly on the same train/test inputs) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Unsupervised baseline pipeline for clustering and finding cell type specific markers used to annotate clusters)
- **SCINA** (Marker-learning baseline run on the same simulated counts with the same marker signatures via markersupervised(test,marker,method='SCINA'))
- **scmap-cluster** (Eager-learning baseline for cluster-level cell-type prediction compared against marker-based methods) — https://github.com/hemberg-lab/scmap
- **R** (Implementation environment for the benchmarking workflow and metric computation (with the caret package for data splits))

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- The output CSV contains a row for every evaluated method with numeric accuracy and F1-score columns.
- Accuracy equals correctly predicted cells divided by total labeled cells, with unlabeled predictions excluded.
- F1-score is the harmonic mean of precision and recall averaged over cell types.
- The resulting ranking is consistent with the relative ordering of methods reported in the source study (e.g. the marker-based method outperforming unsupervised, eager-learning and other marker-learning baselines).
- All methods were run on the identical count matrix and marker list, so metric differences are attributable to the methods, not the inputs.

## Limitations

- Metrics depend on the simulated data setup; rankings may not transfer to real tissues or different cell-type compositions.
- Methods may leave cells unlabeled, which affects accuracy calculations and requires tracking the unlabeling rate.
- The Python-based wrapped methods require loading the corresponding conda environment before invoking them from R.

## Evidence

- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
- [readme] markersupervised methods include scTyper, Markercount, SCSA, DigitalCellSorter and SCINA.: [evidence span withheld — anchor status: not_found]
