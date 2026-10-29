---
name: novel-cell-type-detection
description: Use when a labeled t0 scRNA-seq matrix has been cross-timepoint-annotated
  onto a t1 matrix (CrossTimeAnnotation) and you need to flag emerged/novel cell types
  or tumor subclones rather than force-assigning them known labels.
when_to_use_negative:
- The novel population of interest is transcriptomically very similar to a known cell
  type — the article notes the detection framework 'may not work well' in that case
- The t0 data differ dramatically from later timepoints, so the neural-network annotation
  step that feeds this skill cannot be well trained
- You need de novo unsupervised clustering of a single-timepoint dataset with no labeled
  reference — this skill assumes labeled t0 cells and prior cross-timepoint annotation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CASi
  role: Provides CreateCASiList, CrossTimeAnnotation, and DetectNovelCells — the detection
    algorithm (UMAP mean-correlation, Louvain bisection, t-test) is implemented in
    DetectNovelCells
  repo: https://github.com/yizhuo-wang/CASi
- name: Seurat
  role: Underlying single-cell infrastructure used for the UMAP embedding of test
    cells per cell type
- name: keras
  role: Required dependency for the TensorFlow-backend neural network in CrossTimeAnnotation
    that precedes novel-cell detection
- name: umap
  role: Generates the UMAP embedding used for cell-type-specific correlation computation
    and the DrawUMAP=TRUE diagnostic plots
- name: CHETAH
  role: Existing annotation method benchmarked against CASi's novel-cell recovery
    (identifies only a small portion of novel cells in simulation scenario 3)
- name: scPred
  role: Existing annotation method benchmarked against CASi (fails to capture any
    novel cells in simulation scenario 3)
- name: scmap
  role: Existing annotation method benchmarked against CASi (fails to capture any
    novel cells in simulation scenario 3)
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:d50f37f8588b420c6bd274405c001eb1276a39efc629dc11082f096aa683f66c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CAMLU
  - CASi
  - CHETAH
  - Seurat
  - ggplot2
  - keras
  - scPred
  - scmap
  - singleR
  - umap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6190476190476191
n_claims: 21
---

# novell-cell-type-detection

## Summary

Detect cell populations at later timepoints (t1) that do not correspond to any known initial-timepoint (t0) cell type, using CASi's DetectNovelCells: UMAP-based cell-type-specific correlations, iterative Louvain two-cluster bisection, and a two-sample t-test with corr.cutoff=0.4 and p.cutoff=0.01.

## When to use

Use when a labeled t0 scRNA-seq matrix has been cross-timepoint-annotated onto a t1 matrix (CrossTimeAnnotation) and you need to flag emerged/novel cell types or tumor subclones rather than force-assigning them known labels.

## When NOT to use

- The novel population of interest is transcriptomically very similar to a known cell type — the article notes the detection framework 'may not work well' in that case
- The t0 data differ dramatically from later timepoints, so the neural-network annotation step that feeds this skill cannot be well trained
- You need de novo unsupervised clustering of a single-timepoint dataset with no labeled reference — this skill assumes labeled t0 cells and prior cross-timepoint annotation

## Inputs

- Gene expression count matrix of the initial timepoint (t0)
- Gene expression count matrix of the following timepoint (t1)
- Cell labels of the initial timepoint (t0)
- A CASi list object returned by CrossTimeAnnotation (annotated t1 cells)

## Outputs

- Per-cell novel/known designations (e.g. written to casinovel_cells.csv)
- UMAP plots colored by cell-type-specific correlation and cell labels (e.g. novel_cells_umap.png) when DrawUMAP=TRUE
- Correlation-defined novel cell clusters (e.g. two tumor-cell clusters in the MCL application)

## How to apply

Build a CASi list with CreateCASiList(t0_count, t1_count, t0_label) using the default nfeature=2000 highly variable genes, then run CrossTimeAnnotation (a Keras neural network on the TensorFlow backend trained only on t0 labels) to obtain an annotated CASi list. For the novel-cell detection, compute a UMAP embedding of the t1 test cells per cell type reduced to 20 dimensions, average the UMAP vectors per known cell type, and Pearson-correlate each test cell to each known-type mean vector. For each assigned cell type, iteratively re-cluster with the Louvain algorithm until a two-cluster split is obtained, then run a two-sample t-test on the cell-type-specific correlations; the lower-correlation cluster is flagged as suspicious. Apply the default cutoffs corr.cutoff=0.4 (cells with correlation larger than 0.4 are not novel) and p.cutoff=0.01 (cells with p value smaller than 0.01 are novel), and optionally set DrawUMAP=TRUE to visualize cell-type-specific correlations and labels.

## Related tools

- **CASi** (Provides CreateCASiList, CrossTimeAnnotation, and DetectNovelCells — the detection algorithm (UMAP mean-correlation, Louvain bisection, t-test) is implemented in DetectNovelCells) — https://github.com/yizhuo-wang/CASi
- **Seurat** (Underlying single-cell infrastructure used for the UMAP embedding of test cells per cell type)
- **keras** (Required dependency for the TensorFlow-backend neural network in CrossTimeAnnotation that precedes novel-cell detection)
- **umap** (Generates the UMAP embedding used for cell-type-specific correlation computation and the DrawUMAP=TRUE diagnostic plots)
- **CHETAH** (Existing annotation method benchmarked against CASi's novel-cell recovery (identifies only a small portion of novel cells in simulation scenario 3))
- **scPred** (Existing annotation method benchmarked against CASi (fails to capture any novel cells in simulation scenario 3))
- **scmap** (Existing annotation method benchmarked against CASi (fails to capture any novel cells in simulation scenario 3))

## Examples

```
casi_list <- CreateCASiList(t0_count, t1_count, t0_label); casi_list <- CrossTimeAnnotation(casi_list); novel <- DetectNovelCells(casi_list, corr.cutoff = 0.4, p.cutoff = 0.01, DrawUMAP = TRUE)
```

## Evaluation signals

- The analysis script runs without error and the output figure (e.g. novel_cells_umap.png) exists as a valid image file
- In simulation scenario 3 (novel B cells at t1/t2), the recovered novel cells match the ground truth — the novel cluster is flagged by correlation/p-value rather than misassigned to known t0 labels
- With default cutoffs, flagged novel cells satisfy correlation <= 0.4 to every known-type mean UMAP vector and two-sample t-test p < 0.01
- In the MCL real-data application, tumor cells separate into exactly two correlation-defined clusters in the UMAP, as reported in the paper
- CASi's novel-cell detection outperforms CHETAH, scPred, and scmap, which recover only a small portion or none of the novel cells

## Limitations

- Novel cell detection may fail when the novel cell type is transcriptomically similar to known cell types
- It depends on the upstream neural-network annotation: if the initial timepoint's data is dramatically different from later timepoint data, the network cannot borrow useful information and the pipeline degrades
- The framework requires labeled t0 cells and t1 data from the same experimental context; it is not a de novo single-timepoint novel-population discovery method
- Results for the three simulation scenarios (ARI comparisons) appear only in supplementary Figure 11, not the main text

## Evidence

- [abstract] detection of potentially novel cell types emerged over time: "(2) detection of potentially novel cell types emerged over time"
- [methods] For the DetectNovelCells function, we have the following parameters: "For the DetectNovelCells function, we have the following parameters"
- [methods] The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package.: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
- [discussion] the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
- [readme] CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation, (2) detection of potentially novel cell types"
- [readme] please make sure to set up TensorFlow from https://tensorflow.rstudio.com/install/, and that the dependent package "keras" is available to use.: [evidence span withheld — anchor status: external_doc]
