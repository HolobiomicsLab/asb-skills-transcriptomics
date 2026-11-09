---
name: louvain-two-group-iterative-reclustering
description: Use when CrossTimeAnnotation has assigned labels to t1 scRNA-seq cells
  but some cells may represent newly emerged cell types that the t0-trained neural
  network classifier would mislabel.
when_to_use_negative:
- The novel cell type is transcriptomically very similar to a known cell type — the
  article states 'the novel cell detection framework may not work well when the novel
  cell is transcriptomically similar to the known cell types'
- Data are single-timepoint/cross-sectional with no earlier timepoint to borrow from
  — CASi is designed for multi-timepoint scRNA-seq, where few pipelines 'considered
  multi-timepoint data specifically'
- You only need standard unsupervised clustering or a supervised annotation method
  that already handles novel cells (e.g. scmap, CHETAH, singleR, CAMLU) rather than
  cross-timepoint novel-type detection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: Provides DetectNovelCells, which implements the iterative Louvain bisection,
    t-test, and cutoff logic; installed from GitHub
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: Supports scRNA-seq object handling and clustering in the demo workflow
- name: umap
  role: Computes the 20-dimensional UMAP embedding from which per-known-type mean
    vectors are averaged
- name: keras
  role: Backend dependency (with TensorFlow) required for the CrossTimeAnnotation
    neural network that precedes novel-cell detection
- name: ggplot2
  role: Renders the UMAP diagnostic plots of correlations and cell labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:41e9e80f581c8bdb969a48ee79eb8bbbef4f32301f7308f1c3325fab309e9240
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - Seurat
  - ggplot2
  - keras
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# louvain-two-group-iterative-reclustering

## Summary

CASi's novel-cell detection step (DetectNovelCells) flags cells in later timepoint data that do not belong to any known type by iteratively bisecting each annotated cell type with Louvain clustering and applying a two-sample t-test on cell-type-specific UMAP-mean correlations, using corr.cutoff = 0.4 and p.cutoff = 0.01.

## When to use

Use when CrossTimeAnnotation has assigned labels to t1 scRNA-seq cells but some cells may represent newly emerged cell types that the t0-trained neural network classifier would mislabel.

## When NOT to use

- The novel cell type is transcriptomically very similar to a known cell type — the article states 'the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types'
- Data are single-timepoint/cross-sectional with no earlier timepoint to borrow from — CASi is designed for multi-timepoint scRNA-seq, where few pipelines 'considered multi-timepoint data specifically'
- You only need standard unsupervised clustering or a supervised annotation method that already handles novel cells (e.g. scmap, CHETAH, singleR, CAMLU) rather than cross-timepoint novel-type detection

## Inputs

- Annotated CASi list produced by CreateCASiList (t0_count, t1_count, t0_label) followed by CrossTimeAnnotation on the t1 count matrix
- pbmc_demo lazy-loaded demo dataset (or equivalent multi-timepoint scRNA-seq count matrices with t0 cell labels)

## Outputs

- casinovel_cells.csv with per-cell novel/known designations
- UMAP plots of cell type-specific correlations and cell labels when DrawUMAP = TRUE

## How to apply

After CrossTimeAnnotation (a Keras/TensorFlow neural network) produces the annotated CASi list, run the novel-cell detection algorithm per cell type: embed the test cells with a UMAP reduced to 20 dimensions, average the UMAP vectors per known cell type, and Pearson-correlate each test cell to each known-type mean vector. For each assigned cell type, iteratively re-cluster the cells with the Louvain algorithm until a two-cluster split is obtained, then run a two-sample t-test on the cell-type-specific correlations. Apply the default cutoffs corr.cutoff = 0.4 (cells with correlation larger than 0.4 are not novel) and p.cutoff = 0.01 (cells with p value smaller than 0.01 are novel); the cluster with smaller correlation values is flagged as suspicious (novel) cells. Write the per-cell novel/known designations to casinovel_cells.csv and set DrawUMAP = TRUE to generate UMAP plots of the cell type-specific correlations and cell labels for visual inspection.

## Related tools

- **CASi** (Provides DetectNovelCells, which implements the iterative Louvain bisection, t-test, and cutoff logic; installed from GitHub) — github.com/yizhuo-wang/CASi
- **Seurat** (Supports scRNA-seq object handling and clustering in the demo workflow)
- **umap** (Computes the 20-dimensional UMAP embedding from which per-known-type mean vectors are averaged)
- **keras** (Backend dependency (with TensorFlow) required for the CrossTimeAnnotation neural network that precedes novel-cell detection)
- **ggplot2** (Renders the UMAP diagnostic plots of correlations and cell labels)

## Examples

```
novel <- DetectNovelCells(annotated_casi_list, corr.cutoff = 0.4, p.cutoff = 0.01, DrawUMAP = TRUE)
```

## Evaluation signals

- For every annotated cell type, the iterative Louvain re-clustering terminates in exactly a two-cluster split before the t-test is applied
- Cells with correlation larger than corr.cutoff = 0.4 to their known-type mean UMAP vector are never designated novel, regardless of the t-test
- Cells designated novel must have p value smaller than p.cutoff = 0.01 in the two-sample t-test on cell-type-specific correlations
- Output file casinovel_cells.csv exists and contains a novel/known designation for every cell in the annotated t1 CASi list
- With DrawUMAP = TRUE, per-cell-type UMAP plots of correlations and cell labels are produced for visual inspection

## Limitations

- Novel cell detection may fail when the novel cell type is transcriptomically similar to known cell types (stated in the discussion)
- It depends on CrossTimeAnnotation succeeding; if the initial timepoint's data is dramatically different from later timepoints, the neural network fails to borrow useful information and could not be well trained, corrupting downstream detection
- CASi is not yet on Bioconductor (a stated to-do), so installation is via devtools::install_github and TensorFlow/keras must be configured first

## Evidence

- [methods] For the DetectNovelCells function, we have the following parameters: "For the DetectNovelCells function, we have the following parameters"
- [discussion] discussion limitation: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
- [readme] README capability statement: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation, (2) detection of potentially novel cell types emerged"
- [other] task workflow cutoffs: [evidence span withheld — anchor status: not_found]
