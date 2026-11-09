---
name: perturbation-zero-expression-rate-quantification
description: 'Use when auditing MUSIC-style single-cell CRISPR screening pipelines:
  before or after the perturbation-filtering step, to identify which perturbations
  (e.'
when_to_use_negative:
- The goal is off-target or sgRNA knockout-efficiency evaluation rather than assessing
  the perturbation-level zero-expression filter (MUSIC handles those via CRISPRseek
  and separate sgRNA evaluation steps).
- The perturbation target gene is nonzero in at least some cells in every dataset,
  so the filter removes nothing and the with-/without-filter comparison is moot.
- Input data is fastq or raw 10X output that has not yet been converted to MUSIC's
  expression_profile.txt / perturb_information.txt format via Input_preprocess() or
  Input_preprocess_10X().
tools:
- name: MUSIC
  role: Provides Input_preprocess(), Cell_qc, Data_imputation, Cell_filtering, Get_topics/Select_topic_number,
    Diff_topic_distri, Rank_overall and Rank_specific needed to produce comparable
    rankings with and without the zero-expression filter
  repo: https://github.com/bm2-lab/MUSIC
- name: R
  role: Language in which MUSIC is implemented and in which the zero_rate statistic
    and Pearson rank correlations are computed
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Imputation step (Data_imputation) applied before cell filtering and topic
    modeling when reproducing the pipeline on affected K562 and MCF10A datasets
  repo: https://github.com/mohuangx/SAVER
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:152a5d397e4c14597e3735b2d787b6bd9a0c1ba826eb1565dcc1f6734a7718f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# perturbation-zero-expression-rate-quantification

## Summary

Quantifies, for each of the 326 knockout/knockdown perturbations across 14 datasets, the zero_rate statistic — the fraction of cells in which the targeted gene has zero expression — and evaluates whether MUSIC's rule of dropping perturbations whose targeted gene is zero in all cells materially changes the perturbation effect rankings.

## When to use

Use when auditing MUSIC-style single-cell CRISPR screening pipelines: before or after the perturbation-filtering step, to identify which perturbations (e.g. CDKN2A, PTPRD, IER3IP1) are removed by zero-expression filtering and to test whether downstream Rank_overall/Rank_specific outputs are robust to that filter.

## When NOT to use

- The goal is off-target or sgRNA knockout-efficiency evaluation rather than assessing the perturbation-level zero-expression filter (MUSIC handles those via CRISPRseek and separate sgRNA evaluation steps).
- The perturbation target gene is nonzero in at least some cells in every dataset, so the filter removes nothing and the with-/without-filter comparison is moot.
- Input data is fastq or raw 10X output that has not yet been converted to MUSIC's expression_profile.txt / perturb_information.txt format via Input_preprocess() or Input_preprocess_10X().

## Inputs

- expression_profile.txt gene-by-cell count matrix for each of the 14 screen datasets
- perturb_information.txt cell-to-perturbation assignment table with control label 'CTRL'
- MUSIC preprocessing output (crop_seq_list from Input_preprocess())
- MUSIC perturbation ranking lists from Rank_overall/Rank_specific (with and without zero-expression filtering)

## Outputs

- zero_rate table with entries for all 326 perturbations across the 14 datasets, with all-zero genes flagged
- List of perturbations removed by zero-expression filtering (expected: CDKN2A, PTPRD, IER3IP1)
- Pearson correlation values between with-filter and without-filter perturbation rankings per affected dataset (expected 0.99, 0.93, 0.98)

## How to apply

Load expression_profile.txt and perturb_information.txt for each screen dataset and preprocess with MUSIC's Input_preprocess(), ensuring the control sample label is 'CTRL'. Compute a per-perturbation zero_rate as the fraction of cells with zero expression of the targeted gene across all 326 knockouts/knockdowns in the 14 datasets, flagging perturbations whose targeted gene is zero in every cell — MUSIC's rule is that 'if the corresponding gene expression values of the perturbation are all zero among all the cells, this perturbation will be filtered directly'. For affected datasets (K562, MCF10A dox/untreated), run the full MUSIC pipeline with Cell_qc (genes detected > 500, UMIs > 1000, mitochondrial percentage < 10%), SAVER Data_imputation, and Cell_filtering (cell_num_threshold = 20). Build topic models twice per dataset with Get_topics over topic_number = 4:6, select the optimal model with Select_topic_number, then derive rankings via Diff_topic_distri, Rank_overall and Rank_specific once with and once without zero-expression filtering. Judge correctness by comparing Pearson correlations of the with-/without-filter ranking lists against the reported 0.99, 0.93, and 0.98, and by confirming only CDKN2A, PTPRD, and IER3IP1 are filtered while known functional transcription factors are retained.

## Related tools

- **MUSIC** (Provides Input_preprocess(), Cell_qc, Data_imputation, Cell_filtering, Get_topics/Select_topic_number, Diff_topic_distri, Rank_overall and Rank_specific needed to produce comparable rankings with and without the zero-expression filter) — https://github.com/bm2-lab/MUSIC
- **R** (Language in which MUSIC is implemented and in which the zero_rate statistic and Pearson rank correlations are computed) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Imputation step (Data_imputation) applied before cell filtering and topic modeling when reproducing the pipeline on affected K562 and MCF10A datasets) — https://github.com/mohuangx/SAVER

## Examples

```
crop_seq_list<-Input_preprocess(expression_profile,perturb_information); zero_rate<-apply(crop_seq_list$expression_profile[target_genes,],1,function(x) mean(x==0))
```

## Evaluation signals

- The zero_rate table contains entries for all 326 knockouts/knockdowns across the 14 datasets.
- Exactly three perturbations are flagged as all-zero and removed: CDKN2A (doxorubicin-treated and untreated MCF10A), PTPRD (doxorubicin-treated MCF10A), and IER3IP1 (K562).
- Known functional transcription factors are retained after filtering (no over-removal).
- Pearson correlations between with-filter and without-filter perturbation rankings reproduce the reported values 0.99, 0.93, and 0.98 for the affected datasets.
- Cell_qc retains cells with genes detected > 500, UMIs > 1000, and mitochondrial percentage < 10% before rankings are computed.

## Limitations

- The filter only removes perturbations whose targeted gene is zero in every cell, so it detects the extreme case; partial dropout of the target gene is not assessed by this statistic.
- Article authors note that 'future improvements are expected to refine and update such filtering steps to make it more effective', so the filtering heuristics are acknowledged as provisional.
- Ranking-impact conclusions hold only for the datasets tested (K562, MCF10A with/without doxorubicin); other datasets may respond differently.

## Evidence

- [discussion] Zero-expression filter rule: "If the corresponding gene expression values of the perturbation are all zero among all the cells, this perturbation will be filtered directly"
- [other] Reproduced finding on filtered genes and ranking impact: [evidence span withheld — anchor status: not_found]
- [other] Pearson correlation values with and without filtering: [evidence span withheld — anchor status: not_found]
- [discussion] Cell QC thresholds applied before ranking comparison: "number of genes detected (default 500), number of unique molecular identifiers induced (default 1000), and percentage of mitochondrial"
- [readme] MUSIC input preprocessing requirement: "You can apply function "Input_preprocess()" to handle this data format. You can format your own data according to the provided example files."
