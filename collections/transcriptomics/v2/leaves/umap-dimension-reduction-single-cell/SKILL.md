---
name: umap-dimension-reduction-single-cell
description: Use when single-cell expression data needs a low-dimensional embedding
  for downstream similarity or clustering tasks — e.g. computing per-cell-type 20-dim
  UMAP vectors for CASi's correlation-based novel-cell detection.
when_to_use_negative:
- The initial timepoint's data is dramatically different from later timepoint data
  — the neural network annotation step upstream would fail to train, corrupting the
  cells fed into the UMAP embedding
- The goal is unsupervised clustering of a single cross-sectional dataset without
  cross-timepoint label transfer — CASi's UMAP-then-correlate scheme assumes known
  t0 cell-type means to correlate against
- Novel cells are transcriptomically very similar to known cell types — the article
  notes the framework may not work well in that regime
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CASi
  role: Framework orchestrating the annotation-to-UMAP-to-novel-detection pipeline;
    DetectNovelCells wraps the UMAP/correlation computation
  repo: https://github.com/yizhuo-wang/CASi
- name: umap
  role: R package used to compute the per-cell-type UMAP embedding reduced to 20 dimensions
- name: Seurat
  role: Single-cell preprocessing/clustering pipeline used alongside CASi in the demo
    workflow
- name: keras
  role: Backend for CrossTimeAnnotation, which produces the annotated cells that the
    UMAP embedding is computed on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:04e9c2b4f7fa8b23842f3006e7cd84d6c5814cf173f4d7ca2e93594ac6cd4d48
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# umap-dimension-reduction-single-cell

## Summary

Applies UMAP dimension reduction to single-cell data in the CASi framework: per-cell-type UMAP embeddings reduced to 20 dimensions are averaged per known cell type, and each test cell's Pearson correlation to the type-mean vector drives downstream novel-cell detection.

## When to use

Use when single-cell expression data needs a low-dimensional embedding for downstream similarity or clustering tasks — e.g. computing per-cell-type 20-dim UMAP vectors for CASi's correlation-based novel-cell detection.

## When NOT to use

- The initial timepoint's data is dramatically different from later timepoint data — the neural network annotation step upstream would fail to train, corrupting the cells fed into the UMAP embedding
- The goal is unsupervised clustering of a single cross-sectional dataset without cross-timepoint label transfer — CASi's UMAP-then-correlate scheme assumes known t0 cell-type means to correlate against
- Novel cells are transcriptomically very similar to known cell types — the article notes the framework may not work well in that regime

## Inputs

- Gene expression matrix of the initial timepoint (t0)
- Gene expression matrix of the later timepoint (t1)
- Cell labels of the initial data point (t0_label)
- Annotated CASi list produced by CrossTimeAnnotation

## Outputs

- Per-cell-type UMAP embeddings reduced to 20 dimensions
- Per-cell Pearson correlation values against known-cell-type mean UMAP vectors
- casinovel_cells.csv with per-cell novel/known designations
- UMAP plots of cell-type-specific correlations and cell labels (when DrawUMAP = TRUE)

## How to apply

After CASi's CrossTimeAnnotation step labels the t1 cells (Keras neural network on TensorFlow backend), compute a UMAP embedding of the test cells separately per assigned cell type, reducing the embedding to 20 dimensions. Average the 20-dim UMAP vectors within each known cell type to obtain type-mean vectors, then Pearson-correlate each test cell against each known-type mean vector. These cell-type-specific correlations feed the novel-cell detection filter: cells with correlation above corr.cutoff = 0.4 are not novel, and cells with p < 0.01 (two-sample t-test after iterative Louvain two-cluster bisection) are designated novel. Set DrawUMAP = TRUE in DetectNovelCells to generate UMAP plots of the cell-type-specific correlations and cell labels for visual inspection of the embedding quality.

## Related tools

- **CASi** (Framework orchestrating the annotation-to-UMAP-to-novel-detection pipeline; DetectNovelCells wraps the UMAP/correlation computation) — https://github.com/yizhuo-wang/CASi
- **umap** (R package used to compute the per-cell-type UMAP embedding reduced to 20 dimensions)
- **Seurat** (Single-cell preprocessing/clustering pipeline used alongside CASi in the demo workflow)
- **keras** (Backend for CrossTimeAnnotation, which produces the annotated cells that the UMAP embedding is computed on)

## Examples

```
require(keras); require(CASi); casi <- CreateCASiList(t0_count, t1_count, t0_label); casi <- CrossTimeAnnotation(casi); casi <- DetectNovelCells(casi, corr.cutoff = 0.4, p.cutoff = 0.01, DrawUMAP = TRUE)
```

## Evaluation signals

- The UMAP embedding for each cell type has exactly 20 dimensions before averaging into type-mean vectors
- All Pearson correlations fall in [-1, 1]; cells with correlation > 0.4 (corr.cutoff) must be labeled non-novel
- Cells with p < 0.01 (p.cutoff) from the two-sample t-test on cell-type-specific correlations are flagged novel
- casinovel_cells.csv exists and contains a per-cell novel/known designation for every t1 cell
- With DrawUMAP = TRUE, UMAP plots of the correlations and cell labels are produced for each cell type and are inspectable

## Limitations

- The novel cell detection framework may not work well when the novel cell type is transcriptomically similar to known cell types
- If the initial timepoint's data is dramatically different from later timepoint data, the upstream neural network annotation fails to borrow useful information, degrading the embedding input
- CASi is not yet published on Bioconductor; it must be installed from GitHub via devtools and requires TensorFlow/keras setup before any UMAP-based downstream step can run

## Evidence

- [other] compute a UMAP embedding of the test cells per cell type reduced to 20 dimensions, average the UMAP vectors per known cell type, and Pearson-correlate each test cell: [evidence span withheld — anchor status: not_found]
- [readme] CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data"
- [methods] The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package.: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
- [intro] the downstream analysis of scRNA-seq data includes dimension reduction, unsupervised clustering: "the downstream analysis of scRNA-seq data includes dimension reduction, unsupervised clustering"
- [discussion] the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
