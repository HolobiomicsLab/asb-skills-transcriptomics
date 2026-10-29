---
name: bootstrap-resampling-sample-imbalance-correction
description: Use when scoring perturbation impact from per-cell topic distributions
  in single-cell CRISPR screens (e.g., crop_stimulated CROP-seq data) where perturbed
  and CTRL cell numbers differ and a sample-imbalance-corrected ranking (TPDS/TPDR/CS)
  is required.
when_to_use_negative:
- You only need to identify clusters or annotate topic functions and do not need a
  case-vs-control perturbation ranking
- The dataset has not yet passed Input_preprocess/Cell_qc/Cell_filtering and no topic
  model (Get_topics) has been fitted
- Perturbations with fewer than 30 perturbed cells, which MUSIC excludes from consideration
  by default
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MUSIC
  role: Supplies Get_topics, Diff_topic_distri, Rank_overall and Rank_specific, whose
    bootstrapping strategy corrects case/control sample imbalance when prioritizing
    perturbation effects
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Optional upstream single-cell RNA-seq imputation whose output feeds the topic
    model used in the bootstrap-based scoring
  repo: https://github.com/mohuangx/SAVER
- name: topicmodels
  role: R package used to fit the graded LDA topic models whose topic probability
    profiles are compared under bootstrapping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:4fb34f68acf6561c48a6464e30b8b21c81f5cfce98ec2a639d1a01422cff9bc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - Get_topics
  - MUSIC
  - R
  - SAVER
  - topicmodels
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bootstrap-resampling-sample-imbalance-correction

## Summary

MUSIC's bootstrapping strategy that corrects the case/control cell-count imbalance when ranking gene perturbation effects from topic-model distributions, yielding the TPDS (overall), TPDR (topic-specific), and CS (efficiency) scores.

## When to use

Use when scoring perturbation impact from per-cell topic distributions in single-cell CRISPR screens (e.g., crop_stimulated CROP-seq data) where perturbed and CTRL cell numbers differ and a sample-imbalance-corrected ranking (TPDS/TPDR/CS) is required.

## When NOT to use

- You only need to identify clusters or annotate topic functions and do not need a case-vs-control perturbation ranking
- The dataset has not yet passed Input_preprocess/Cell_qc/Cell_filtering and no topic model (Get_topics) has been fitted
- Perturbations with fewer than 30 perturbed cells, which MUSIC excludes from consideration by default

## Inputs

- crop_stimulated-style expression profile (gene-by-cell matrix, e.g. expression_profile.txt with 36722 genes x 3259 cells)
- Per-cell perturbation labels (perturb_information), with the control sample labeled 'CTRL'
- distri_diff object from MUSIC's Diff_topic_distri() on the optimal LDA model

## Outputs

- Perturbation-level impact scores: TPD (z-normalized topic probability difference), TPDS (overall), TPDR (topic-specific), CS (efficiency score)
- Verified perturbation ranking table written to music_topic_scores_rankings.csv

## How to apply

After preprocessing (Input_preprocess, Cell_qc, optional SAVER imputation, Cell_filtering) and fitting graded LDA topic models with Get_topics over topic_number = c(4:6), select the optimal topic number with Select_topic_number (scoring topic specificity, purity, and their combination). Compute per-cell topic distributions for each perturbed sample against controls using Diff_topic_distri() on the optimal model, with the control labeled 'CTRL'. Then z-normalize the topic probability difference (TPD) between perturbed and control cells, and run 1000x bootstrap resampling so the topic distribution comparison does not depend on the ratio of case to control sample numbers. Aggregate the resampled TPD values into the overall impact score TPDS and topic-specific statistic TPDR, and compute the efficiency score CS by sweeping random thresholds. Cross-check the implemented scores against Rank_overall() and Rank_specific() on the same distri_diff object and write the verified table to music_topic_scores_rankings.csv.

## Related tools

- **MUSIC** (Supplies Get_topics, Diff_topic_distri, Rank_overall and Rank_specific, whose bootstrapping strategy corrects case/control sample imbalance when prioritizing perturbation effects) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Optional upstream single-cell RNA-seq imputation whose output feeds the topic model used in the bootstrap-based scoring) — https://github.com/mohuangx/SAVER
- **topicmodels** (R package used to fit the graded LDA topic models whose topic probability profiles are compared under bootstrapping)

## Examples

```
distri_diff <- Diff_topic_distri(optimal_model, perturb_information); rank_overall <- Rank_overall(distri_diff); rank_specific <- Rank_specific(distri_diff)
```

## Evaluation signals

- Independently implemented TPD/TPDS/TPDR/CS values and perturbation rankings numerically match Rank_overall() and Rank_specific() output on the same distri_diff object
- Identical ordering of perturbations on the crop_stimulated dataset between the reimplementation and MUSIC's built-in ranking functions
- Verified table music_topic_scores_rankings.csv is produced and agrees with MUSIC's outputs
- Bootstrap iterations (1000x) are applied so scores are stable despite differing numbers of perturbed vs CTRL cells
- TPD values are z-normalized before aggregation into TPDS/TPDR

## Limitations

- The control sample label must be 'CTRL' for the pipeline to run correctly
- MUSIC restricts the tried topic model number to a narrow scope (4 to 6), which bounds the models available for scoring
- Current filtering steps for data noise may need future refinement, which can affect the cell populations fed to the bootstrap scoring
- Off-target information is only provided for knockout, not CRISPRi, perturbations

## Evidence

- [readme] README: sample imbalance is the motivation: "such a sample imbalance issue is addressed by the bootstrapping strategy when prioritizing the perturbation effect"
- [readme] README: imbalance arises from case/control sample ratio: "The distribution of topics between cases and controls is affected by the ratio of their sample numbers"
- [abstract] Abstract: three ranking perspectives produced by the scoring: "prioritizing the gene perturbation effect as an overall perturbation effect, in a functional topic-specific way, and quantifying the relationships between different perturbations"
- [discussion] Discussion: topic model fitted on whole screen for fair comparison: "topic modeling was performed on the entire screen dataset to compare the impact of different perturbations under the same background"
- [readme] README: control label requirement: "The label of the control sample needs to be "CTRL"."
