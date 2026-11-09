---
name: embedding-connectivity-evaluation
description: Use when candidate 2D embeddings (e.g., PCA, t-SNE, UMAP, diffusion maps
  of scRNA-seq data) must be screened for disconnected cell clusters before being
  accepted for trajectory inference scoring in Escort.
when_to_use_negative:
- When the representation is not a two-dimensional embedding — Escort only considers
  trajectory inference approaches that use two-dimensional representations, so higher-dimensional
  or velocity-derived embeddings are out of scope.
- When the goal is to fit or score the trajectory itself (pseudotime estimation) rather
  than to gate which embeddings are admissible — this skill is only the connectivity
  screening step.
- When the task is choosing an upstream QC or normalization method — those steps are
  assumed fixed here, even though they can significantly impact Escort's embedding
  evaluations.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R package providing the connectivity evaluation (LD_DCClusterscheck) that
    classifies disconnected embeddings as not recommended
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Implementation environment for the Escort package and its Shiny application
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:73f22d9b557f7affff1372674482b51be135faa3f69141e030a988fbfca8f4c2
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# embedding-connectivity-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's connectivity evaluation screens each candidate 2D embedding for newly disconnected cell clusters using Jaccard-index-based cluster comparison, so that embeddings that break the dataset's cluster structure are rejected before any trajectory scoring.

## When to use

Use when candidate 2D embeddings (e.g., PCA, t-SNE, UMAP, diffusion maps of scRNA-seq data) must be screened for disconnected cell clusters before being accepted for trajectory inference scoring in Escort.

## When NOT to use

- When the representation is not a two-dimensional embedding — Escort only considers trajectory inference approaches that use two-dimensional representations, so higher-dimensional or velocity-derived embeddings are out of scope.
- When the goal is to fit or score the trajectory itself (pseudotime estimation) rather than to gate which embeddings are admissible — this skill is only the connectivity screening step.
- When the task is choosing an upstream QC or normalization method — those steps are assumed fixed here, even though they can significantly impact Escort's embedding evaluations.

## Inputs

- Two-dimensional candidate embedding (low-dimensional projection such as PCA, t-SNE, UMAP, or diffusion maps)
- Cell cluster labels (group assignments) for the same cells
- Normalized expression data / candidate embeddings from the Escort exampleData_linear dataset (for the worked example)

## Outputs

- Per-embedding connectivity verdict (connected vs. newly disconnected clusters)
- Classification of each embedding as recommended or not recommended for trajectory inference
- Set of embeddings passing connectivity that are eligible for scoring via calcScore

## How to apply

Run the connectivity check (LD_DCClusterscheck) on each candidate embedding with its cell cluster labels: compute pairwise cluster connectivity via Jaccard-index similarity between clusters, using a 0.3 Jaccard cutoff and requiring a minimum of 10 cells per cluster in pairwise comparisons. Compare connectivity against the reference structure of the dataset so that clusters newly disconnected in the embedding are detected. Any embedding found to have newly disconnected clusters is classified immediately as not recommended for trajectory inference and excluded from the downstream calcScore combination of SimiRetain, GOF, and USHAPE metrics. The rationale is that a trajectory cannot be fit meaningfully across disconnected populations, and this check implements Escort's first, gating evaluation of whether the dataset structure supports trajectory analysis at all.

## Related tools

- **Escort** (R package providing the connectivity evaluation (LD_DCClusterscheck) that classifies disconnected embeddings as not recommended) — https://github.com/xiaorudong/Escort
- **R** (Implementation environment for the Escort package and its Shiny application)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort)
```

## Evaluation signals

- Every pairwise cluster comparison uses a Jaccard-index similarity with a 0.3 cutoff and excludes clusters with fewer than 10 cells.
- Any embedding with newly disconnected clusters is flagged not recommended and does not contribute to the calcScore total (score range negative infinity to two).
- Embeddings that pass connectivity proceed to the remaining Step 2/3 metrics (SimiRetain, GOF, USHAPE) before scoring.
- The output table should be consistent with the rule that score > 0 corresponds to Recommended, once disconnected embeddings are excluded.
- Running the Shiny app (shinyEscort) or the R vignette on exampleData_linear should reproduce the same connectivity verdicts for the bundled candidate embeddings.

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated — all candidates may fail the connectivity check.
- Quality control steps and normalization may significantly impact Escort's embedding evaluations, including connectivity results.
- Escort presently only considers trajectory inference approaches that use two-dimensional representations; RNA velocity-derived dimension reduction methods are not yet implemented.

## Evidence
