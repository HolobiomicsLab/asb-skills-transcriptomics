---
name: slingshot-pseudotime-trajectory-fitting
description: Use when a normalized scRNA-seq matrix plus cell cluster labels must
  be ordered along a continuous differentiation dynamic — after an embedding choice
  (dimension reduction + HVG selection) has been evaluated, e.g. by Escort scoring
  > 0.
when_to_use_negative:
- The candidate embedding is disconnected — Escort classifies disconnected embeddings
  as immediately not recommended for trajectory inference
- Escort's first-step assessment indicates no trajectory signal exists (disconnected
  clusters or homogeneous cells lacking trajectory signal), so fitting any pseudotime
  trajectory is inappropriate
- The trajectory method requires higher-dimensional or non-2D representations (e.g.,
  RNA velocity-derived reductions) — Escort/Skills here only consider two-dimensional
  representations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Slingshot
  role: Core trajectory/pseudotime inference method — fits simultaneous principal
    curves through mclust hierarchical clusters on the chosen 2D embedding
- name: Escort
  role: 'Pre-step: scores 13 candidate embeddings (score > 0 recommended; density
    cutoff 0.53, ambiguous-cell cutoff 0.029) to select the embedding Slingshot is
    fit on'
  repo: https://github.com/xiaorudong/Escort
- name: mclust
  role: Generates the hierarchical cluster labels supplied to Slingshot as clusterLabels
- name: scLANE
  role: 'Post-fit validation: tests each gene for trajectory differential expression
    at FDR-adjusted p < 0.01 on both Slingshot trajectories'
- name: Enrichr
  role: Gene set enrichment of dynamic genes using the BioPlanet 2019 collection to
    interpret lineage-specific dynamics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:22de78932366bded7495fe9ae06e7afb31f3a4104bf55c43840c62d5f2d7b633
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
  - Enrichr
  - Escort
  - R
  - Slingshot
  - mclust
  - scLANE
  edam_operation: http://edamontology.org/operation_3432
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

# slingshot-pseudotime-trajectory-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits Slingshot lineage trajectories and pseudotime on a pre-evaluated low-dimensional embedding of single-cell RNA-seq data (e.g., the Escort-recommended embedding), using cluster labels as the skeleton. In the hypertrophic chondrocyte case study (GSE190616), this recovered two biologically meaningful lineages where the original paper's embedding gave three, only one of which was interpretable.

## When to use

Use when a normalized scRNA-seq matrix plus cell cluster labels must be ordered along a continuous differentiation dynamic — after an embedding choice (dimension reduction + HVG selection) has been evaluated, e.g. by Escort scoring > 0.

## When NOT to use

- The candidate embedding is disconnected — Escort classifies disconnected embeddings as immediately not recommended for trajectory inference
- Escort's first-step assessment indicates no trajectory signal exists (disconnected clusters or homogeneous cells lacking trajectory signal), so fitting any pseudotime trajectory is inappropriate
- The trajectory method requires higher-dimensional or non-2D representations (e.g., RNA velocity-derived reductions) — Escort/Skills here only consider two-dimensional representations

## Inputs

- Normalized scRNA-seq expression matrix (e.g., Seurat LogNormalize output from GSE190616 preprocessed data)
- Cell cluster labels (hierarchical clusters from mclust)
- Evaluated low-dimensional embedding (2D; e.g., Escort-recommended PCA embedding on top 10% HVGs with total score > 0)

## Outputs

- Fitted lineage structure (number of lineages) with pseudotime ordering of cells along each lineage
- Trajectory comparison (e.g., figure6_embedding_comparison.png contrasting Escort-recommended vs original embedding lineages)
- Downstream scLANE dynamic-gene list (FDR < 0.01, last-segment slope > 5 or < -5) and Enrichr BioPlanet 2019 / MSigDB GO enrichment results

## How to apply

First obtain a two-dimensional embedding that supports trajectory structure: Escort classifies any embedding found to be disconnected as immediately not recommended, and recommends embeddings with a total score greater than zero (range negative infinity to two); in the Long et al. case the recommended choice was PCA on the top 10% of highly variable genes (density cutoff 0.53, ambiguous-cell cutoff 0.029). Supply Slingshot with hierarchical clusters generated with mclust as clusterLabels and fit the simultaneous principal-curve trajectory on the recommended embedding. Compare the number of lineages recovered against alternative embeddings — Escort-recommended: two lineages (osteoblastic fate and terminal apoptosis) capturing hypertrophic chondrocyte transdifferentiation; original Monocle3-default embedding: three lineages characterizing only the osteoblastic lineage. Validate dynamic genes downstream with scLANE at FDR-adjusted p < 0.01, filtering slopes > 5 or < -5 at the last segment, then run Enrichr (BioPlanet 2019) / GSEA MSigDB mouse GO biological processes. Rationale: the article's discussion notes processing choices for dimension reduction and highly variable gene inclusion impacted trajectory accuracy more than the choice between Slingshot and Monocle3, and Slingshot most accurately ordered cells in the benchmarking discussion.

## Related tools

- **Slingshot** (Core trajectory/pseudotime inference method — fits simultaneous principal curves through mclust hierarchical clusters on the chosen 2D embedding)
- **Escort** (Pre-step: scores 13 candidate embeddings (score > 0 recommended; density cutoff 0.53, ambiguous-cell cutoff 0.029) to select the embedding Slingshot is fit on) — https://github.com/xiaorudong/Escort
- **mclust** (Generates the hierarchical cluster labels supplied to Slingshot as clusterLabels)
- **scLANE** (Post-fit validation: tests each gene for trajectory differential expression at FDR-adjusted p < 0.01 on both Slingshot trajectories)
- **Enrichr** (Gene set enrichment of dynamic genes using the BioPlanet 2019 collection to interpret lineage-specific dynamics)

## Examples

```
library(slingshot); sds <- slingshot(escort_embedding, clusterLabels = mclust_clusters)
```

## Evaluation signals

- The embedding Slingshot is fit on must come from an Escort evaluation with total score > 0 (range: negative infinity to 2); disconnected embeddings are never used
- Number of lineages recovered must match the expected contrast: 2 lineages (osteoblastic fate + terminal apoptosis) on the Escort-recommended embedding vs 3 lineages on the original paper's embedding, reproducing the paper's Figure 6 comparison
- scLANE dynamic genes pass FDR-adjusted p < 0.01 with last-segment slope > 5 or < -5 filter
- Enrichr enrichment of dynamic genes returns coherent BioPlanet 2019 / GO biological process terms consistent with the transdifferentiation process
- Trajectory cells ordered continuously without disconnected or ambiguous cells breaking the principal curves

## Limitations

- Trajectory inference method performance is highly dataset-specific — even universal steps like feature selection and dimension reduction change results, so parameters do not transfer across datasets
- Escort is not guaranteed to identify a recommended embedding among the choices evaluated; if none scores > 0, Slingshot fitting should be revisited
- Escort (and this skill's embedding evaluation) only considers trajectory inference approaches using two-dimensional representations; RNA velocity-derived reductions are not yet supported
- No software version specified for Escort, Slingshot, or R in the article

## Evidence
