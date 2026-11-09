---
name: cross-batch-highly-variable-gene-selection
description: Use when integrating multiple independently preprocessed single-sample
  scRNA-seq batches in the scdrake integration pipeline and a joint HVG set is needed
  before running batch integration methods.
when_to_use_negative:
- Running the single-sample pipeline on one dataset — no cross-batch HVG selection
  is needed or available there
- Batches were not preprocessed by the single-sample pipeline (e.g. raw Cell Ranger
  output without prior QC/normalization)
- Only one batch is supplied — integration HVG selection has nothing to reconcile
  across samples
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scdrake
  role: Provides hvg_int_list_fn(), integration_methods_df_fn(), and the integration
    configs/plans used for cross-batch HVG selection
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Pipeline framework that binds the integration plan and tracks the HVG-selection
    target so unchanged parts are skipped on rerun
- name: yq
  role: Command-line tool used to manipulate the YAML config files (e.g. 01_integration.yaml)
  repo: https://github.com/mikefarah/yq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:a77fb76ce110c9c11a01e1b72564a07671d5af414eb662bea956e938a3ddfc59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - Docker
  - R
  - drake
  - scdrake
  - yq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Cross-batch highly variable gene selection

## Summary

Selects highly variable genes jointly across multiple single-sample scRNA-seq batches (e.g. PBMC 1k and PBMC 3k) as the HVG step of the scdrake 01_integration stage, so that batch integration methods can compute corrected reduced dimensions on a shared gene space.

## When to use

Use when integrating multiple independently preprocessed single-sample scRNA-seq batches in the scdrake integration pipeline and a joint HVG set is needed before running batch integration methods.

## When NOT to use

- Running the single-sample pipeline on one dataset — no cross-batch HVG selection is needed or available there
- Batches were not preprocessed by the single-sample pipeline (e.g. raw Cell Ranger output without prior QC/normalization)
- Only one batch is supplied — integration HVG selection has nothing to reconcile across samples

## Inputs

- Per-batch SingleCellExperiment objects from the single-sample scdrake pipeline (one per sample, e.g. PBMC 1k, PBMC 3k)
- config/integration/01_integration.yaml with INTEGRATION_SOURCES set
- Integration pipeline configs from scdrake::load_integration_configs()

## Outputs

- List of integration HVGs selected across batches (via hvg_int_list_fn())
- Integrated SingleCellExperiment carrying cross-batch HVGs and corrected reduced dimensions
- 01_integration stage HTML report saved under BASE_OUT_DIR

## How to apply

Set INTEGRATION_SOURCES in config/integration/01_integration.yaml to point at the per-batch outputs (e.g. the PBMC 1k single-sample pipeline output and the PBMC 3k 10x Genomics matrix), then load configs with scdrake::load_integration_configs() and build the plan with scdrake::get_integration_plan()/get_integration_subplan() under drake. Import and merge the per-batch SingleCellExperiment objects (batches = individual samples). Select integration HVGs across all batches with scdrake::hvg_int_list_fn(), building on the pipeline's HVG machinery (e.g. get_top_hvgs() with a fitted HVG metric). Pass the cross-batch HVG set to scdrake::integration_methods_df_fn() to compute corrected reduced dimensions, and render the 01_integration stage HTML report as the final drake target under BASE_OUT_DIR. The rationale is that integration requires a gene set defined consistently across batches rather than per-sample HVG lists.

## Related tools

- **scdrake** (Provides hvg_int_list_fn(), integration_methods_df_fn(), and the integration configs/plans used for cross-batch HVG selection) — https://github.com/bioinfocz/scdrake
- **drake** (Pipeline framework that binds the integration plan and tracks the HVG-selection target so unchanged parts are skipped on rerun)
- **yq** (Command-line tool used to manipulate the YAML config files (e.g. 01_integration.yaml)) — https://github.com/mikefarah/yq

## Examples

```
configs <- scdrake::load_integration_configs(); plan <- scdrake::get_integration_plan()
```

## Evaluation signals

- The 01_integration stage HTML report file exists under BASE_OUT_DIR and renders
- The integrated object carries the HVGs selected across batches (not per-sample HVG lists)
- The integrated object carries corrected reduced dimensions computed from those HVGs
- The HVGs and reduced dimensions are usable by the downstream 02_int_clustering stage
- drake treats up-to-date targets as finished — changing only HVG parameters reruns only affected targets

## Limitations

- Batches are assumed to be individual samples already preprocessed by the single-sample pipeline; raw feature-barcode matrices must first go through single-sample QC and normalization
- The pipeline is marked 'Lifecycle: experimental' in the repository README, so defaults may change between versions
- No specific HVG count or threshold for integration HVG selection is stated in the article; parameters must be set in the integration YAML configs

## Evidence

- [methods] scdrake offers two pipelines: single-sample and multi-sample integration: "`{scdrake}` offers two pipelines - one for single-sample, and second one for integration of multiple samples"
- [readme] README lists HVG detection and dataset integration as main features: [evidence span withheld — anchor status: external_doc]
- [readme] README lists integration of multiple datasets as a main feature: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract lists detection of highly variable genes among pipeline steps: "The pipeline includes quality control, cell and gene filtering, normalization, detection of highly variable genes"
