---
name: scrna-seq-imputation-ablation-with-saver
description: Use when you need to quantify how MUSIC's SAVER-based Data_imputation
  and Cell_filtering (cell_num_threshold=20) preprocessing steps change the overall
  gene perturbation effect ranking, e.g. to benchmark preprocessing sensitivity of
  a CROP-seq/Perturb-seq analysis.
when_to_use_negative:
- Input has not passed Input_preprocess()/Cell_qc(), so the ablation arms cannot share
  a common quality-controlled starting point
- The dataset is not a pooled CRISPR knockout screening dataset with per-cell sgRNA
  assignments (e.g. plain scRNA-seq without perturbation labels), since Rank_overall
  perturbation rankings cannot be produced
- You only need SAVER imputation itself as a final result rather than a sensitivity
  comparison — in that case call Data_imputation directly without the ablation design
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MUSIC
  role: Host pipeline providing Input_preprocess, Cell_qc, Data_imputation, Cell_filtering,
    Get_high_varGenes, Get_topics, Select_topic_number, Diff_topic_distri, and Rank_overall
    for the ablation arms
  repo: github.com/bm2-lab/MUSIC
- name: SAVER
  role: Imputation engine invoked by MUSIC's Data_imputation step in the imputation-on
    ablation arms
  repo: github.com/mohuangx/SAVER
- name: R
  role: Runtime environment (R>=3.4.1) in which the MUSIC package and all ablation
    pipeline calls are executed
  repo: github.com/bm2-lab/MUSIC
- name: devtools
  role: Used to install the MUSIC package from GitHub via install_github('bm2-lab/MUSIC')
  repo: github.com/bm2-lab/MUSIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:7fb6f7ea56958adf52d75a6ca9b478bf6b5325805667c854c39d45c312c53cbb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUSIC
  - R
  - SAVER
  - devtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrna-seq-imputation-ablation-with-saver

## Summary

An ablation skill that re-runs the MUSIC single-cell CRISPR screening pipeline with SAVER imputation and cell filtering toggled on/off, quantifying how each preprocessing strategy shifts the final perturbation effect ranking (average pairwise ranking similarity ≈ 0.6 across 14 datasets, with the combined skip being the most disruptive).

## When to use

Use when you need to quantify how MUSIC's SAVER-based Data_imputation and Cell_filtering (cell_num_threshold=20) preprocessing steps change the overall gene perturbation effect ranking, e.g. to benchmark preprocessing sensitivity of a CROP-seq/Perturb-seq analysis.

## When NOT to use

- Input has not passed Input_preprocess()/Cell_qc(), so the ablation arms cannot share a common quality-controlled starting point
- The dataset is not a pooled CRISPR knockout screening dataset with per-cell sgRNA assignments (e.g. plain scRNA-seq without perturbation labels), since Rank_overall perturbation rankings cannot be produced
- You only need SAVER imputation itself as a final result rather than a sensitivity comparison — in that case call Data_imputation directly without the ablation design

## Inputs

- crop_stimulated-style expression_profile.txt (genes x cells count matrix, e.g. 36722 x 3259)
- perturb_information.txt (per-cell sgRNA perturbation labels with control labelled 'CTRL')
- MUSIC pipeline objects after Input_preprocess() and Cell_qc()

## Outputs

- Four overall perturbation effect ranking lists (full baseline, imputation-only, filtering-only, neither)
- Pairwise Pearson correlation matrix of normalized ranking profiles
- ablation_ranking_correlations.csv with all correlations and summary values (mean similarity ≈ 0.6)

## How to apply

Load the CROP-seq expression profile and perturbation information with Input_preprocess(), ensuring the control sample is labelled 'CTRL', then run Cell_qc to establish a common starting point. Execute four pipeline variants: (1) full baseline with Data_imputation(SAVER, cpu_num=30) followed by Cell_filtering(cell_num_threshold=20); (2) imputation-only, skipping Cell_filtering; (3) filtering-only, skipping Data_imputation; (4) neither, skipping both. Each variant continues through Get_high_varGenes, Get_topics, Select_topic_number, Diff_topic_distri, and Rank_overall to yield its overall perturbation ranking. Normalize the ranking scores to a common scale and compute pairwise Pearson correlations between ranking profiles across the four settings, then aggregate to an average similarity (expected ≈ 0.6) and check that the combined-skip strategy shifts the ranking most. The rationale is that MUSIC adopted SAVER, an R package for single-cell RNA-seq imputation, purely to improve noisy/sparse data quality, so ablation reveals whether downstream topic-model rankings depend on that preprocessing.

## Related tools

- **MUSIC** (Host pipeline providing Input_preprocess, Cell_qc, Data_imputation, Cell_filtering, Get_high_varGenes, Get_topics, Select_topic_number, Diff_topic_distri, and Rank_overall for the ablation arms) — github.com/bm2-lab/MUSIC
- **SAVER** (Imputation engine invoked by MUSIC's Data_imputation step in the imputation-on ablation arms) — github.com/mohuangx/SAVER
- **R** (Runtime environment (R>=3.4.1) in which the MUSIC package and all ablation pipeline calls are executed) — github.com/bm2-lab/MUSIC
- **devtools** (Used to install the MUSIC package from GitHub via install_github('bm2-lab/MUSIC')) — github.com/bm2-lab/MUSIC

## Examples

```
crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30)
```

## Evaluation signals

- Mean Pearson correlation of ranking profiles across the four ablation settings is close to the reported ~0.6
- The combined-skip (no imputation, no filtering) strategy exhibits the largest shift in the overall ranking list
- All four arms produce a valid Rank_overall output from the same Cell_qc input and the control label is 'CTRL'
- Ranking scores from all settings are normalized to a common scale before correlation is computed
- All pairwise correlations and summary values are written to ablation_ranking_correlations.csv

## Limitations

- SAVER imputation is optional and time-consuming: it 'may take a little long time without parallel computation', motivating cpu_num=30
- MUSIC authors note that 'future improvements are expected to refine and update such filtering steps to make it more effective', so filtering-based ranking changes may reflect immature heuristics
- Ablation conclusions were drawn across 14 datasets with an average similarity of only ~0.6, so rankings should not be treated as invariant to preprocessing choices

## Evidence

- [discussion] SAVER imputation rationale in MUSIC: "To improve the quality of data, MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation"
- [methods] Data_imputation call with cpu_num=30: "crop_seq_imputation<-Data_imputation(crop_seq_qc$expression_profile,crop_seq_qc$perturb_information,cpu_num=30)"
- [readme] Cell_filtering with cell_num_threshold=20: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
- [readme] Control label requirement: "**Attention:**  **（1）** The label of the control sample needs to be "CTRL"."
- [discussion] Filtering steps may need future refinement: "future improvements are expected to refine and update such filtering steps to make it more effective"
