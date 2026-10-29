---
name: spatial-transcriptomics-figure-reproduction
description: Use when validating cell-cell communication predictions against spatial
  transcriptomics data — e.g., correlating LIANA-ranked LR interactions with spot-level
  cell-type colocalization (Visium/SPOTlight) or neighborhood enrichment (seqFISH/merFISH).
when_to_use_negative:
- The goal is only to rank LR interactions from scRNA-seq without any spatial validation
  — LIANA alone suffices; this skill adds the spatial agreement layer.
- No spatially resolved data (Visium, seqFISH, merFISH) or matched scRNA-seq reference
  is available, so colocalization or adjacency cannot be computed.
- Evaluating intracellular-process methods such as NicheNet, Cytotalk, or SoptSC,
  which the paper states 'are not directly comparable' in this benchmark.
tools:
- name: LIANA
  role: Ranks ligand-receptor interactions per cell-type pair across methods (CellChat,
    CellPhoneDB, Connectome, NATMI, SingleCellSignalR, logFC Mean, Crosstalk score)
    and their consensus; top-ranked pairs feed the odds-ratio computation
  repo: https://github.com/saezlab/liana
- name: SPOTlight
  role: Deconvolves 10x Visium spot proportions from the matched scRNA-seq reference
    to derive cell-type colocalization z-scores
- name: Squidpy
  role: Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data
    to identify statistically adjacent cell-type pairs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:a4bcb68a685d1ab53a05f0cd16f8378c2fbe04133c60570acf09c537cfc4e4f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - SPOTlight
  - Squidpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# spatial-transcriptomics-figure-reproduction

## Summary

Reproduces the spatial-validation analyses of the Dimitrov et al. (Nat Commun 2022) CCC benchmark, testing whether top-ranked ligand-receptor interactions from LIANA are preferentially enriched between spatially adjacent or colocalized cell types in murine brain cortex and triple-negative breast cancer 10x Visium data.

## When to use

Use when validating cell-cell communication predictions against spatial transcriptomics data — e.g., correlating LIANA-ranked LR interactions with spot-level cell-type colocalization (Visium/SPOTlight) or neighborhood enrichment (seqFISH/merFISH).

## When NOT to use

- The goal is only to rank LR interactions from scRNA-seq without any spatial validation — LIANA alone suffices; this skill adds the spatial agreement layer.
- No spatially resolved data (Visium, seqFISH, merFISH) or matched scRNA-seq reference is available, so colocalization or adjacency cannot be computed.
- Evaluating intracellular-process methods such as NicheNet, Cytotalk, or SoptSC, which the paper states 'are not directly comparable' in this benchmark.

## Inputs

- Eight 10x Visium slides (murine brain cortex and triple-negative breast cancer)
- Matched scRNA-seq reference datasets for SPOTlight deconvolution
- seqFISH dataset (spatial adjacency ground truth)
- merFISH dataset (spatial adjacency ground truth)
- Ligand-receptor resources used via LIANA (CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb)

## Outputs

- Fig 6B: spatial colocalization agreement plot
- Supplementary Fig S15: adjacency odds ratios including the merFISH null trend
- spatial_colocalization_odds_ratios.csv (per-method odds ratios)
- Pairwise cell-type colocalization z-score matrices per Visium slide
- Squidpy neighbourhood enrichment results for seqFISH/merFISH cell-type pairs

## How to apply

First deconvolve the eight 10x Visium slides (murine brain cortex and triple-negative breast cancer) with SPOTlight, using the matched scRNA-seq reference, to estimate cell-type proportions per spot. From these proportions compute pairwise cell-type colocalization z-scores per slide and call pairs colocalized at z-score >= 1.645 (one-sided 95% threshold). Independently run Squidpy neighbourhood enrichment with label permutation on the seqFISH and merFISH datasets to identify statistically adjacent cell-type pairs. Rank ligand-receptor interactions per cell-type pair with LIANA and compute odds ratios of top-ranked interactions being preferentially found between adjacent/colocalized versus non-adjacent cell-type pairs; build a label-permutation null distribution of odds ratios for merFISH to establish the expected trend under no adjacency preference. Finally plot Fig 6B (spatial colocalization agreement) and Supplementary Fig S15 (adjacency odds ratios including the merFISH null trend) and export spatial_colocalization_odds_ratios.csv.

## Related tools

- **LIANA** (Ranks ligand-receptor interactions per cell-type pair across methods (CellChat, CellPhoneDB, Connectome, NATMI, SingleCellSignalR, logFC Mean, Crosstalk score) and their consensus; top-ranked pairs feed the odds-ratio computation) — https://github.com/saezlab/liana
- **SPOTlight** (Deconvolves 10x Visium spot proportions from the matched scRNA-seq reference to derive cell-type colocalization z-scores)
- **Squidpy** (Runs neighbourhood enrichment with label permutation on seqFISH/merFISH data to identify statistically adjacent cell-type pairs)

## Examples

```
liana_rank <- liana_wrap(sce, method='natmi'); squidpy.gr.nhood_enrichment(adata, cluster_key='cell_type', n_perms=1000); # then odds ratio of top-ranked LR pairs vs z-score >= 1.645 colocalized pairs
```

## Evaluation signals

- Colocalized pairs are called at z-score >= 1.645, matching the one-sided 95% threshold; z-scores below this must be treated as non-colocalized.
- Reproduced odds ratios should show a positive trend between cell type colocalisation and prioritised CCC interactions in Visium data.
- In the murine brain cortex, all methods except the Crosstalk scores should show the colocalization association; in triple-negative breast cancer only the consensus and LogFC Mean should.
- The merFISH label-permutation null distribution should bracket odds ratios expected under no adjacency preference, and S15 should show it alongside observed odds ratios.
- spatial_colocalization_odds_ratios.csv should contain one odds ratio per CCC method per tissue with consistent method labels (including the consensus).

## Limitations

- In triple-negative breast cancer the spatial association was weak — only the consensus and LogFC Mean showed it — so a null result there is not evidence of failure.
- Agreement analyses are limited by restricted coverage of cytokine activity signatures and receptor proteins, and by technical shortcomings of current spatial transcriptomics technologies.
- Benchmarks cannot distinguish simple co-expression from actual CCC events; no gold standard exists, so a definitive best method cannot be identified.
- Gene expression is a proxy of protein levels alone and does not capture long-distance endocrine signalling or processes preceding transmitter-receiver interactions such as secretion and diffusion.

## Evidence

- [results] Positive colocalization trend across all methods (except Crosstalk) in cortex: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract: spatial agreement assessment: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [methods] Workflow: colocalization threshold definition: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: merFISH permutation null rationale: [evidence span withheld — anchor status: not_found]
- [readme] LIANA README: flexible method-resource combinations: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
