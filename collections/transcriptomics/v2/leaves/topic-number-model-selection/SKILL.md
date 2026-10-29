---
name: topic-number-model-selection
description: Use when fitting graded LDA topic models to single-cell CRISPR screening
  (Perturb-seq/CROP-seq/CRISP-seq) expression data and you must choose the optimal
  topic number in a data-driven manner rather than fixing it arbitrarily.
when_to_use_negative:
- When the analysis is a standard cell-type clustering task rather than perturbation-effect
  topic modeling of single-cell CRISPR screening data — ordinary LDA topic-count heuristics
  or clustering metrics are more appropriate
- When expression values are not normalized/rounded for topic modeling or the input
  is raw FASTQ data — MUSIC requires gene expression profiles as input, not fastq
- When perturbations with fewer than 30 perturbed cells or invalid edits have not
  yet been filtered — topic modeling on unfiltered data yields unreliable topic profiles
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: MUSIC
  role: Provides Get_topics() to fit graded LDA topic models over topic_number=c(4:6)
    and Select_topic_number() to automatically pick the optimal topic number using
    topic specificity and purity scores
  repo: https://github.com/bm2-lab/MUSIC
- name: topicmodels
  role: R package underlying the graded LDA topic model fitting in MUSIC's Get_topics()
- name: SAVER
  role: Optional Data_imputation() step to recover sparse expression values before
    topic model building
  repo: https://github.com/mohuangx/SAVER
- name: clusterProfiler
  role: Used in Topic_func_anno() to annotate the function of each topic after topic
    number selection
- name: R
  role: Runtime environment; MUSIC installs via devtools::install_github under R>=3.4.1
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:67bf6f8496204783f4ce7571a0e299cac8d6e4942be86577dc60c1459ade2d7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - Diff_topic_distri
  - Get_topics
  - MUSIC
  - R
  - SAVER
  - Select_topic_number
  - clusterProfiler
  - devtools
  - topicmodels
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# topic-number-model-selection

## Summary

Data-driven selection of the optimal number of topics for graded LDA topic models fitted to single-cell CRISPR screening data in MUSIC, by scoring candidate models (topic_number = 4:6) with topic specificity, purity, and their combination.

## When to use

Use when fitting graded LDA topic models to single-cell CRISPR screening (Perturb-seq/CROP-seq/CRISP-seq) expression data and you must choose the optimal topic number in a data-driven manner rather than fixing it arbitrarily.

## When NOT to use

- When the analysis is a standard cell-type clustering task rather than perturbation-effect topic modeling of single-cell CRISPR screening data — ordinary LDA topic-count heuristics or clustering metrics are more appropriate
- When expression values are not normalized/rounded for topic modeling or the input is raw FASTQ data — MUSIC requires gene expression profiles as input, not fastq
- When perturbations with fewer than 30 perturbed cells or invalid edits have not yet been filtered — topic modeling on unfiltered data yields unreliable topic profiles

## Inputs

- Preprocessed, QC-filtered (and optionally SAVER-imputed) expression profile from crop_stimulated/crop_unstimulated-style data (genes x cells matrix)
- Perturbation information vector with control cells labeled 'CTRL'
- Candidate topic numbers, e.g. topic_number = c(4:6)

## Outputs

- Fitted graded LDA topic models (one per candidate topic number) from Get_topics()
- Optimal topic number selected by Select_topic_number() based on topic specificity, purity, and combined scores
- Optimal topic model passed downstream to Diff_topic_distri() for perturbation effect scoring (TPD/TPDS/TPDR)

## How to apply

After preprocessing (Input_preprocess with control label 'CTRL'), Cell_qc (genes > 500, UMIs > 1000, mitochondrial percentage below threshold), optional SAVER imputation, and Cell_filtering, fit graded LDA topic models over topic_number = c(4:6) with MUSIC's Get_topics(), running each topic number separately and combining models if needed. The article restricts the search to this narrow scope because it is 'a reasonable scope' of topic model number to try. Then call Select_topic_number(), which scores each candidate model by topic specificity, by purity, and by their combination, and selects the optimal topic number automatically in a data-driven manner rather than by user choice. Use the selected model downstream for Diff_topic_distri() and perturbation ranking; topic-specific interpretation can be aided by Topic_func_anno() (clusterProfiler enrichment).

## Related tools

- **MUSIC** (Provides Get_topics() to fit graded LDA topic models over topic_number=c(4:6) and Select_topic_number() to automatically pick the optimal topic number using topic specificity and purity scores) — https://github.com/bm2-lab/MUSIC
- **topicmodels** (R package underlying the graded LDA topic model fitting in MUSIC's Get_topics())
- **SAVER** (Optional Data_imputation() step to recover sparse expression values before topic model building) — https://github.com/mohuangx/SAVER
- **clusterProfiler** (Used in Topic_func_anno() to annotate the function of each topic after topic number selection)
- **R** (Runtime environment; MUSIC installs via devtools::install_github under R>=3.4.1)

## Examples

```
crop_seq_topics <- Get_topics(crop_seq_filtered$expression_profile, crop_seq_filtered$perturb_information, topic_number = c(4:6)); optimal <- Select_topic_number(crop_seq_topics)
```

## Evaluation signals

- The selection procedure returns a definite optimal topic number from the candidate set {4, 5, 6} with accompanying specificity, purity, and combined scores
- The chosen model, when carried into Diff_topic_distri() and Rank_overall()/Rank_specific(), reproduces paper-reported rankings (e.g., PERK TPDS 94.0 > IRE1alpha 23.2 > ATF6 11.0 on the UPR dataset)
- Re-running Get_topics() with the same seed yields reproducible models for each candidate topic number
- Topic functions from Topic_func_anno() (clusterProfiler enrichment) are biologically interpretable for the selected topic number
- The optimal topic number is chosen by the specificity/purity combination metric, not fixed by the user, confirming the data-driven strategy was applied

## Limitations

- The recommended topic number scope is narrow (only 4 to 6), which may miss the true optimum for datasets needing fewer or more topics
- The distribution of topics between cases and controls is affected by their sample-size ratio; this imbalance is addressed later by bootstrapping during effect prioritization, not during topic selection
- Topic modeling is performed on the entire screen dataset so different perturbations are compared under the same background; per-perturbation model fitting is not supported

## Evidence

- [discussion] Automatic topic number selection: [evidence span withheld — anchor status: not_found]
- [discussion] Narrow recommended scope: "we recommended a reasonable scope (now 4 to 6) of topic model number to be tried"
- [readme] Topic model fitting in README: "The optimal topic number is automatically selected by **MUSIC** in a data-driven manner."
- [discussion] Topic modeling on entire dataset: "topic modeling was performed on the entire screen dataset to compare the impact of different perturbations under the same background"
- [methods] Install instruction: "You can install the MUSIC package from Github using devtools packages with R>=3.4.1"
