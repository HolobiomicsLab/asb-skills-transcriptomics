---
name: ccc-method-agreement-evaluation
description: Use when you have scRNA-seq data with cell-type annotations and want
  to validate that a CCC method's top-ranked ligand-receptor interactions are coherent
  with an independent modality such as predicted cytokine activity.
when_to_use_negative:
- When a spatial transcriptomics or receptor protein abundance modality is the intended
  validation target instead of cytokine activities — the paper evaluates those modalities
  separately
- When the goal is to determine which CCC method is best — the article states no gold
  standard exists, so agreement analyses cannot identify a definitive best method
- When studying long-distance endocrine signalling, since scRNA-seq gene expression
  does not capture such events
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: LIANA
  role: Ranks cytokine-receptor interactions per cell-type pair across CCC methods
    and applies CellChat/CellPhoneDB post-filtering
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: Retrieves cytokine-receptor interactions and cytokine target gene sets from
    OmniPath as decoupleR input
- name: decoupleR
  role: Scores cytokine activities on pseudobulk profiles using the multivariate linear
    model (mlm)
- name: CellChat
  role: Source of post-filtering behaviour (removing interactions with insufficiently
    expressed transmitter/receiver genes) applied before ranking
- name: CellPhoneDB
  role: Source of post-filtering behaviour (minimum expression of heteromeric complex
    subunits) applied before ranking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:bf84e3f3783347c19e19b3c18158965da525928c1c87c7b6578a3e1a92ab6ddb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - LIANA
  - OmnipathR
  - decoupleR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ccc-method-agreement-evaluation

## Summary

Evaluates whether ligand-receptor interactions prioritised by CCC inference methods (ranked via LIANA with CellChat/CellPhoneDB post-filtering) agree with decoupleR-inferred cytokine activities in the HER2+ and triple-negative breast cancer scRNA-seq datasets, quantified by Fisher's exact test odds ratios over cumulative rank intervals.

## When to use

Use when you have scRNA-seq data with cell-type annotations and want to validate that a CCC method's top-ranked ligand-receptor interactions are coherent with an independent modality such as predicted cytokine activity.

## When NOT to use

- When a spatial transcriptomics or receptor protein abundance modality is the intended validation target instead of cytokine activities — the paper evaluates those modalities separately
- When the goal is to determine which CCC method is best — the article states no gold standard exists, so agreement analyses cannot identify a definitive best method
- When studying long-distance endocrine signalling, since scRNA-seq gene expression does not capture such events

## Inputs

- scRNA-seq count matrices for HER2+ and triple-negative breast cancer datasets with cell-type annotations
- Pseudobulk expression profiles per cell type
- Cytokine-receptor interactions from OmniPath (retrieved via OmnipathR)
- Cytokine target gene sets from OmniPath (decoupleR input)

## Outputs

- fig6a_odds_ratios.csv — Fisher's exact test odds-ratio table per method, dataset, and cumulative rank interval
- Fig 6A odds-ratio matrix/panel visualisation
- Per-cytokine, per-cell-type mlm activity estimates and p-values
- LIANA-ranked cytokine-receptor interaction lists per cell-type pair

## How to apply

Aggregate the HER2+ and triple-negative breast cancer scRNA-seq count matrices into pseudobulk expression profiles per cell type. Retrieve cytokine-receptor interactions and cytokine target gene sets from OmniPath via OmnipathR, then score cytokine activities on the pseudobulk profiles with decoupleR's multivariate linear model (mlm), yielding an activity estimate and p-value per cytokine per cell type. Rank cytokine-receptor interactions with the LIANA framework per cell-type pair, applying CellChat/CellPhoneDB post-filtering behaviour (removing interactions with insufficiently expressed receptor-complex subunits or transmitter/receiver genes) before ranking. Bin the ranked interactions into cumulative rank intervals spanning 100-10000, tabulate overlap between highly ranked interactions and cytokines with significant mlm activity, and compute Fisher's exact test odds ratios per interval per dataset. The rationale is that agreement with cytokine activities serves as an orthogonal validation since no gold standard exists for CCC; expect generally positive trends at the top ranks that converge toward the random baseline as more interactions are considered. Assemble the odds-ratio matrix (Fig 6A) and save it as fig6a_odds_ratios.csv.

## Related tools

- **LIANA** (Ranks cytokine-receptor interactions per cell-type pair across CCC methods and applies CellChat/CellPhoneDB post-filtering) — https://github.com/saezlab/liana
- **OmnipathR** (Retrieves cytokine-receptor interactions and cytokine target gene sets from OmniPath as decoupleR input)
- **decoupleR** (Scores cytokine activities on pseudobulk profiles using the multivariate linear model (mlm))
- **CellChat** (Source of post-filtering behaviour (removing interactions with insufficiently expressed transmitter/receiver genes) applied before ranking)
- **CellPhoneDB** (Source of post-filtering behaviour (minimum expression of heteromeric complex subunits) applied before ranking)

## Examples

```
cytokine_acts <- run_mlm(mat=pseudobulk, densi=cytokine_targets); lr_ranks <- liana_wrap(seu, method='natmi', resource='OmniPath'); fisher_or <- enrich_rank_bins(lr_ranks, cytokine_acts, bins=seq(100,10000))
```

## Evaluation signals

- Odds ratios per cumulative rank interval (100-10000) are generally positive for top-ranked interactions across all CCC methods
- Odds-ratio trends converge toward the random baseline as more interactions are included — deviations from this pattern suggest a pipeline error
- mlm step yields a finite activity estimate and p-value for each cytokine per cell type on both breast cancer datasets
- Filtered LIANA rankings contain no interactions with insufficiently expressed receptor-complex subunits or transmitter/receiver genes
- Output table matches the fig6a_odds_ratios.csv schema with one odds ratio per method x dataset x rank interval

## Limitations

- No gold standard exists and the biological ground truth for CCC is largely unknown, so agreement cannot identify a definitive best method
- Agreement analyses are limited by restricted coverage of the cytokine activity signatures and receptor proteins
- scRNA-seq gene expression does not capture long-distance endocrine signalling events or processes preceding transmitter-receiver interactions such as secretion and diffusion

## Evidence

- [abstract] we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [other] mlm scoring workflow: [evidence span withheld — anchor status: not_found]
- [intro] CellPhoneDB filtering: "CellPhoneDBv2#8CellPhoneDB(1) Truncated Mean—average expression of transmitter and receivers, the minimum expression (by default) of heteromeric complex of subunits"
- [discussion] no gold standard: [evidence span withheld — anchor status: not_found]
- [discussion] limited coverage caveat: "The limitations include the restricted coverage of the cytokine activity signatures and receptor proteins, and the technical shortcomings of current spatial transcriptomics technologies."
