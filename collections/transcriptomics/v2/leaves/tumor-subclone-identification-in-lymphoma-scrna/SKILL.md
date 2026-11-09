---
name: tumor-subclone-identification-in-lymphoma-scrna
description: Use when a multi-timepoint (e.g., pre- vs post-treatment) scRNA-seq dataset
  such as MCL specimens has t0 cell labels transferred via CrossTimeAnnotation and
  you need to flag emergent tumor subclones among t1 cells.
when_to_use_negative:
- The novel cells of interest are transcriptomically very similar to known cell types
  — the article states the novel cell detection framework may not work well in that
  case
- The t0 (initial timepoint) data differs dramatically from later timepoints, since
  the neural network would fail to borrow useful information and train poorly
- The task is single-timepoint analysis with no prior labeled timepoint to transfer
  labels from
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: Provides CreateCASiList and DetectNovelCells (correlation + t-test novel-cell
    detection on a CASiList)
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: UMAP visualization of cell population evolution and the correlation-defined
    tumor-cell clusters
- name: Keras
  role: Neural-network backend for CrossTimeAnnotation whose labels precede novel-cell
    detection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:24983268fed44647b7a9ac53d4b4e7531d3fad0d63360f5385cbbd82337c1744
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMLU
  - CASi
  - CHETAH
  - Keras
  - Seurat
  - scmap
  - singleR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# tumor-subclone-identification-in-lymphoma-scrna

## Summary

Detects emergent tumor subclones in longitudinal lymphoma scRNA-seq by flagging cells whose correlation to known tumor-cell profiles falls at or below a cutoff (corr.cutoff=0.4) with a significant t-test (p < 0.01), using CASi's DetectNovelCells on an annotated CASiList.

## When to use

Use when a multi-timepoint (e.g., pre- vs post-treatment) scRNA-seq dataset such as MCL specimens has t0 cell labels transferred via CrossTimeAnnotation and you need to flag emergent tumor subclones among t1 cells.

## When NOT to use

- The novel cells of interest are transcriptomically very similar to known cell types — the article states the novel cell detection framework may not work well in that case
- The t0 (initial timepoint) data differs dramatically from later timepoints, since the neural network would fail to borrow useful information and train poorly
- The task is single-timepoint analysis with no prior labeled timepoint to transfer labels from

## Inputs

- t0 gene expression count matrix with cell labels (initial timepoint)
- t1 gene expression count matrix (follow-up timepoint)
- CASiList object built via CreateCASiList with default nfeature=2000 top highly variable genes
- Group (pre/post treatment) and Time metadata for specimens

## Outputs

- Novel-cell flags on t1 cells (correlation <= 0.4, t-test p < 0.01)
- UMAP visualization separating emergent tumor subclones (two correlation-defined tumor-cell clusters in MCL)
- Cluster summary; optionally accuracy, ARI, and cluster summary written to casi_mcl_results.csv

## How to apply

Load the longitudinal scRNA-seq data (e.g., 21 MCL specimens split into t0 and t1 count matrices with Group pre/post treatment metadata) and build a CASiList with CreateCASiList(t0_count, t1_count, t0_label) using the default nfeature=2000 highly variable genes. Run CrossTimeAnnotation, which trains a Keras/TensorFlow neural network on labeled t0 cells to assign t1 labels (DrawUMAP=TRUE), because ignoring emergent cell types can lead to inaccurate annotation and misleading results. Then run DetectNovelCells with corr.cutoff=0.4 and p.cutoff=0.01: cells with correlation <= 0.4 to existing populations and t-test p < 0.01 are flagged as potentially novel. Visualize the flagged cells on a UMAP (Seurat/umap) and expect tumor cells to separate into the two correlation-defined clusters reported for MCL; write accuracy, ARI, and cluster summaries to casi_mcl_results.csv when benchmarking against scmap, CHETAH, singleR, and CAMLU.

## Related tools

- **CASi** (Provides CreateCASiList and DetectNovelCells (correlation + t-test novel-cell detection on a CASiList)) — github.com/yizhuo-wang/CASi
- **Seurat** (UMAP visualization of cell population evolution and the correlation-defined tumor-cell clusters)
- **Keras** (Neural-network backend for CrossTimeAnnotation whose labels precede novel-cell detection)

## Examples

```
require(keras); require(CASi); casi_list <- CreateCASiList(t0_count, t1_count, t0_label); casi_list <- DetectNovelCells(casi_list, corr.cutoff=0.4, p.cutoff=0.01)
```

## Evaluation signals

- In the MCL application, tumor cells separate into exactly two correlation-defined clusters in the UMAP as reported
- CASi's cross-timepoint annotation (prerequisite step) achieves the highest accuracy and highest ARI versus scmap, CHETAH, singleR, and CAMLU
- Flagged novel cells satisfy both thresholds: correlation <= 0.4 and t-test p < 0.01
- The CASiList was built with the default nfeature=2000 highly variable genes before detection
- Cluster summary (and benchmark accuracy/ARI if reproduced) is written to casi_mcl_results.csv

## Limitations

- Novel cell detection may fail when the novel cell is transcriptomically similar to the known cell types
- If the initial timepoint's data is dramatically different from later timepoint's data, the neural network would fail to borrow useful information and could not be very well trained
- The current cell type annotation step pools the data from all the samples together for training, which may be inappropriate when cell populations differ across conditions

## Evidence

- [methods] DetectNovelCells parameters: "For the DetectNovelCells function, we have the following parameters"
- [intro] supervised neural network labeling: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [discussion] novel-cell limitation: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
- [results] MCL benchmark finding: [evidence span withheld — anchor status: not_found]
- [readme] package capability: "(2) detection of potentially novel cell types emerged over time"
