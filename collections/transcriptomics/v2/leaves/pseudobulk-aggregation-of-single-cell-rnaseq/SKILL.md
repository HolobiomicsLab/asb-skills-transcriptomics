---
name: pseudobulk-aggregation-of-single-cell-rnaseq
description: Use when single-cell scRNA-seq counts must be aggregated per cell type
  into pseudobulk profiles as input for pathway/cytokine activity scoring (decoupleR
  mlm) or pseudobulk-level differential/statistical tests.
when_to_use_negative:
- The analysis requires single-cell resolution or intracellular process inference
  (e.g., NicheNet-, Cytotalk-, or SoptSC-style methods), which are not directly comparable
  to the pseudobulk activity-scored framework
- The question targets long-distance endocrine signalling, which scRNA-seq gene expression
  within the dataset does not capture
- No cell-type labels are available, so counts cannot be aggregated into meaningful
  per-cell-type pseudobulk profiles
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: OmnipathR
  role: Retrieves cytokine-receptor interactions and cytokine target gene sets from
    OmniPath to serve as decoupleR input for scoring activities on the pseudobulk
    profiles
  repo: https://github.com/saezlab/OmnipathR
- name: decoupleR
  role: Scores cytokine activities on the pseudobulk profiles using its multivariate
    linear model (mlm), yielding an activity estimate and p-value per cytokine per
    cell type
  repo: https://github.com/saezlab/decoupleR
- name: LIANA
  role: Ranks cytokine-receptor interactions per cell-type pair (with CellChat/CellPhoneDB-style
    post-filtering) so overlap with significant cytokine activities can be tested
    over cumulative rank intervals
  repo: https://github.com/saezlab/liana
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:047171a7eaf1977a46fc2a4cf620df64ab977ad71472b03117f7ed0bd85670ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/OmnipathR
  tools:
  - CellChat
  - CellPhoneDB
  - LIANA
  - OmnipathR
  - decoupleR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pseudobulk-aggregation-of-single-cell-rnaseq

## Summary

Collapses single-cell RNA-seq count matrices (e.g., HER2+ and triple-negative breast cancer datasets) into per-cell-type pseudobulk expression profiles so downstream methods such as decoupleR's mlm can score cytokine activities and evaluate agreement with prioritised cell-cell communication (CCC) predictions.

## When to use

Use when single-cell scRNA-seq counts must be aggregated per cell type into pseudobulk profiles as input for pathway/cytokine activity scoring (decoupleR mlm) or pseudobulk-level differential/statistical tests.

## When NOT to use

- The analysis requires single-cell resolution or intracellular process inference (e.g., NicheNet-, Cytotalk-, or SoptSC-style methods), which are not directly comparable to the pseudobulk activity-scored framework
- The question targets long-distance endocrine signalling, which scRNA-seq gene expression within the dataset does not capture
- No cell-type labels are available, so counts cannot be aggregated into meaningful per-cell-type pseudobulk profiles

## Inputs

- scRNA-seq count matrices (e.g., HER2+ and triple-negative breast cancer single-cell count matrices)
- Cell-type annotation labels per cell
- Cytokine-receptor interactions from OmniPath (via OmnipathR)
- Cytokine target gene sets from OmniPath (via OmnipathR)
- Ranked ligand-receptor interactions from LIANA per cell-type pair

## Outputs

- Pseudobulk expression profile matrix (one column/row per cell type)
- decoupleR mlm output: activity estimate and p-value per cytokine per cell type
- Fisher's exact test odds ratios per cumulative rank interval (100-10000) per dataset
- fig6a_odds_ratios.csv (odds-ratio table for the Fig 6A panel)

## How to apply

First, aggregate the scRNA-seq count matrices (here the HER2+ and triple-negative breast cancer datasets) into pseudobulk expression profiles by summing counts per cell type, using cell-type labels from the single-cell analysis. Next, retrieve cytokine-receptor interactions and cytokine target gene sets from OmniPath via OmnipathR so the pseudobulk profiles can be scored as decoupleR input. Run decoupleR's multivariate linear model (mlm) on the pseudobulk profiles, which yields an activity estimate and p-value per cytokine per cell type. To judge the biological relevance of these activity scores, rank cytokine-receptor interactions with the LIANA framework per cell-type pair (applying the CellChat/CellPhoneDB post-filtering behaviour that removes interactions with insufficiently expressed receptor-complex subunits or transmitter/receiver genes), bin ranked interactions into cumulative rank intervals spanning 100-10000, and compute Fisher's exact test odds ratios per rank interval. Correct application is signalled by generally positive odds ratios between highly ranked cytokine-receptor interactions and cytokines with significant mlm activity at the top ranks.

## Related tools

- **OmnipathR** (Retrieves cytokine-receptor interactions and cytokine target gene sets from OmniPath to serve as decoupleR input for scoring activities on the pseudobulk profiles) — https://github.com/saezlab/OmnipathR
- **decoupleR** (Scores cytokine activities on the pseudobulk profiles using its multivariate linear model (mlm), yielding an activity estimate and p-value per cytokine per cell type) — https://github.com/saezlab/decoupleR
- **LIANA** (Ranks cytokine-receptor interactions per cell-type pair (with CellChat/CellPhoneDB-style post-filtering) so overlap with significant cytokine activities can be tested over cumulative rank intervals) — https://github.com/saezlab/liana

## Examples

```
decoupleR::run_mlm(mat = pseudobulk_counts, regulons = omni_cytokine_targets)  # activity estimate + p-value per cytokine per cell type
```

## Evaluation signals

- Pseudobulk matrix has one profile per cell type (cell types are collapsed correctly from cell-level annotations)
- decoupleR mlm returns both an activity estimate and a p-value for each cytokine in each cell type
- Fisher's exact test odds ratios between top-ranked cytokine-receptor interactions and significantly active cytokines are positive at low rank intervals across both breast cancer datasets
- Odds-ratio trends converge toward the random baseline as more interactions (higher cumulative rank bins up to 10000) are considered
- Final odds-ratio table is assembled as fig6a_odds_ratios.csv matching the Fig 6A panel structure

## Limitations

- Agreement analyses are limited by the restricted coverage of the cytokine activity signatures and receptor proteins
- Positive trends between cytokine activities and prioritised CCC interactions converge toward the random baseline as more interactions are considered, so top-rank agreement should not be extrapolated to lower ranks
- The biological ground truth for CCC is largely unknown and no gold standard is available, so agreement with cytokine activities cannot identify a definitively best method
- scRNA-seq gene expression does not capture long-distance endocrine signalling events, limiting what the pseudobulk activity scores can represent

## Evidence

- [methods] Aggregate the HER2+ and triple-negative breast cancer scRNA-seq count matrices into pseudobulk expression profiles per cell type.: [evidence span withheld — anchor status: not_found]
- [methods] Score cytokine activities on the pseudobulk profiles using decoupleR's multivariate linear model (mlm), which yields an activity estimate and p-value per cytokine per cell type.: [evidence span withheld — anchor status: not_found]
- [abstract] we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [discussion] the restricted coverage of the cytokine activity signatures and receptor proteins: "The limitations include the restricted coverage of the cytokine activity signatures and receptor proteins, and the technical shortcomings of current spatial transcriptomics technologies."
- [readme] An `R` client for the **OmniPath** web service and many other resources.: [evidence span withheld — anchor status: external_doc]
