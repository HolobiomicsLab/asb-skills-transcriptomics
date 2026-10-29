---
name: two-sample-t-test-cell-correlation-testing
description: Use when annotating t1 scRNA-seq cells against a t0 label set (via CASi's
  neural network) and you need to flag cells that may belong to newly emerged cell
  types rather than forcing known labels.
when_to_use_negative:
- The novel cell type is transcriptomically similar to a known cell type — the paper
  states the framework may not work well in this case.
- You only have a single timepoint with no annotated t0 reference to correlate against
  — the t-test operates on correlations to known-type mean UMAP vectors.
- You need unsupervised discovery of all cell types rather than screening for novel
  cells among already-annotated t1 labels — use unsupervised clustering pipelines
  (e.g. Seurat, scran) instead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: Provides DetectNovelCells, which implements the Louvain bisection, two-sample
    t-test on cell-type-specific correlations, and the corr.cutoff/p.cutoff cutoffs
  repo: https://github.com/yizhuo-wang/CASi
- name: Seurat
  role: Single-cell pipeline used alongside CASi for preprocessing/clustering of the
    scRNA-seq data
- name: umap
  role: Computes the per-cell-type UMAP embeddings (reduced to 20 dimensions) whose
    mean vectors define the correlations tested
- name: keras
  role: Backend dependency for CrossTimeAnnotation, which produces the annotated cells
    that DetectNovelCells screens for novelty
- name: ggplot2
  role: Renders the UMAP plots of cell type-specific correlations and cell labels
    for visual inspection
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:db4b2d1a8707648a74f97cc9ca283f69743aba6dda9842812fa2a6ad54a15a75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CASi
  - Seurat
  - ggplot2
  - keras
  - umap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# two-sample-t-test-cell-correlation-testing

## Summary

CASi's novel-cell detection procedure: within each annotated cell type at a later timepoint, cells are Pearson-correlated to the mean UMAP vector of their assigned known type, iteratively bisected by Louvain clustering, and a two-sample t-test plus cutoffs (corr.cutoff = 0.4, p.cutoff = 0.01) flags the low-correlation cluster as potentially novel cell type members.

## When to use

Use when annotating t1 scRNA-seq cells against a t0 label set (via CASi's neural network) and you need to flag cells that may belong to newly emerged cell types rather than forcing known labels.

## When NOT to use

- The novel cell type is transcriptomically similar to a known cell type — the paper states the framework may not work well in this case.
- You only have a single timepoint with no annotated t0 reference to correlate against — the t-test operates on correlations to known-type mean UMAP vectors.
- You need unsupervised discovery of all cell types rather than screening for novel cells among already-annotated t1 labels — use unsupervised clustering pipelines (e.g. Seurat, scran) instead.

## Inputs

- Annotated CASi list produced by CreateCASiList and CrossTimeAnnotation (t0_count, t1_count, t0_label; e.g. pbmc_demo)
- Per-cell-type UMAP embeddings of test cells reduced to 20 dimensions
- Cell-type-specific Pearson correlations of each test cell to each known-type mean UMAP vector

## Outputs

- casinovel_cells.csv with per-cell novel/known designations
- UMAP plots of cell type-specific correlations and cell labels when DrawUMAP = TRUE

## How to apply

First run CreateCASiList on the t0_count/t1_count matrices with t0_label, then CrossTimeAnnotation (Keras/TensorFlow neural network) to assign t1 labels. For DetectNovelCells, compute a UMAP embedding of the test cells per cell type reduced to 20 dimensions, average the UMAP vectors per known cell type, and Pearson-correlate each test cell to each known-type mean vector. For each assigned cell type, iteratively re-cluster the cells with Louvain into two groups until a two-cluster split is obtained, then run a two-sample t-test on the cell-type-specific correlations and designate the lower-correlation group as novel. Apply the default cutoffs — corr.cutoff = 0.4 (cells with correlation > 0.4 are not novel) and p.cutoff = 0.01 (cells with p < 0.01 are novel) — and optionally set DrawUMAP = TRUE to inspect cell-type-specific correlation UMAP plots visually.

## Related tools

- **CASi** (Provides DetectNovelCells, which implements the Louvain bisection, two-sample t-test on cell-type-specific correlations, and the corr.cutoff/p.cutoff cutoffs) — https://github.com/yizhuo-wang/CASi
- **Seurat** (Single-cell pipeline used alongside CASi for preprocessing/clustering of the scRNA-seq data)
- **umap** (Computes the per-cell-type UMAP embeddings (reduced to 20 dimensions) whose mean vectors define the correlations tested)
- **keras** (Backend dependency for CrossTimeAnnotation, which produces the annotated cells that DetectNovelCells screens for novelty)
- **ggplot2** (Renders the UMAP plots of cell type-specific correlations and cell labels for visual inspection)

## Examples

```
novel <- DetectNovelCells(annotated_casilist, corr.cutoff = 0.4, p.cutoff = 0.01, DrawUMAP = TRUE)
```

## Evaluation signals

- casinovel_cells.csv is written and contains a per-cell novel/known designation for every annotated t1 cell
- Cells with correlation larger than 0.4 (corr.cutoff) are designated not novel
- Cells flagged novel have t-test p value smaller than 0.01 (p.cutoff)
- Each per-cell-type re-clustering terminates in a two-cluster Louvain split before the t-test is applied
- With DrawUMAP = TRUE, UMAP plots of cell type-specific correlations and cell labels are generated and visually consistent with the CSV designations

## Limitations

- The novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types.
- Neural-network annotation (the upstream step feeding this skill) fails when the initial timepoint's data is dramatically different from later timepoint data.
- Cell type annotation pools data from all samples together for training/applying the neural network, which may be inappropriate when cell populations differ across conditions.

## Evidence

- [methods] For the DetectNovelCells function, we have the following parameters: "For the DetectNovelCells function, we have the following parameters"
- [discussion] the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
- [intro] Second, new cell types may emerge over the experimental course. Ignoring the potential for new cell types can lead to inaccurate cell annotation: "Second, new cell types may emerge over the experimental course. Ignoring the potential for new cell types can lead to inaccurate cell annotation"
- [readme] CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data"
