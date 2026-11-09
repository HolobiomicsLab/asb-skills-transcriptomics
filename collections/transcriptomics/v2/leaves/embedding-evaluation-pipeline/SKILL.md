---
name: embedding-evaluation-pipeline
description: Use when a scRNA-seq dataset has multiple candidate embeddings (PCA,
  t-SNE, UMAP, diffusion maps) from varied feature-selection/dimension-reduction choices
  and you must score which are suitable for trajectory fitting with Escort.
when_to_use_negative:
- Your trajectory inference method does not rely on a two-dimensional representation
  — Escort only considers approaches using two-dimensional representations.
- You want to evaluate RNA velocity-derived dimension reduction methods — support
  is not yet implemented (future work).
- You need generic trajectory-method selection guidance without embedding-level scoring
  of preprocessing choices (e.g., dynguidelines covers method choice but does not
  account for the impact of processing steps).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Escort
  role: R package / Shiny app whose three-step framework performs trajectory suitability
    testing (Step 1), embedding evaluation and trajectory fitting (Steps 2–3) with
    scoring from negative infinity to two
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Default trajectory method used in Escort Step 3 to fit the trajectory on the
    evaluated embedding
- name: Scaffold
  role: R package v0.2.0 used to simulate benchmark scRNA-seq datasets of 1000 and
    3000 cells with 20,000 genes via estimateScaffoldParameters() on the human pancreas
    reference
- name: R
  role: Implementation environment in which Escort is run for the timing benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:a28cce919e11a3de8ba55cb883edff2d2f945aa7f82e4609e18f828dba1986c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Escort
  - R
  - Scaffold
  - Slingshot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# embedding-evaluation-pipeline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs Escort's embedding evaluation pipeline (Steps 2–3), scoring candidate dimension-reduction embeddings of a scRNA-seq dataset for trajectory-inference suitability, with benchmark runtimes of 0.4–1.5 min per embedding at 1000–3000 cells.

## When to use

Use when a scRNA-seq dataset has multiple candidate embeddings (PCA, t-SNE, UMAP, diffusion maps) from varied feature-selection/dimension-reduction choices and you must score which are suitable for trajectory fitting with Escort.

## When NOT to use

- Your trajectory inference method does not rely on a two-dimensional representation — Escort only considers approaches using two-dimensional representations.
- You want to evaluate RNA velocity-derived dimension reduction methods — support is not yet implemented (future work).
- You need generic trajectory-method selection guidance without embedding-level scoring of preprocessing choices (e.g., dynguidelines covers method choice but does not account for the impact of processing steps).

## Inputs

- Simulated or real scRNA-seq dataset (e.g., Scaffold-generated 1000 or 3000 cells x 20,000 genes from the human pancreas reference)
- One or more candidate 2D embeddings (PCA, t-SNE, UMAP, diffusion maps) reflecting different feature-selection and dimension-reduction choices
- Trajectory method choice (Slingshot is Escort's default; Monocle3 also supported)

## Outputs

- Embedding scores on Escort's benchmark scale from negative infinity to two, with score > 0 recommended for trajectory construction
- Recommended / not-recommended classification per embedding (disconnected embeddings flagged immediately as not recommended)
- Fitted trajectory and pseudotime from the default method (Slingshot)
- escort_computation_time.csv reporting mean and SD runtime (min) for Step 1 and Steps 2–3 per scenario across five repeats

## How to apply

Install Escort from GitHub (remotes::install_github("xiaorudong/Escort")) and run its three-step framework: Step 1 tests trajectory suitability independently of any embedding, while Steps 2 and 3 combined evaluate a single embedding and fit the trajectory with Slingshot as the default method. For benchmarking, simulate datasets with the Scaffold R package v0.2.0 using estimateScaffoldParameters() on the human pancreas reference, generating 1000-cell and 3000-cell datasets with 20,000 genes. Time Step 1 separately from Steps 2–3, repeat each scenario five times, and record mean and SD runtime in minutes to escort_computation_time.csv. Expected timings on laptop-class hardware (16 GB RAM) are ~5 min (1000 cells) and ~18 min (3000 cells) for Step 1, and 0.4–1.5 min per single embedding for Steps 2–3. Interpret scores on Escort's standardized scale from negative infinity to two: embeddings scoring greater than zero are recommended for trajectory construction, and any embedding found disconnected is classified immediately as not recommended.

## Related tools

- **Escort** (R package / Shiny app whose three-step framework performs trajectory suitability testing (Step 1), embedding evaluation and trajectory fitting (Steps 2–3) with scoring from negative infinity to two) — https://github.com/xiaorudong/Escort
- **Slingshot** (Default trajectory method used in Escort Step 3 to fit the trajectory on the evaluated embedding)
- **Scaffold** (R package v0.2.0 used to simulate benchmark scRNA-seq datasets of 1000 and 3000 cells with 20,000 genes via estimateScaffoldParameters() on the human pancreas reference)
- **R** (Implementation environment in which Escort is run for the timing benchmark)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- escort_computation_time.csv contains mean and SD runtime (min) for both Step 1 and Steps 2–3 across five repeats per cell-count scenario.
- Step 1 timing must be recorded independently of any embedding; Steps 2–3 timing must correspond to a single embedding.
- Per-embedding Steps 2–3 runtimes should fall roughly in the 0.4–1.5 min range for 1000–3000 cells on 16 GB laptop-class hardware.
- All reported embedding scores must lie on the standardized scale from negative infinity to two, with only scores > 0 recommended.
- Embeddings detected as disconnected must be classified as not recommended regardless of their numeric score.

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation.
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated.
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work).
- Benchmark timings are from laptop-class hardware (16 GB RAM); no software version specified for Escort or R in the article.
- Quality control steps and normalization may significantly impact Escort's embedding evaluations.

## Evidence
