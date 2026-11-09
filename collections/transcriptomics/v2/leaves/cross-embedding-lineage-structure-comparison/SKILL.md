---
name: cross-embedding-lineage-structure-comparison
description: Use when a scRNA-seq dataset has multiple candidate embeddings (feature
  selection x dimension reduction choices) and you must decide which embedding yields
  the optimal trajectory — e.
when_to_use_negative:
- When the trajectory inference method does not rely on a two-dimensional representation
  — Escort only considers trajectory inference approaches that use two-dimensional
  representations
- When Escort's evaluation yields no candidate embedding with a total score > 0, since
  Escort is not guaranteed to identify a recommended embedding among the choices evaluated
- When the dataset contains disconnected cell clusters or homogeneous cells lacking
  trajectory signal, so that fitting a trajectory at all is inappropriate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: Runs the three-step evaluation over candidate embeddings to score and recommend
    the embedding for trajectory construction
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Fits the trajectories/lineages on both the Escort-recommended and original
    embeddings using mclust hierarchical clusters
- name: scLANE
  role: Tests each gene for trajectory differential expression along both fitted trajectories
    to compare dynamic-gene programs
- name: Enrichr
  role: Gene set enrichment of dynamic genes using the BioPlanet 2019 collection
- name: Monocle3
  role: Source of the original default embedding used as the baseline trajectory for
    comparison
- name: R
  role: Implementation environment; Escort is an R package and Shiny app for the evaluation
  repo: https://github.com/xiaorudong/Escort
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:f802069d4499e7982d94fab3b8e82f29b271f2122172241c7a47f3199744b6ba
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
  - Monocle3
  - R
  - Slingshot
  - scLANE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-embedding-lineage-structure-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for comparing trajectory/lineage structures fitted on different embeddings of the same scRNA-seq dataset: run Escort's three-step evaluation over candidate embeddings, then fit Slingshot + scLANE on the Escort-recommended vs the original (e.g., Monocle3-default) embedding to show which recovers more biologically faithful lineages.

## When to use

Use when a scRNA-seq dataset has multiple candidate embeddings (feature selection x dimension reduction choices) and you must decide which embedding yields the optimal trajectory — e.g., comparing lineage counts and dynamic genes between Escort-recommended and original-paper embeddings before committing to pseudotime analysis.

## When NOT to use

- When the trajectory inference method does not rely on a two-dimensional representation — Escort only considers trajectory inference approaches that use two-dimensional representations
- When Escort's evaluation yields no candidate embedding with a total score > 0, since Escort is not guaranteed to identify a recommended embedding among the choices evaluated
- When the dataset contains disconnected cell clusters or homogeneous cells lacking trajectory signal, so that fitting a trajectory at all is inappropriate

## Inputs

- Normalized preprocessed scRNA-seq expression matrix (e.g., GEO accession GSE190616, Long et al. 2022 Col10a1 lineage-tracing hypertrophic chondrocytes)
- List of candidate embedding options (feature-selection x dimension-reduction combinations, e.g., 13 candidates)
- Hierarchical cluster assignments (mclust) for Slingshot input
- Reference gene set collections for enrichment (BioPlanet 2019; MSigDB mouse GO biological processes)

## Outputs

- Escort three-step evaluation scores per embedding with recommended/not-recommended classification (total score > 0)
- Fitted Slingshot trajectories (lineages) on each compared embedding
- scLANE trajectory-differential-expression gene lists per embedding (FDR < 0.01, last-segment slope > 5 or < -5)
- Enrichr/MSigDB GO biological process enrichment results for dynamic genes
- Side-by-side embedding comparison figure (e.g., figure6_embedding_comparison.png)

## How to apply

First run Escort's three-step evaluation over the candidate embedding options (e.g., 13 choices spanning PCA/t-SNE/UMAP with varying highly variable gene percentages), and select the embedding with total score > 0 — for the Long et al. Col10a1 hypertrophic chondrocyte dataset this was PCA using the top 10% highly variable genes (density cutoff 0.53; ambiguous-cell cutoff 0.029; Escort scores range from negative infinity to 2). Fit Slingshot on both the Escort-recommended embedding and the original paper's default embedding, supplying mclust hierarchical clusters as input. Run scLANE with default settings on both trajectories, calling genes significantly dynamic at FDR-adjusted p < 0.01 and filtering for enrichment at last-segment slope > 5 or < -5. Perform gene set enrichment on the dynamic genes via Enrichr (BioPlanet 2019) and GSEA Molecular Signatures Database for mouse (GO biological processes). Compare the number of lineages recovered and the scLANE differential dynamics — on this dataset the Escort-recommended embedding gave two lineages (osteoblastic fate and terminal apoptosis) that better characterized transdifferentiation, versus three lineages in the original embedding that could only characterize the osteoblastic lineage — and render the contrast as a figure (figure6_embedding_comparison.png).

## Related tools

- **Escort** (Runs the three-step evaluation over candidate embeddings to score and recommend the embedding for trajectory construction) — https://github.com/xiaorudong/Escort
- **Slingshot** (Fits the trajectories/lineages on both the Escort-recommended and original embeddings using mclust hierarchical clusters)
- **scLANE** (Tests each gene for trajectory differential expression along both fitted trajectories to compare dynamic-gene programs)
- **Enrichr** (Gene set enrichment of dynamic genes using the BioPlanet 2019 collection)
- **Monocle3** (Source of the original default embedding used as the baseline trajectory for comparison)
- **R** (Implementation environment; Escort is an R package and Shiny app for the evaluation) — https://github.com/xiaorudong/Escort

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Escort-recommended embedding receives a total score greater than zero (Escort's benchmark scoring system ranges from negative infinity to two)
- Recommended-embedding trajectory recovers the expected lineage count/structure (two lineages: osteoblastic fate and terminal apoptosis) versus the original embedding's three lineages that only characterize the osteoblastic lineage, reproducing the paper's Figure 6 contrast
- scLANE dynamic genes pass the FDR-adjusted p-value < 0.01 criterion with last-segment slope > 5 or < -5
- Enrichment on dynamic genes via Enrichr (BioPlanet 2019) and mouse GO biological processes yields biologically coherent transdifferentiation terms
- Disconnected embeddings must be immediately classified as not recommended for trajectory inference

## Limitations

- Presently, Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- Trajectory method performance is highly dataset-specific even for universal processing steps like feature selection and dimension reduction, so results do not transfer across datasets

## Evidence
