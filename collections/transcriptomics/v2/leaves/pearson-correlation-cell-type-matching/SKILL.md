---
name: pearson-correlation-cell-type-matching
description: Use when a multi-timepoint scRNA-seq analysis must identify cells at
  t1 that do not match any t0-annotated cell type, after CrossTimeAnnotation has assigned
  preliminary labels and new cell types may have emerged over time.
when_to_use_negative:
- When the putative novel cell type is transcriptomically similar to a known cell
  type — the article states the detection framework 'may not work well' in that case
- Single-timepoint scRNA-seq data with no prior t0 labels to borrow from — the method
  depends on correlating t1 cells to t0-annotated cell-type mean vectors
- When you only need cross-timepoint annotation without novelty detection — CrossTimeAnnotation
  alone suffices and DetectNovelCells adds the bisection/t-test step unnecessarily
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: Provides CreateCASiList, CrossTimeAnnotation, and DetectNovelCells implementing
    the correlation-matching and novel-cell flagging pipeline
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: Single-cell toolkit used in the demo workflow alongside CASi for scRNA-seq
    handling
- name: umap
  role: Computes the UMAP embedding of test cells reduced to 20 dimensions whose per-type
    mean vectors are the Pearson-correlation targets
- name: keras
  role: Backend dependency for the neural-network classifier in CrossTimeAnnotation
    that supplies annotated t1 labels before novelty detection
- name: ggplot2
  role: Used for the DrawUMAP = TRUE visualization of cell-type-specific correlations
    and labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:6206949a77b9434f28951891dd120393877a495cd261c9655087993222e92d0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - Seurat
  - ggplot2
  - keras
  - umap
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pearson-correlation-cell-type-matching

## Summary

Novel-cell-type detection procedure from CASi's DetectNovelCells function: each cell of an annotated later timepoint (t1) is Pearson-correlated against mean UMAP vectors of known cell types from the initial timepoint (t0), and a Louvain two-cluster bisection plus two-sample t-test flags the low-correlation group as potentially novel cells.

## When to use

Use when a multi-timepoint scRNA-seq analysis must identify cells at t1 that do not match any t0-annotated cell type, after CrossTimeAnnotation has assigned preliminary labels and new cell types may have emerged over time.

## When NOT to use

- When the putative novel cell type is transcriptomically similar to a known cell type — the article states the detection framework 'may not work well' in that case
- Single-timepoint scRNA-seq data with no prior t0 labels to borrow from — the method depends on correlating t1 cells to t0-annotated cell-type mean vectors
- When you only need cross-timepoint annotation without novelty detection — CrossTimeAnnotation alone suffices and DetectNovelCells adds the bisection/t-test step unnecessarily

## Inputs

- CASi list (from CreateCASiList) containing t0_count, t1_count, and t0_label
- Annotated CASi list produced by CrossTimeAnnotation (t1 cells with neural-network cell labels)
- Demo dataset pbmc_demo (lazy-loaded with the package) for testing

## Outputs

- casinovel_cells.csv with per-cell novel/known designations
- Per-cell Pearson correlation values to each known cell-type mean UMAP vector
- UMAP plots of cell-type-specific correlations and cell labels (DrawUMAP = TRUE)

## How to apply

Build a CASi list with CreateCASiList (t0_count, t1_count, t0_label) and run CrossTimeAnnotation (Keras/TF neural network) to label t1 cells. For each assigned cell type, compute a UMAP embedding of the test cells reduced to 20 dimensions, average the UMAP vectors per known cell type, and Pearson-correlate each test cell to each known-type mean vector. Iteratively re-cluster each cell type with the Louvain algorithm into two groups, run a two-sample t-test on the cell-type-specific correlations, and designate the lower-correlation group as suspicious. Apply the default cutoffs — corr.cutoff = 0.4 (correlation above 0.4 means not novel) and p.cutoff = 0.01 (p below 0.01 means novel) — write per-cell designations to casinovel_cells.csv, and with DrawUMAP = TRUE generate UMAP plots of cell-type-specific correlations and labels for visual inspection.

## Related tools

- **CASi** (Provides CreateCASiList, CrossTimeAnnotation, and DetectNovelCells implementing the correlation-matching and novel-cell flagging pipeline) — github.com/yizhuo-wang/CASi
- **Seurat** (Single-cell toolkit used in the demo workflow alongside CASi for scRNA-seq handling)
- **umap** (Computes the UMAP embedding of test cells reduced to 20 dimensions whose per-type mean vectors are the Pearson-correlation targets)
- **keras** (Backend dependency for the neural-network classifier in CrossTimeAnnotation that supplies annotated t1 labels before novelty detection)
- **ggplot2** (Used for the DrawUMAP = TRUE visualization of cell-type-specific correlations and labels)

## Examples

```
require(keras); require(CASi); res <- DetectNovelCells(annotated_casilist, corr.cutoff = 0.4, p.cutoff = 0.01, DrawUMAP = TRUE)
```

## Evaluation signals

- casinovel_cells.csv is produced and contains one novel/known designation per t1 cell
- Cells with Pearson correlation larger than 0.4 (corr.cutoff) are never flagged as novel
- Cells with two-sample t-test p value smaller than 0.01 (p.cutoff) in the low-correlation Louvain cluster are flagged as novel
- Each annotated cell type yields a two-cluster Louvain split before the t-test is applied
- With DrawUMAP = TRUE, UMAP plots of cell-type-specific correlations and cell labels are generated for inspection

## Limitations

- The novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types
- If the initial timepoint's data is dramatically different from later timepoint's data, the neural network annotation step would fail to borrow useful information and could not be well trained, undermining downstream novelty detection
- The current cell type annotation step pools data from all samples together, which may be inappropriate when cell populations differ across conditions

## Evidence

- [methods] novel-cell detection via correlation matching: "For the DetectNovelCells function, we have the following parameters"
- [methods] demo dataset availability: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
- [discussion] transcriptomic-similarity failure mode: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
- [readme] package scope and setup requirements: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation, (2) detection of potentially novel cell types"
- [readme] TensorFlow/keras prerequisite: [evidence span withheld — anchor status: external_doc]
