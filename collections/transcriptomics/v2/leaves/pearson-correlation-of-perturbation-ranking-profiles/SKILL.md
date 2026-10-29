---
name: pearson-correlation-of-perturbation-ranking-profiles
description: Use when comparing perturbation effect ranking lists produced under alternative
  MUSIC preprocessing settings (imputation on/off, Cell_filtering on/off) to measure
  ranking-list similarity and identify which strategy shifts the ranking most.
when_to_use_negative:
- When only a single MUSIC run exists — at least two ranking profiles from different
  preprocessing settings are required for any pairwise correlation
- When the question is about topic-specific rankings or perturbation-perturbation
  relationships rather than the Rank_overall overall ranking list
- When comparing non-MUSIC perturbation prioritization outputs whose score distributions
  were not normalized to a common scale first
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: MUSIC
  role: Produces the overall perturbation effect rankings via Rank_overall under each
    preprocessing ablation setting
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Data imputation step whose inclusion/exclusion defines the ablation conditions
    being compared
  repo: https://github.com/mohuangx/SAVER
- name: R
  role: Environment in which the MUSIC pipeline, normalization, and Pearson correlation
    computation are executed (R>=3.4.1)
  repo: https://github.com/bm2-lab/MUSIC
- name: devtools
  role: Used to install the MUSIC package from GitHub via install_github('bm2-lab/MUSIC')
  repo: https://github.com/bm2-lab/MUSIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:0354d782c96d29d33b1a864c2bbf0f24a4687803f837a6f8ab2e6e16d815ae10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  - devtools
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pearson-correlation-of-perturbation-ranking-profiles

## Summary

Quantify how much MUSIC's preprocessing choices (SAVER imputation, Cell_filtering, or their combination) alter the overall perturbation effect ranking by normalizing Rank_overall scores to a common scale and computing pairwise Pearson correlations between ranking profiles across ablation settings.

## When to use

Use when comparing perturbation effect ranking lists produced under alternative MUSIC preprocessing settings (imputation on/off, Cell_filtering on/off) to measure ranking-list similarity and identify which strategy shifts the ranking most.

## When NOT to use

- When only a single MUSIC run exists — at least two ranking profiles from different preprocessing settings are required for any pairwise correlation
- When the question is about topic-specific rankings or perturbation-perturbation relationships rather than the Rank_overall overall ranking list
- When comparing non-MUSIC perturbation prioritization outputs whose score distributions were not normalized to a common scale first

## Inputs

- crop_stimulated expression_profile.txt and perturb_information.txt (control labelled 'CTRL', ~36722 genes x 3259 cells)
- Four overall perturbation effect ranking lists from Rank_overall (baseline, imputation-only, filtering-only, combination ablations)

## Outputs

- Pairwise Pearson correlation matrix of normalized ranking profiles across ablation settings
- Average ranking similarity value (expected ~0.6) and per-strategy ranking-shift magnitudes
- ablation_ranking_correlations.csv with all correlations and summary values

## How to apply

Run the MUSIC baseline pipeline on the crop_stimulated data — Input_preprocess, Cell_qc, Data_imputation with SAVER (cpu_num=30), Cell_filtering with cell_num_threshold=20, Get_high_varGenes, Get_topics, Select_topic_number, Diff_topic_distri, Rank_overall — to obtain the reference overall ranking. Then produce three ablation rankings: imputation-only (skip Cell_filtering), filtering-only (skip Data_imputation), and combination (skip both). Normalize all four overall ranking scores to a common scale and compute pairwise Pearson correlations between the ranking profiles. Average the correlations and compare ranking-shift magnitudes across strategies; expect a mean similarity near 0.6 across datasets, with the combination (no imputation, no filtering) strategy perturbing the ranking list the most. Write per-pair correlations and summary values to ablation_ranking_correlations.csv.

## Related tools

- **MUSIC** (Produces the overall perturbation effect rankings via Rank_overall under each preprocessing ablation setting) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Data imputation step whose inclusion/exclusion defines the ablation conditions being compared) — https://github.com/mohuangx/SAVER
- **R** (Environment in which the MUSIC pipeline, normalization, and Pearson correlation computation are executed (R>=3.4.1)) — https://github.com/bm2-lab/MUSIC
- **devtools** (Used to install the MUSIC package from GitHub via install_github('bm2-lab/MUSIC')) — https://github.com/bm2-lab/MUSIC

## Examples

```
crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F); set.seed(234); crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30)
```

## Evaluation signals

- Mean Pearson correlation of ranking profiles across ablation settings should be close to the reported ~0.6 across the 14 datasets
- The combination ablation (no imputation and no Cell_filtering) should exhibit the largest shift in the overall ranking list
- All four ranking profiles must be normalized to a common scale before computing correlations
- The control sample label must be 'CTRL' in perturb_information, otherwise preprocessing fails or rankings are invalid
- ablation_ranking_correlations.csv should contain every pairwise correlation (4 settings -> 6 pairs) plus the aggregated similarity value

## Limitations

- The ~0.6 similarity is an average across 14 datasets; individual dataset correlations may deviate substantially
- Ranking similarity depends on Cell_filtering's cell_num_threshold=20 and SAVER's stochastic imputation; set.seed(234) is needed for reproducibility
- MUSIC's filtering steps may need future refinement — ranking changes may partly reflect noise-removal artifacts rather than biology

## Evidence

- [results] Imputation, filtering, and their combination each change the overall perturbation effect ranking lists with an average similarity of about 0.6 across the 14 datasets: [evidence span withheld — anchor status: not_found]
- [methods] Data imputation with SAVER and cpu_num=30: "crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30)"
- [methods] Cell_filtering with cell_num_threshold=20: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
- [discussion] MUSIC adopted SAVER, a R package for single-cell RNA-seq data imputation: "To improve the quality of data, MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation"
- [readme] Control sample must be labelled CTRL before running the pipeline: "**Attention:**  **（1）** The label of the control sample needs to be "CTRL"."
