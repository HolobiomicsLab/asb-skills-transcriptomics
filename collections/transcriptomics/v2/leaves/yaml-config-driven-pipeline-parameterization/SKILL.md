---
name: yaml-config-driven-pipeline-parameterization
description: Use when adapting or re-running a scdrake scRNA-seq/SRT pipeline run
  for new parameters or cell groupings — e.g. changing a MAD threshold, restricting
  DRAKE_TARGETS, or setting CONTRASTS_SOURCES in contrasts.yaml.
when_to_use_negative:
- The task requires changing pipeline logic or adding new computations rather than
  tuning parameters — scdrake's pipeline definition is an R/drake object meant to
  be extended in R, not through YAML
- The analysis is a one-off custom script outside the scdrake/drn (drake) framework,
  where incremental target skipping provides no benefit
- The input is raw data import itself (10x Cell Ranger output, delimited table, or
  SingleCellExperiment) rather than pipeline configuration — this skill covers parameterization,
  not data ingestion
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scdrake
  role: The pipeline package whose per-stage YAML configs (main.yaml, pipeline.yaml,
    contrasts.yaml) are edited to parameterize runs
  repo: https://github.com/bioinfocz/scdrake
- name: yq
  role: Command-line YAML processor used to programmatically manipulate scdrake's
    YAML config files
  repo: https://github.com/mikefarah/yq
- name: drake
  role: Pipeline toolkit that reads the configs and re-runs only targets whose parameters
    changed
  repo: https://github.com/bioinfocz/scdrake
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:55636291cf44212eecdfc4577b98139204f95c6dc521a5deb202ec5bb59c6a8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - R
  - drake
  - scdrake
  - yq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# yaml-config-driven-pipeline-parameterization

## Summary

Parameterize scdrake's single-sample and integration pipelines by editing per-stage YAML config files (main.yaml, pipeline.yaml, contrasts.yaml, etc.) so that drake re-runs only the targets whose parameters changed, while skipping up-to-date stages.

## When to use

Use when adapting or re-running a scdrake scRNA-seq/SRT pipeline run for new parameters or cell groupings — e.g. changing a MAD threshold, restricting DRAKE_TARGETS, or setting CONTRASTS_SOURCES in contrasts.yaml.

## When NOT to use

- The task requires changing pipeline logic or adding new computations rather than tuning parameters — scdrake's pipeline definition is an R/drake object meant to be extended in R, not through YAML
- The analysis is a one-off custom script outside the scdrake/drn (drake) framework, where incremental target skipping provides no benefit
- The input is raw data import itself (10x Cell Ranger output, delimited table, or SingleCellExperiment) rather than pipeline configuration — this skill covers parameterization, not data ingestion

## Inputs

- scdrake YAML config files (main.yaml, pipeline.yaml, per-stage configs such as contrasts.yaml)
- Stage parameters from upstream stages (e.g. cluster assignments from the clustering stage referenced in CONTRASTS_SOURCES)
- Environment variables to inject into config values via yq strenv()

## Outputs

- Updated YAML config files consumed by the drake plan
- A restricted or re-parameterized drake subplan (e.g. the contrasts stage plan)
- Regenerated stage outputs (e.g. contrasts HTML report and per-contrast DE tables under BASE_OUT_DIR) reflecting the new parameters

## How to apply

Load the stage-specific YAML config together with main.yaml and pipeline.yaml (e.g. contrasts.yaml for differential expression, with CONTRASTS_SOURCES pointing at cell groupings from the upstream clustering stage). Manipulate the YAML files programmatically with the yq tool, which scdrake's methods explicitly use to manipulate YAML files — e.g. in-place updates like `yq -i '.a.b[0].c = ...'` or injecting environment variables via `strenv(NAME)`. Set stage-scoping keys such as DRAKE_TARGETS in pipeline.yaml to restrict drake execution to a single stage's targets (e.g. the contrasts subplan from scdrake::get_contrasts_subplan()), and numeric thresholds directly (e.g. MAD_THRESHOLD: 2). Rationale: because drake knows the relationships between targets, only the parts of the pipeline affected by the edited parameters re-run while up-to-date targets are skipped, giving an efficient parameter-change dialogue between bioinformatician and biologist.

## Related tools

- **scdrake** (The pipeline package whose per-stage YAML configs (main.yaml, pipeline.yaml, contrasts.yaml) are edited to parameterize runs) — https://github.com/bioinfocz/scdrake
- **yq** (Command-line YAML processor used to programmatically manipulate scdrake's YAML config files) — https://github.com/mikefarah/yq
- **drake** (Pipeline toolkit that reads the configs and re-runs only targets whose parameters changed) — https://github.com/bioinfocz/scdrake

## Examples

```
NAME=mike yq -i '.a.b[0].c = strenv(NAME)' file.yaml
```

## Evaluation signals

- Edited YAML parses cleanly (e.g. readable/validatable with yq) before a pipeline run
- After a parameter change, drake re-runs only the affected targets and skips up-to-date ones — verify via drake's log/summary
- DRAKE_TARGETS in pipeline.yaml restricts execution to exactly the intended stage's targets
- New parameter values propagate to outputs — e.g. a changed MAD threshold appears as MAD_THRESHOLD: 2 in the corresponding stage report and results
- Contrast outputs cover exactly the comparisons defined in CONTRASTS_SOURCES

## Limitations

- Parameter edits trigger re-computation only within drake's dependency tracking; edits to intermediate R objects or caches bypass the config mechanism
- Docker images fix dependency versions (e.g. Bioconductor 3.21-based image jirinovo/scdrake:1.7.1); running outside Docker requires installing the full R/Bioconductor dependency stack manually
- Config keys are stage-specific (e.g. CONTRASTS_SOURCES only applies to the contrasts stage), so keys set for one stage do not affect others

## Evidence

- [readme] drake rerun efficiency: [evidence span withheld — anchor status: external_doc]
- [methods] yq for YAML manipulation: "the [yq tool](https://github.com/mikefarah/yq) used to manipulate YAML files"
- [methods] concrete parameter example: [evidence span withheld — anchor status: not_found]
- [methods] two pipelines parameterized: "`{scdrake}` offers two pipelines - one for single-sample, and second one for integration of multiple samples"
