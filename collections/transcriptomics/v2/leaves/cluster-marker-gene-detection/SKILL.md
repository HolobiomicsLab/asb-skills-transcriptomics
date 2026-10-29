---
name: cluster-marker-gene-detection
description: Use when a scRNA-seq dataset has already been clustered (e.g. a clustered
  SCE from the clustering stage) and per-cluster marker genes must be identified,
  ranked, and reported for each cell grouping.
when_to_use_negative:
- The input SingleCellExperiment has not yet been clustered — run the clustering stage
  first, as markers are computed per cluster grouping.
- The goal is differential expression between defined contrasts rather than per-cluster
  marker detection — scdrake treats these as separate stages ('contrasts').
- The data are not scRNA-seq/SRT secondary analysis outputs and no SingleCellExperiment
  or cluster annotations exist.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scdrake
  role: Pipeline providing the cluster_markers stage, its configs (cluster_markers.yaml),
    subplan builder get_cluster_markers_subplan(), and HTML report generation
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Workflow framework that manages the cluster_markers_raw -> cluster_markers_processed
    targets and caches intermediate results
  repo: https://github.com/bioinfocz/scdrake
- name: R
  role: Language in which the marker detection stage, filter_markers(), and report
    rendering are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:4fbb10bd9b8a4d426ed5c0d5d4f16ae2d74221b3aba54b680d1ef24ea339d1bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - R
  - drake
  - scdrake
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cluster-marker-gene-detection

## Summary

Detection of cluster marker genes as implemented in the scdrake pipeline: statistical tests of marker genes per cluster grouping of a clustered SingleCellExperiment, followed by top-marker filtering and a standalone HTML report. This stage is shared by both the single-sample and multi-sample integration pipelines.

## When to use

Use when a scRNA-seq dataset has already been clustered (e.g. a clustered SCE from the clustering stage) and per-cluster marker genes must be identified, ranked, and reported for each cell grouping.

## When NOT to use

- The input SingleCellExperiment has not yet been clustered — run the clustering stage first, as markers are computed per cluster grouping.
- The goal is differential expression between defined contrasts rather than per-cluster marker detection — scdrake treats these as separate stages ('contrasts').
- The data are not scRNA-seq/SRT secondary analysis outputs and no SingleCellExperiment or cluster annotations exist.

## Inputs

- clustered SingleCellExperiment object (from the clustering stage, e.g. task_004)
- cluster_markers.yaml stage config
- pipeline.yaml and main config (loaded via load_single_sample_configs())
- CLUSTER_MARKERS_SOURCES selection of cell groupings (clustering columns in the SCE)

## Outputs

- cluster_markers_raw and cluster_markers_processed drake targets (per-cluster marker test results)
- Filtered top marker tables per cluster (via filter_markers)
- Marker heatmap images
- Standalone HTML report under BASE_OUT_DIR with per-cluster marker tables and heatmap links

## How to apply

Load the cluster_markers.yaml stage config together with pipeline.yaml and the main config via scdrake's load_single_sample_configs(). Build the stage subplan with get_cluster_markers_subplan(cfg, cfg_pipeline, cfg_main), using CLUSTER_MARKERS_SOURCES to select the cell groupings (e.g. a clustering column from the clustered SCE) for which markers are computed. Execute the drake targets cluster_markers_raw -> cluster_markers_processed via cluster_markers_fn to run the per-cluster marker statistical tests. Filter the raw marker results with filter_markers(markers, top_n, top_n_wt, distinct_by) to keep the top markers per cluster, then assemble the marker tables and heatmap sections and render the standalone HTML report with generate_stage_report() from the stage Rmd template, saving outputs under BASE_OUT_DIR. Because the pipeline is built on drake, only changed parts rerun when parameters are modified.

## Related tools

- **scdrake** (Pipeline providing the cluster_markers stage, its configs (cluster_markers.yaml), subplan builder get_cluster_markers_subplan(), and HTML report generation) — https://github.com/bioinfocz/scdrake
- **drake** (Workflow framework that manages the cluster_markers_raw -> cluster_markers_processed targets and caches intermediate results) — https://github.com/bioinfocz/scdrake
- **R** (Language in which the marker detection stage, filter_markers(), and report rendering are implemented)

## Examples

```
cfg <- load_single_sample_configs(); plan <- get_cluster_markers_subplan(cfg, cfg_pipeline, cfg_main); drake::make(plan)
```

## Evaluation signals

- The stage's standalone HTML report exists under BASE_OUT_DIR and contains per-cluster marker tables.
- The report contains links to marker heatmaps, with markers ranked per cluster grouping.
- cluster_markers_raw and cluster_markers_processed drake targets both complete and are cached (drake skips up-to-date targets on rerun).
- filter_markers output retains only the top markers per cluster according to top_n, top_n_wt, and distinct_by settings.
- The report is rendered from the stage Rmd template via generate_stage_report() and is openable as a standalone document.

## Limitations

- Marker detection depends on the quality of upstream clustering; markers are only computed for groupings present in the clustered SCE.
- The pipeline is marked as experimental in parts (lifecycle badge) and scdrake is primarily intended for tech-savvy bioinformaticians who pass results on to biologists.
- The qs serialization backend used by the pipeline is deprecated in favor of qs2, which may affect long-term reproducibility of cached intermediate results.

## Evidence

- [abstract] The pipeline includes quality control, cell and gene filtering, normalization, detection of highly variable genes, dimensionality reduction, clustering, cell type annotation, detection of marker: "The pipeline includes quality control, cell and gene filtering, normalization, detection of highly variable genes, dimensionality reduction, clustering, cell type annotation, detection of marker"
- [methods] filter_markers signature: "`filter_markers(markers, top_n, top_n_wt, distinct_by)`"
- [methods] drake target relationships: "Because `{drake}` knows the inner relationships between targets in your plan, it also knows which targets are independent of each other"
- [readme] README marker computation feature: [evidence span withheld — anchor status: external_doc]
- [readme] README rich HTML outputs: "- Rich graphical and HTML outputs based on customizable RMarkdown
  documents."
