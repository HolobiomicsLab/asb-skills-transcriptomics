---
name: html-report-rendering-with-dimred-plots
description: Use when the 02_int_clustering stage of the scdrake integration pipeline
  has completed clustering and SingleR annotation on integrated data and a rich HTML
  report with dimred figures must be generated in the base output directory.
when_to_use_negative:
- The input is a single-sample (non-integrated) object — the single-sample pipeline's
  02_clustering stage should be used instead of the integration 02_int_clustering
  stage
- No clustering or annotation has been run yet — the report depends on the clustering
  and SingleR upstream targets; rerun the drake subplan first
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scdrake
  role: Provides generate_stage_report and the dimred_plots_clustering targets that
    render the HTML stage report with clustering/annotation figures
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Orchestrates the report target and dimred_plots_clustering targets, so only
    changed parts rerun when parameters change
  repo: https://github.com/bioinfocz/scdrake
- name: SingleR
  role: Supplies the automatic cell-type annotations (with celldex Human Primary Cell
    Atlas and Monaco Immune Data) that color the dimred plots in the report
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:6cda285998e3916bd7668a43804b6a1d2df636b50921410bc94b405ba835d18d
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# html-report-rendering-with-dimred-plots

## Summary

Renders the standalone HTML stage report of the scdrake integration pipeline's second stage (02_int_clustering), embedding dimensionality-reduction plots (PCA, t-SNE, UMAP) colored by clusters and cell-type annotations together with clustering results on the integrated data.

## When to use

Use when the 02_int_clustering stage of the scdrake integration pipeline has completed clustering and SingleR annotation on integrated data and a rich HTML report with dimred figures must be generated in the base output directory.

## When NOT to use

- The input is a single-sample (non-integrated) object — the single-sample pipeline's 02_clustering stage should be used instead of the integration 02_int_clustering stage
- No clustering or annotation has been run yet — the report depends on the clustering and SingleR upstream targets; rerun the drake subplan first

## Inputs

- Integrated SingleCellExperiment object from the 01_integration stage (with HVGs, reduced dimensions, and integration methods computed upstream)
- Clustering results (SNN graph, Leiden/Louvain/Walktrap at configured resolutions, optional k-means/SC3)
- SingleR cell-type annotations using celldex::HumanPrimaryCellAtlasData() and celldex::MonacoImmuneData()
- config/integration/02_int_clustering.yaml, main.yaml, pipeline.yaml

## Outputs

- Standalone HTML stage report for 02_int_clustering in the base output directory
- Dimred plots (PCA, t-SNE, UMAP) colored by clusters and cell-type annotations (dimred_plots_clustering targets)
- Cluster number to cell-type label mappings via CELL_GROUPINGS

## How to apply

Load the integration pipeline configs (config/integration/02_int_clustering.yaml, main.yaml, pipeline.yaml) via scdrake::load_integration_configs and set DRAKE_TARGETS to the post-integration clustering report target. After graph-based clustering (Leiden/Louvain/Walktrap via graph_snn_fn targets, optionally k-means or SC3) and SingleR annotation against celldex references (Human Primary Cell Atlas, Monaco Immune Data) with optional CELL_GROUPINGS cluster-to-label mapping, the dimred_plots_clustering targets produce PCA/t-SNE/UMAP plots colored by cluster and annotation. These figures are assembled by generate_stage_report into a standalone RMarkdown-based HTML report written to the base output directory, consistent with scdrake's design of one rich HTML report per stage.

## Related tools

- **scdrake** (Provides generate_stage_report and the dimred_plots_clustering targets that render the HTML stage report with clustering/annotation figures) — https://github.com/bioinfocz/scdrake
- **drake** (Orchestrates the report target and dimred_plots_clustering targets, so only changed parts rerun when parameters change) — https://github.com/bioinfocz/scdrake
- **SingleR** (Supplies the automatic cell-type annotations (with celldex Human Primary Cell Atlas and Monaco Immune Data) that color the dimred plots in the report)

## Examples

```
configs <- scdrake::load_integration_configs(); DRAKE_TARGETS <- "report_int_clustering"
```

## Evaluation signals

- The HTML stage report file for 02_int_clustering exists in the base output directory
- The report contains clustering and cell-annotation figures (PCA, t-SNE, UMAP colored by cluster and annotation), consistent with each stage outputting an HTML report with rich graphics
- The selected final integration method from 02_int_clustering.yaml is the one carried into the displayed clusters
- Reported annotation labels match the configured CELL_GROUPINGS cluster-to-label mapping

## Limitations

- The report is only as current as the drake cache — up-to-date targets are skipped, so a stale cache can leave an outdated report unless changed parameters trigger rerun
- Docker image availability varies by chipset: only arm64 images up to scdrake version 1.5.0, which may block reproducible report rendering on Apple Silicon

## Evidence

- [readme] Rich graphical and HTML outputs based on customizable RMarkdown documents.: [evidence span withheld — anchor status: external_doc]
- [abstract] outputs rich HTML reports: "The pipeline is reproducible and scalable, has an efficient execution, provides easy extendability and access to intermediate results and outputs rich HTML reports."
- [methods] graph_snn_fn drives clustering: "`graph_snn_fn(sce, snn_k, snn_type, dimred, BPPARAM)`"
- [methods] celldex references used for annotation shown in report: "Human Primary Cell Atlas (`celldex::HumanPrimaryCellAtlasData()`)"
