---
name: squidpy-neighbourhood-enrichment-analysis
description: Use when you have spatially resolved single-cell or spot-level data (seqFISH,
  merFISH, 10x Visium) with cell-type labels and need to test whether prioritized
  CCC interactions are enriched between spatially adjacent cell types.
when_to_use_negative:
- The data have no spatial coordinates or cell-type annotations (e.g., dissociated
  scRNA-seq alone), so adjacency cannot be assessed
- The goal is to prioritize or score ligand-receptor pairs themselves rather than
  to evaluate the spatial coherence of already-ranked CCC predictions
- Studying long-distance endocrine signalling, which scRNA-seq/spatial gene expression
  does not capture
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Squidpy
  role: Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data
    to identify statistically adjacent cell-type pairs
- name: SPOTlight
  role: Deconvolves 10x Visium spots against the scRNA-seq reference to estimate cell-type
    proportions for colocalization z-scores
- name: LIANA
  role: Ranks ligand-receptor interactions per cell-type pair that are then tested
    for spatial adjacency enrichment via odds ratios
  repo: https://github.com/saezlab/liana
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:c5efccf959401bfa9901c4838b53066d6336eead124f6e7b5faf5c503ca6f452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - SPOTlight
  - Squidpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# squidpy-neighbourhood-enrichment-analysis

## Summary

Uses Squidpy's label-permutation neighbourhood enrichment on spatial transcriptomics data (seqFISH, merFISH, Visium) to identify statistically adjacent/colocalized cell-type pairs, then tests whether top-ranked LIANA ligand-receptor interactions are preferentially found between those pairs via odds ratios.

## When to use

Use when you have spatially resolved single-cell or spot-level data (seqFISH, merFISH, 10x Visium) with cell-type labels and need to test whether prioritized CCC interactions are enriched between spatially adjacent cell types.

## When NOT to use

- The data have no spatial coordinates or cell-type annotations (e.g., dissociated scRNA-seq alone), so adjacency cannot be assessed
- The goal is to prioritize or score ligand-receptor pairs themselves rather than to evaluate the spatial coherence of already-ranked CCC predictions
- Studying long-distance endocrine signalling, which scRNA-seq/spatial gene expression does not capture

## Inputs

- seqFISH and merFISH datasets with per-cell spatial coordinates and cell-type labels
- Eight 10x Visium slides (murine brain cortex and triple-negative breast cancer)
- Matching scRNA-seq reference for SPOTlight deconvolution
- LIANA-ranked ligand-receptor interactions per cell-type pair (e.g., from CellPhoneDB truncated mean, NATMI mean-expression edge weight, iTALK logFC Mean, Cytotalk Crosstalk score, or consensus)

## Outputs

- Statistically adjacent cell-type pairs per slide from Squidpy neighbourhood enrichment
- Pairwise cell-type colocalization z-scores per Visium slide with significant pairs at z-score >= 1.645
- Odds ratios of top-ranked interactions between adjacent/colocalized vs non-adjacent cell-type pairs per CCC method
- Label-permutation null distribution of odds ratios for merFISH
- Fig 6B (spatial colocalization agreement), Supplementary Fig S15 (adjacency odds ratios with merFISH null trend), and spatial_colocalization_odds_ratios.csv

## How to apply

For Visium data, first run SPOTlight deconvolution on the eight 10x Visium slides (murine brain cortex and triple-negative breast cancer) against the matching scRNA-seq reference to estimate per-spot cell-type proportions, then compute pairwise cell-type colocalization z-scores per slide and call colocalized pairs significant at z-score >= 1.645 (one-sided 95% threshold). For seqFISH and merFISH data, load the spatial datasets into Squidpy and run neighbourhood enrichment with label permutation to identify statistically adjacent cell-type pairs. Next, rank ligand-receptor interactions per cell-type pair with LIANA and compute odds ratios of top-ranked interactions being preferentially found between adjacent (seqFISH/merFISH) or colocalized (Visium) cell-type pairs versus non-adjacent pairs. For merFISH, build a label-permutation null distribution of odds ratios to establish the expected trend under no adjacency preference. Decision point: a method is considered to show spatial coherence when its odds ratio for adjacent/colocalized pairs exceeds 1 and lies outside the permutation null trend.

## Related tools

- **Squidpy** (Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data to identify statistically adjacent cell-type pairs)
- **SPOTlight** (Deconvolves 10x Visium spots against the scRNA-seq reference to estimate cell-type proportions for colocalization z-scores)
- **LIANA** (Ranks ligand-receptor interactions per cell-type pair that are then tested for spatial adjacency enrichment via odds ratios) — https://github.com/saezlab/liana

## Examples

```
import squidpy as sq; sq.gr.nhood_enrichment(adata, cluster_key='cell_type')
```

## Evaluation signals

- Colocalized Visium pairs are called at the one-sided 95% threshold, i.e., colocalization z-score >= 1.645
- Neighbourhood enrichment on seqFISH/merFISH yields significant adjacent pairs only after label permutation, so permutation p-values are computed for every cell-type pair
- Odds ratios for top-ranked interactions should exceed 1 for methods showing spatial agreement, and the merFISH null trend from label permutation should center near odds ratio 1
- Expected pattern: in murine brain cortex all methods except the Crosstalk scores show the association; in triple-negative breast cancer only the consensus and LogFC Mean do
- The odds-ratio table is written as spatial_colocalization_odds_ratios.csv with one odds ratio per CCC method per tissue/dataset

## Limitations

- Agreement analyses are limited by restricted coverage of cytokine activity signatures and receptor proteins, and by technical shortcomings of current spatial transcriptomics technologies
- scRNA-Seq gene expression does not capture long-distance endocrine signalling events, so adjacency-based agreement misses such communication
- Benchmarks cannot distinguish simple co-expression from actual CCC events, so spatial enrichment of predictions is not proof of true communication

## Evidence

- [abstract] we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance"
- [methods] Load the seqFISH and merFISH datasets and run Squidpy neighbourhood enrichment (with label permutation) to identify statistically adjacent cell-type pairs.: [evidence span withheld — anchor status: not_found]
- [methods] call colocalized pairs as significant at z-score >= 1.645 (one-sided 95% threshold): [evidence span withheld — anchor status: not_found]
- [readme] LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus.: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
