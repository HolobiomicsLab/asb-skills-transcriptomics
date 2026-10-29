---
name: filtering-sensitivity-assessment
description: Use when a pipeline applies an automatic filtering step (e.g., dropping
  perturbations whose target gene is zero in all cells) and you must determine whether
  that filter materially alters the final rankings or is a minor edge-case removal.
when_to_use_negative:
- When the filter's removed set is already known to be large (e.g., perturbations
  with fewer than 30 perturbed cells dominate the screen) — a sensitivity test on
  rankings is less informative than a direct report of removed perturbations.
- When the input is not Perturb-seq/CROP-seq single-cell CRISPR screening data with
  a perturbation annotation per cell, since the ranking comparison target (Rank_overall/Rank_specific)
  would not exist.
- When the control sample label cannot be set to 'CTRL', as Input_preprocess requires
  this label to identify controls.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MUSIC
  role: Provides Input_preprocess, Cell_qc, Cell_filtering, Get_topics, Select_topic_number,
    Diff_topic_distri, Rank_overall and Rank_specific used to run the pipeline twice
    (with and without the zero-expression filter) and compare rankings
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Data_imputation step applied identically in both filter conditions so ranking
    differences are attributable to the filter, not imputation variability
  repo: https://github.com/mohuangx/SAVER
- name: R
  role: Environment in which MUSIC, SAVER and the Pearson correlation computation
    between ranking lists are executed (R >= 3.4.1 with devtools)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:47857e7073178315be5e0e662127d91bee8973cb7d6e530a4ab25df7f7bb50b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# filtering-sensitivity-assessment

## Summary

A validation workflow that quantifies how strongly a candidate data-filtering rule (e.g., MUSIC's zero-expression perturbation filter) changes downstream results, by re-running the full pipeline with and without the filter and comparing perturbation effect rankings via Pearson correlation.

## When to use

Use when a pipeline applies an automatic filtering step (e.g., dropping perturbations whose target gene is zero in all cells) and you must determine whether that filter materially alters the final rankings or is a minor edge-case removal.

## When NOT to use

- When the filter's removed set is already known to be large (e.g., perturbations with fewer than 30 perturbed cells dominate the screen) — a sensitivity test on rankings is less informative than a direct report of removed perturbations.
- When the input is not Perturb-seq/CROP-seq single-cell CRISPR screening data with a perturbation annotation per cell, since the ranking comparison target (Rank_overall/Rank_specific) would not exist.
- When the control sample label cannot be set to 'CTRL', as Input_preprocess requires this label to identify controls.

## Inputs

- expression_profile.txt gene-by-cell count matrix per screen dataset
- perturb_information.txt cell-to-perturbation assignment table (control labeled 'CTRL')
- Full set of perturbations to enumerate (326 knockouts/knockdowns across 14 datasets)

## Outputs

- Zero_rate table with one entry per perturbation and flags for all-zero-expression perturbations
- List of perturbations removed by the zero-expression filter (e.g., CDKN2A in doxorubicin-treated and untreated MCF10A, PTPRD in doxorubicin-treated MCF10A, IER3IP1 in K562)
- Paired perturbation ranking lists (Rank_overall/Rank_specific) from with-filter and without-filter runs
- Pearson correlation values between with-filter and without-filter rankings per affected dataset

## How to apply

First compute a zero_rate statistic per perturbation — the fraction of cells in which the targeted gene has zero expression — across all perturbations (here 326 knockouts/knockdowns in 14 datasets), and flag perturbations whose gene expression is zero in all cells. Preprocess each dataset's expression_profile.txt and perturb_information.txt with MUSIC's Input_preprocess() (control label must be 'CTRL'), then run the standard pipeline twice: Cell_qc (genes detected > 500, UMIs > 1000, mitochondrial percentage < 10%), SAVER Data_imputation, and Cell_filtering (cell_num_threshold = 20), followed by Get_topics over topic_number = 4:6, Select_topic_number, Diff_topic_distri, and Rank_overall/Rank_specific — once without the zero-expression filter and once with it. Finally compute Pearson correlations between the with-filter and without-filter perturbation ranking lists per affected dataset; small correlations between filtering status and ranking change indicate the filter is inconsequential. Rationale: MUSIC drops a perturbation directly if its gene expression is all-zero among all cells, so this assessment verifies the filter only removes true edge cases (e.g., CDKN2A, PTPRD, IER3IP1) without distorting overall perturbation prioritization.

## Related tools

- **MUSIC** (Provides Input_preprocess, Cell_qc, Cell_filtering, Get_topics, Select_topic_number, Diff_topic_distri, Rank_overall and Rank_specific used to run the pipeline twice (with and without the zero-expression filter) and compare rankings) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Data_imputation step applied identically in both filter conditions so ranking differences are attributable to the filter, not imputation variability) — https://github.com/mohuangx/SAVER
- **R** (Environment in which MUSIC, SAVER and the Pearson correlation computation between ranking lists are executed (R >= 3.4.1 with devtools))

## Examples

```
crop_seq_list<-Input_preprocess(expression_profile,perturb_information); crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F)
```

## Evaluation signals

- Zero-rate table contains an entry for all 326 perturbations, with exactly the all-zero-expression perturbations flagged (CDKN2A, PTPRD, IER3IP1).
- Filtered set matches the flagged all-zero genes while known functional transcription factors remain in the rankings.
- Pearson correlations between with-filter and without-filter rankings reproduce the reported values 0.99, 0.93, and 0.98 for the affected datasets.
- Identical Cell_qc thresholds (genes > 500, UMIs > 1000, mito % < 10), SAVER imputation, and Cell_filtering cell_num_threshold = 20 are used in both runs so the filter is the only differing condition.
- Topic model selection is restricted to topic_number = 4:6 in both runs per MUSIC's recommended scope.

## Limitations

- Only perturbations whose targeted gene is zero in all cells are removed by this filter — three perturbations total — so the sensitivity conclusion is specific to zero-expression filtering and may not hold for filters with larger removal sets.
- MUSIC authors acknowledge that current filtering steps for data noise may need future refinement, so filter behavior could change in later versions.
- Recommended topic number is restricted to a narrow scope (4 to 6), constraining the ranking comparator's stability across conditions.

## Evidence

- [discussion] Zero-expression filter definition: "If the corresponding gene expression values of the perturbation are all zero among all the cells, this perturbation will be filtered directly"
- [discussion] Cell QC thresholds: "MUSIC evaluates cell quality based on three factors29, i.e., number of genes detected (default 500), number of unique molecular identifiers induced (default 1000), and percentage of mitochondrial"
- [discussion] SAVER imputation step: "To improve the quality of data, MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation"
- [discussion] Topic number scope: "we recommended a reasonable scope (now 4 to 6) of topic model number to be tried"
- [readme] Control label requirement: "The label of the control sample needs to be "CTRL"."
