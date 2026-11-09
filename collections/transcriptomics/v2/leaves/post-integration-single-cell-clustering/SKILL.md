---
name: post-integration-single-cell-clustering
description: Use when a single-sample scRNA-seq pipeline has already produced an integrated
  multi-sample SingleCellExperiment object (with HVGs and reduced dimensions) and
  you need post-integration clustering, cell-type annotation, and an HTML report.
when_to_use_negative:
- Input is raw single-sample data that has not yet been integrated across samples
  — use the single-sample pipeline's second clustering stage instead
- The 01_integration stage has not completed, so no integrated object with computed
  integration methods exists to select from
- The goal is marker gene detection or differential expression between clusters rather
  than clustering/annotation itself
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scdrake
  role: Provides the 02_int_clustering stage, configs, and get_int_clustering_subplan
    for post-integration clustering and annotation
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Drives the clustering subplan as drake targets (graph_snn_fn, dimred_plots_clustering),
    skipping up-to-date parts and exposing intermediate results via cache
  repo: https://github.com/bioinfocz/scdrake
- name: SingleR
  role: Performs automatic cell type annotation of clusters against reference datasets
- name: celldex
  role: Supplies the Human Primary Cell Atlas and Monaco Immune Data reference datasets
    for SingleR annotation
- name: R
  role: Language in which the pipeline and all clustering/annotation steps are implemented
- name: future
  role: Parallel backend for pipeline execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:359bbd730f62621fb56d17fdd7fc960b78515ca59f6f3865a1f4d4d9e50b2f8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - R
  - SingleR
  - celldex
  - drake
  - future
  - scdrake
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# post-integration-single-cell-clustering

## Summary

The scdrake 02_int_clustering stage reruns normalization, dimensionality reduction, graph-based clustering, and SingleR cell-type annotation on data already integrated across samples, producing a standalone HTML report. It lets the analyst pick one final integration method and derive clusters/annotations from the integrated embedding.

## When to use

Use when a single-sample scRNA-seq pipeline has already produced an integrated multi-sample SingleCellExperiment object (with HVGs and reduced dimensions) and you need post-integration clustering, cell-type annotation, and an HTML report.

## When NOT to use

- Input is raw single-sample data that has not yet been integrated across samples — use the single-sample pipeline's second clustering stage instead
- The 01_integration stage has not completed, so no integrated object with computed integration methods exists to select from
- The goal is marker gene detection or differential expression between clusters rather than clustering/annotation itself

## Inputs

- Integrated SingleCellExperiment object from the completed 01_integration stage (with HVGs, reduced dimensions, and computed integration methods)
- Integration pipeline configs: config/integration/02_int_clustering.yaml, main.yaml, pipeline.yaml
- SingleR reference datasets from celldex: Human Primary Cell Atlas and Monaco Immune Data
- Clustering parameters (algorithms, resolutions, snn_k, snn_type) from the 02_int_clustering.yaml config

## Outputs

- Standalone HTML stage report for 02_int_clustering in the base output directory with clustering and cell-annotation figures
- Dimensionality-reduction plots (PCA, t-SNE, UMAP) colored by clusters and annotations (dimred_plots_clustering targets)
- Cluster assignments from Leiden/Louvain/Walktrap (and optionally k-means or SC3) at configured resolutions
- Cell type annotations from SingleR and cluster-to-label mappings from CELL_GROUPINGS

## How to apply

Load the integration pipeline configs (config/integration/02_int_clustering.yaml, main.yaml, pipeline.yaml) with scdrake::load_integration_configs, setting DRAKE_TARGETS to the post-integration clustering report target. Read the integrated object from the completed 01_integration stage, which already contains HVGs, reduced dimensions, and integration methods computed upstream, and select the single final integration method to carry downstream via 02_int_clustering.yaml. Execute graph-based clustering with Leiden/Louvain/Walktrap algorithms at configured resolutions through the get_int_clustering_subplan / graph_snn_fn drake targets, optionally adding k-means or SC3 clustering (calc_sc3). Annotate cells automatically with SingleR using celldex references Human Primary Cell Atlas (HumanPrimaryCellAtlasData()) and Monaco Immune Data (MonacoImmuneData()), and optionally map cluster numbers to labels via CELL_GROUPINGS. Generate PCA/t-SNE/UMAP plots colored by clusters and annotations (dimred_plots_clustering targets) and render the report with generate_stage_report; because drake knows target relationships, only changed parts rerun.

## Related tools

- **scdrake** (Provides the 02_int_clustering stage, configs, and get_int_clustering_subplan for post-integration clustering and annotation) — https://github.com/bioinfocz/scdrake
- **drake** (Drives the clustering subplan as drake targets (graph_snn_fn, dimred_plots_clustering), skipping up-to-date parts and exposing intermediate results via cache) — https://github.com/bioinfocz/scdrake
- **SingleR** (Performs automatic cell type annotation of clusters against reference datasets)
- **celldex** (Supplies the Human Primary Cell Atlas and Monaco Immune Data reference datasets for SingleR annotation)
- **R** (Language in which the pipeline and all clustering/annotation steps are implemented)
- **future** (Parallel backend for pipeline execution)

## Examples

```
configs <- scdrake::load_integration_configs(DRAKE_TARGETS = "report_int_clustering")
```

## Evaluation signals

- The HTML stage report file for 02_int_clustering exists in the base output directory
- The report contains clustering and cell-annotation figures, consistent with each stage outputting an HTML report with rich graphics
- Dimensionality-reduction plots (PCA, t-SNE, UMAP) are colored by both cluster assignments and SingleR annotations
- Exactly one integration method selected downstream, as required by the stage via 02_int_clustering.yaml
- Cluster-to-cell-type label mappings agree with the configured CELL_GROUPINGS parameter

## Limitations

- The stage requires the 01_integration stage to have completed upstream; it cannot run on unintegrated single-sample objects
- Only one final integration method can be carried downstream, so alternatives computed upstream are discarded for this stage
- k-means and SC3 clustering are optional extras; the primary path is graph-based SNN clustering (graph_snn_fn)

## Evidence

- [other] The integration pipeline's second stage repeats the single-sample pipeline's second stage (normalization, dimensionality reduction, clustering, cell type annotation) on the integrated data, ending in a standalone HTML report.: [evidence span withheld — anchor status: not_found]
- [methods] SingleR with celldex references: "Human Primary Cell Atlas (`celldex::HumanPrimaryCellAtlasData()`)"
- [methods] Monaco Immune Data reference: "Monaco Immune Data (`celldex::MonacoImmuneData()`)"
- [abstract] pipeline overview of workflow steps: "The pipeline includes quality control, cell and gene filtering, normalization, detection of highly variable genes, dimensionality reduction, clustering, cell type annotation, detection of marker"
- [readme] drake efficiency rationale: [evidence span withheld — anchor status: external_doc]
- [methods] graph-based clustering targets: "`graph_snn_fn(sce, snn_k, snn_type, dimred, BPPARAM)`"
