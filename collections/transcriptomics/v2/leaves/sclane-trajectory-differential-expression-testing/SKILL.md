---
name: sclane-trajectory-differential-expression-testing
description: Use when a fitted single-cell trajectory (Slingshot lineages or Monocle3
  pseudotime on an Escort-recommended embedding) needs gene-level trajectory differential
  expression testing to identify dynamically expressed genes along lineages.
when_to_use_negative:
- Escort has classified the embedding as not recommended for trajectory inference
  (e.g. a disconnected embedding) — fix the embedding choice first, since downstream
  scLANE dynamics inherit that error
- The analysis goal is to select among candidate embeddings or decide whether a trajectory
  is appropriate at all — use Escort's three-step evaluation rather than running scLANE
  directly
- The trajectory inference approach does not use a two-dimensional representation;
  Escort's evaluation pipeline (and hence this downstream workflow) only considers
  two-dimensional representations
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLANE
  role: Tests each gene along the fitted trajectory for trajectory differential expression
    under default settings; dynamic genes are called at FDR-adjusted p-value < 0.01
- name: Slingshot
  role: Provides the fitted lineages/pseudotime over which scLANE tests each gene
    for differential dynamics
- name: Escort
  role: Supplies the data-driven recommended embedding (total score > 0) on which
    trajectories are fitted before scLANE testing
  repo: https://github.com/xiaorudong/Escort
- name: Enrichr
  role: Performs gene set enrichment on the scLANE dynamic genes using the BioPlanet
    2019 collection
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:ddc746e68bde11e3ed068f074b617027de0f9ba1e998200a3b0c81e80453ab3f
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
  - scLANE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sclane-trajectory-differential-expression-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs scLANE on fitted Slingshot/Monocle3 trajectories (built on an Escort-recommended embedding) to test each gene for trajectory differential expression, then enriches the resulting dynamic genes via Enrichr (BioPlanet 2019) and MSigDB GO biological processes for mouse. This links embedding-choice decisions to gene-level dynamic programs, as in the Long et al. Col10a1 hypertrophic chondrocyte case study.

## When to use

Use when a fitted single-cell trajectory (Slingshot lineages or Monocle3 pseudotime on an Escort-recommended embedding) needs gene-level trajectory differential expression testing to identify dynamically expressed genes along lineages.

## When NOT to use

- Escort has classified the embedding as not recommended for trajectory inference (e.g. a disconnected embedding) — fix the embedding choice first, since downstream scLANE dynamics inherit that error
- The analysis goal is to select among candidate embeddings or decide whether a trajectory is appropriate at all — use Escort's three-step evaluation rather than running scLANE directly
- The trajectory inference approach does not use a two-dimensional representation; Escort's evaluation pipeline (and hence this downstream workflow) only considers two-dimensional representations

## Inputs

- Fitted trajectory object (Slingshot lineages supplied with mclust hierarchical clusters, or Monocle3 pseudotime)
- Escort-recommended low-dimensional embedding (e.g. PCA on the top 10% of highly variable genes, total score > 0)
- Normalized scRNA-seq expression matrix (e.g. LogNormalize output from Seurat's NormalizeData(), from GEO accession GSE190616)

## Outputs

- Per-gene scLANE trajectory differential expression test statistics with FDR-adjusted p-values
- Dynamic gene list: genes significant at FDR-adjusted p-value < 0.01 with last-segment slope > 5 or < -5
- Gene set enrichment results from Enrichr (BioPlanet 2019 collection) and GSEA MSigDB GO biological processes for mouse
- Embedding comparison figure of lineage number and differential dynamics (e.g. figure6_embedding_comparison.png)

## How to apply

First obtain trajectories fitted with Slingshot (using mclust hierarchical clusters as input) on the Escort-recommended embedding — the embedding with total score > 0, e.g. PCA on the top 10% highly variable genes in the Long et al. Col10a1 case — since processing choices for dimension reduction and highly variable gene selection impacted trajectory accuracy more than the choice between Slingshot and Monocle3. Run scLANE with default settings on each trajectory to test every gene for trajectory differential expression, calling genes significantly dynamic at FDR-adjusted p-value < 0.01. Filter the significant dynamic genes for enrichment at the last segment with slope > 5 or < -5 to isolate genes enriched toward trajectory endpoints. Perform gene set enrichment on the dynamic genes using Enrichr with the BioPlanet 2019 collection and the GSEA Molecular Signatures Database for mouse (GO biological processes). When comparing embeddings, contrast the number of lineages recovered and the scLANE differential dynamics between the Escort-recommended and original embeddings to render a comparison figure (e.g. figure6_embedding_comparison.png).

## Related tools

- **scLANE** (Tests each gene along the fitted trajectory for trajectory differential expression under default settings; dynamic genes are called at FDR-adjusted p-value < 0.01)
- **Slingshot** (Provides the fitted lineages/pseudotime over which scLANE tests each gene for differential dynamics)
- **Escort** (Supplies the data-driven recommended embedding (total score > 0) on which trajectories are fitted before scLANE testing) — https://github.com/xiaorudong/Escort
- **Enrichr** (Performs gene set enrichment on the scLANE dynamic genes using the BioPlanet 2019 collection)

## Evaluation signals

- All reported dynamic genes pass the FDR-adjusted p-value < 0.01 criterion from scLANE
- Retained genes show last-segment slope > 5 or < -5 (endpoint enrichment filter)
- Enrichment results derive from BioPlanet 2019 (Enrichr) and GSEA MSigDB GO biological processes for mouse
- The Escort embedding supplying the trajectory has total score > 0 (e.g. density cutoff 0.53, ambiguous-cell cutoff 0.029 in the Long et al. case)
- Comparing scLANE differential dynamics across embeddings reproduces the paper's Figure 6 contrast (two lineages: osteoblastic fate and terminal apoptosis, vs three lineages)

## Limitations

- The slope > 5 / < -5 last-segment filter restricts reported dynamic genes to those strongly enriched at trajectory endpoints; genes dynamic only in intermediate segments may be excluded
- Downstream dynamic-gene results are sensitive to upstream embedding and preprocessing choices — quality control steps may also significantly impact Escort's embedding evaluations, which feed the trajectories scLANE tests
- Escort is not guaranteed to identify a recommended embedding among evaluated choices, in which case there is no validated embedding on which to fit trajectories for scLANE testing

## Evidence

- [methods] Run scLANE with default settings on both trajectories to test each gene for trajectory differential expression, calling genes significantly dynamic at FDR-adjusted p-value < 0.01: [evidence span withheld — anchor status: not_found]
