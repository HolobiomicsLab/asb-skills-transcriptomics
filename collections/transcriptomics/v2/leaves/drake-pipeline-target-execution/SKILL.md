---
name: drake-pipeline-target-execution
description: Use when running or re-running a scdrake single-sample or integration
  pipeline stage (input QC, integration, clustering, cluster_markers, contrasts) where
  incremental, reproducible target execution via drake is needed.
when_to_use_negative:
- You need to run a non-scdrake workflow — this skill is specific to scdrake's drake-based
  plan structure and its config YAMLs
- You expect every stage to recompute from scratch on each run — drake deliberately
  skips up-to-date targets, so a full fresh rerun requires clearing the cache
- The task is primary analysis of raw sequencing data (FASTQ/Cell Ranger itself) —
  scdrake covers secondary analysis of the feature-barcode matrix
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scdrake
  role: Pipeline package whose stages are defined as drake plans; provides run_single_sample_r(),
    subplan getters, and stage config loaders
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Make-like pipeline toolkit that tracks target dependencies, skips up-to-date
    targets, and caches checkpoints
  repo: https://github.com/bioinfocz/scdrake
- name: future
  role: Parallel backend for target execution; install with future.callr
- name: yq
  role: Command-line tool used to manipulate the pipeline YAML config files
  repo: https://github.com/mikefarah/yq
- name: qs
  role: Fast serialization of R objects used to cache intermediate pipeline results
  repo: https://github.com/qsbase/qs
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:e33bc7134a81f5074fdd0d231d3a00ee3d07d4a396c9f05ad1d0205334ebcdc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - Docker
  - Monaco Immune Data
  - R
  - SingleR
  - celldex
  - drake
  - future
  - qs
  - scdrake
  - yq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# drake-pipeline-target-execution

## Summary

Execute scdrake's secondary scRNA-seq analysis stages as {drake} targets, so that only changed parts of the pipeline rerun while up-to-date targets are skipped. Targets are driven by per-stage YAML configs (main.yaml, pipeline.yaml, e.g. 01_input_qc.yaml) and end in standalone HTML reports.

## When to use

Use when running or re-running a scdrake single-sample or integration pipeline stage (input QC, integration, clustering, cluster_markers, contrasts) where incremental, reproducible target execution via drake is needed.

## When NOT to use

- You need to run a non-scdrake workflow — this skill is specific to scdrake's drake-based plan structure and its config YAMLs
- You expect every stage to recompute from scratch on each run — drake deliberately skips up-to-date targets, so a full fresh rerun requires clearing the cache
- The task is primary analysis of raw sequencing data (FASTQ/Cell Ranger itself) — scdrake covers secondary analysis of the feature-barcode matrix

## Inputs

- scdrake YAML config files (main.yaml, pipeline.yaml, stage configs such as config/single_sample/01_input_qc.yaml, 02_norm_clustering.yaml, contrasts.yaml, cluster_markers.yaml)
- 10x Genomics Cell Ranger output (feature-barcode matrix, e.g. example_data/pbmc1k from PBMC 1k) or SingleCellExperiment objects from upstream stages (e.g. sce_final_input_qc)
- drake plan/subplan R objects (e.g. from get_cluster_markers_subplan or get_integration_plan)

## Outputs

- Executed drake targets and intermediate SingleCellExperiment objects cached in the drake cache
- Stage HTML reports (e.g. output/single_sample/01_input_qc/01_input_qc.html, 02_norm_clustering.html) with rich graphical outputs under BASE_OUT_DIR
- Per-contrast DE tables and cluster marker tables

## How to apply

Load the stage configs (e.g. scdrake::load_single_sample_configs() or load_integration_configs() for main.yaml, pipeline.yaml, and the stage YAML), and restrict execution with DRAKE_TARGETS in pipeline.yaml (e.g. report_input_qc, report_norm_clustering, report_norm_clustering_simple, or a stage subplan such as get_contrasts_subplan()). Build the stage plan/subplan — e.g. get_cluster_markers_subplan(cfg, cfg_pipeline, cfg_main) or get_integration_plan() — and run it via scdrake::run_single_sample_r() or the CLI `scdrake --pipeline-type single_sample run`, with the future backend installed (BiocManager::install(c('future', 'future.callr'))) for parallel execution. Because drake knows the inner relationships between targets, editing a parameter (e.g. MAD_THRESHOLD: 2 in 01_input_qc.yaml) reruns only downstream targets while up-to-date ones are skipped; intermediate results are accessible as smartly defined checkpoints loaded from the drake cache. Each stage terminates in a rendered HTML report (e.g. output/single_sample/01_input_qc/01_input_qc.html) written under BASE_OUT_DIR via generate_stage_report(). YAML files can be manipulated with the yq tool, and R objects are serialized with the fast qs format.

## Related tools

- **scdrake** (Pipeline package whose stages are defined as drake plans; provides run_single_sample_r(), subplan getters, and stage config loaders) — https://github.com/bioinfocz/scdrake
- **drake** (Make-like pipeline toolkit that tracks target dependencies, skips up-to-date targets, and caches checkpoints) — https://github.com/bioinfocz/scdrake
- **future** (Parallel backend for target execution; install with future.callr)
- **yq** (Command-line tool used to manipulate the pipeline YAML config files) — https://github.com/mikefarah/yq
- **qs** (Fast serialization of R objects used to cache intermediate pipeline results) — https://github.com/qsbase/qs

## Examples

```
scdrake::load_single_sample_configs(); scdrake::run_single_sample_r()
```

## Evaluation signals

- drake make completes without failed targets
- The stage HTML report file exists at the expected path (e.g. output/single_sample/01_input_qc/01_input_qc.html) and contains the stage's QC/clustering/DE summaries
- After a parameter change, only the downstream targets rerun while up-to-date targets are skipped
- Intermediate results can be loaded from the drake cache via the defined checkpoints
- Outputs cover exactly the sources defined in the stage config (e.g. CONTRASTS_SOURCES comparisons or CLUSTER_MARKERS_SOURCES groupings)

## Limitations

- The future backend requires installing {future} and {future.callr} (BiocManager::install) before parallel target execution works
- The qs serialization package is deprecated in favor of qs2, which may affect cached-object compatibility
- scdrake was marked lifecycle-experimental/stable with fixed Docker image versions (e.g. jirinovo/scdrake:1.7.1 based on Bioconductor 3.21); results depend on the pinned dependency versions

## Evidence

- [abstract] implemented in R within the drake framework: "which is fully implemented in the R language and built within the drake framework"
- [methods] drake target dependency awareness: "Because `{drake}` knows the inner relationships between targets in your plan, it also knows which targets are independent of each other"
- [methods] future backend: ""future": uses the `{future}` as the backend. This backend should work by simply installing the `{future}` package."
- [readme] incremental reruns: [evidence span withheld — anchor status: external_doc]
- [readme] checkpoint access: [evidence span withheld — anchor status: external_doc]
