---
name: flux-profile-clustering-umap-visualization
description: Use when per-sample reaction fluxes (e.g., METAFlux compute_flux output)
  are available and the goal is to discover metabolically coherent sample subgroups,
  visualize them in UMAP, and relate them to phenotypes like glucose uptake or survival.
when_to_use_negative:
- When the analysis target is metabolic interaction among cell types within a tumor
  microenvironment — use METAFlux's single-cell workflow (compute_sc_flux / calculate_avg_exp
  with Seurat) instead of bulk-sample clustering
- When samples derive from cell lines rather than human tissue — the medium constraint
  should be cell_medium, not human_blood, before clustering fluxes
- When the goal is pathway-level activity summaries only (mean fluxes of reactions
  per pathway) rather than sample-level subgroup discovery
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: METAFlux
  role: Predicts the 13,082-reaction flux profiles (via calculate_reaction_score and
    compute_flux with human_blood medium) that serve as clustering features
  repo: https://github.com/KChen-lab/METAFlux
- name: Seurat
  role: Used to cluster the flux profiles and embed samples with UMAP for subgroup
    visualization
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Environment for running METAFlux functions, clustering, HMR_9034 extraction,
    and survival (KM/log-rank) analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:3eb1d0e689f98664b69a1254ee0a53fc3814e75582426fefb3abbca166f430fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# flux-profile-clustering-umap-visualization

## Summary

Cluster samples on METAFlux-predicted metabolic flux profiles (13,082 Human-GEM reactions) and embed them with UMAP to reveal metabolically distinct subgroups — e.g., a 'LUSC-like' LUAD cluster in TCGA — that expression data alone does not resolve.

## When to use

Use when per-sample reaction fluxes (e.g., METAFlux compute_flux output) are available and the goal is to discover metabolically coherent sample subgroups, visualize them in UMAP, and relate them to phenotypes like glucose uptake or survival.

## When NOT to use

- When the analysis target is metabolic interaction among cell types within a tumor microenvironment — use METAFlux's single-cell workflow (compute_sc_flux / calculate_avg_exp with Seurat) instead of bulk-sample clustering
- When samples derive from cell lines rather than human tissue — the medium constraint should be cell_medium, not human_blood, before clustering fluxes
- When the goal is pathway-level activity summaries only (mean fluxes of reactions per pathway) rather than sample-level subgroup discovery

## Inputs

- METAFlux flux matrix (samples x 13,082 Human-GEM reaction flux scores), computed with compute_flux using the human_blood medium
- Normalized, non-negative bulk RNA-seq gene x sample expression matrix with human gene symbols as row names (TCGA LUAD and LUSC)
- Sample-level clinical metadata (cancer subtype labels, survival times/status) for cluster annotation and Kaplan-Meier analysis
- nutrient_lookup_files (1,648 exchange reactions) for identifying reaction IDs such as HMR_9034 (glucose uptake)

## Outputs

- Two-cluster sample assignments based on flux profiles, with the LUSC-like LUAD subgroup identified
- UMAP embedding of samples colored by cluster / cancer type (PNG)
- Per-sample glucose uptake flux table for HMR_9034 with LUSC > LUAD comparison
- Kaplan-Meier survival curves comparing LUAD1 vs LUSC-like LUAD clusters with log-rank P value (expected P = 0.0084)
- Exported flux matrix CSV

## How to apply

First generate the flux matrix: compute MRAS from the gene x sample expression matrix (human gene symbols as row names) with calculate_reaction_score, then predict fluxes for all 13,082 Human-GEM reactions using compute_flux with the human_blood medium (64 blood metabolites) for human-derived samples such as TCGA tumors; optionally apply cubic-root normalization cbrt(x) = sign(x)*abs(x)^(1/3). Next, cluster samples into two clusters based on their full flux profiles (e.g., in R with Seurat), and embed with UMAP to visualize whether the clusters separate by cancer type, flagging any LUAD samples that co-cluster with LUSC as the 'LUSC-like' LUAD group. To validate biological relevance, extract the glucose uptake exchange reaction HMR_9034 (found via nutrient_lookup_files, which contains 1,648 exchange reactions) and test glucose uptake flux differences between clusters, then run Kaplan-Meier survival analysis comparing the metabolically active LUAD1 subgroup against LUSC-like LUADs. In the TCGA LUAD/LUSC application, this reproduced a LUSC > LUAD glucose uptake difference and a significant survival separation (log-rank P = 0.0084).

## Related tools

- **METAFlux** (Predicts the 13,082-reaction flux profiles (via calculate_reaction_score and compute_flux with human_blood medium) that serve as clustering features) — https://github.com/KChen-lab/METAFlux
- **Seurat** (Used to cluster the flux profiles and embed samples with UMAP for subgroup visualization) — https://github.com/KChen-lab/METAFlux
- **R** (Environment for running METAFlux functions, clustering, HMR_9034 extraction, and survival (KM/log-rank) analysis)

## Examples

```
scores <- calculate_reaction_score(bulk_test_example); flux <- compute_flux(mras=scores, medium = human_blood)
```

## Evaluation signals

- The two-cluster solution on flux profiles places TCGA LUSC samples predominantly in a distinct cluster, revealing a 'LUSC-like' LUAD subgroup within the other cluster
- Glucose uptake flux (HMR_9034) is significantly higher in the LUSC-like cluster than in LUAD1, consistent with the reported LUSC > LUAD GUF comparison
- Kaplan-Meier log-rank test comparing LUAD1 vs LUSC-like LUAD clusters reproduces the reported P = 0.0084
- Flux matrix exported as CSV contains 13,082 reaction columns per bulk sample
- UMAP embedding shows spatial separation of the two clusters consistent with their flux-based assignments

## Limitations

- No version numbers are stated for METAFlux, R, or Seurat in the article or README, so reproduction may require pinning environments empirically
- METAFlux input requires a non-negative gene x sample matrix with human gene symbols as row names, and only one medium file (cell_medium or human_blood) can be used per model run; wrong medium choice will distort flux profiles and hence clustering
- The objective function maximizes the generic biomass reaction (biomass_human from Human1); predicted glucose uptake did not correlate as consistently as other metabolites in the NCI-60 panel evaluation

## Evidence

- [methods] Clustering METAFlux flux profiles of TCGA LUAD/LUSC samples identified a metabolically more active 'LUSC-like' LUAD subgroup with higher glucose uptake: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [methods] Seurat is the interface used for the flux-based clustering and UMAP embedding of samples: "Here we only demonstrate how to interface with Seurat object for  one **patient sample**"
- [readme] METAFlux derives the 13,082 fluxes that constitute the feature space for clustering: [evidence span withheld — anchor status: external_doc]
- [readme] osqp is a required dependency for the optimization underlying flux computation: [evidence span withheld — anchor status: external_doc]
