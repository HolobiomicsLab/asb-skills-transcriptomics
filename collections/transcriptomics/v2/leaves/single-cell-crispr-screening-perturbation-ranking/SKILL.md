---
name: single-cell-crispr-screening-perturbation-ranking
description: Use when you need to validate that a single-cell CRISPR screening pipeline's
  perturbation-effect ranking (e.g., MUSIC's Rank_overall) is stable under control-cell
  label noise, or to reproduce the reported robustness figures before trusting downstream
  ranking-driven conclusions.
when_to_use_negative:
- You want an absolute perturbation-effect estimate or functional topic annotation,
  not a stability/reproducibility check on existing rankings
- Your dataset lacks a blank-control cell population (no 'CTRL' label), since the
  test relabels control cells as a held-out group
- Perturbations fall below MUSIC's minimal cell threshold (fewer than 30 perturbed
  cells), which MUSIC excludes from analysis anyway
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MUSIC
  role: Integrated pipeline whose Rank_overall and Diff_topic_distri functions produce
    the perturbation rankings and topic-distribution differences being stress-tested
  repo: https://github.com/bm2-lab/MUSIC
- name: R
  role: Runtime environment (>=3.4.1) in which the relabeling, ranking, and correlation
    computations are scripted
- name: devtools
  role: Used to install the MUSIC package from GitHub via install_github("bm2-lab/MUSIC")
- name: SAVER
  role: Imputation backend (Data_imputation step) whose optional use is one of the
    preprocessing variants whose effect on ranking stability can be compared
  repo: https://github.com/mohuangx/SAVER
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:4a898721e81d7acdb67ec9fa115297087504df32d222be82efa7fad5b09e09aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - MUSIC
  - R
  - SAVER
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-crispr-screening-perturbation-ranking

## Summary

A robustness test for MUSIC's overall perturbation effect rankings: randomly relabel a fraction (e.g., 20%) of control cells as a held-out test perturbation, re-rank perturbations with Rank_overall, and quantify ranking stability via disturbance rate and Pearson correlation across repeated random relabelings.

## When to use

Use when you need to validate that a single-cell CRISPR screening pipeline's perturbation-effect ranking (e.g., MUSIC's Rank_overall) is stable under control-cell label noise, or to reproduce the reported robustness figures before trusting downstream ranking-driven conclusions.

## When NOT to use

- You want an absolute perturbation-effect estimate or functional topic annotation, not a stability/reproducibility check on existing rankings
- Your dataset lacks a blank-control cell population (no 'CTRL' label), since the test relabels control cells as a held-out group
- Perturbations fall below MUSIC's minimal cell threshold (fewer than 30 perturbed cells), which MUSIC excludes from analysis anyway

## Inputs

- crop_stimulated expression_profile.txt (36722 genes x 3259 cells matrix) and perturb_information.txt with control cells labeled 'CTRL'
- Number of cells to relabel (e.g., 20% of control cells) and random seeds for repeated relabelings (e.g., 10 repetitions)

## Outputs

- Overall perturbation effect ranking (Rank_overall) including the relabeled control-test group, per repetition
- Average disturbance rate of the control-test ranking across repetitions (expected ~0.06)
- Average Pearson correlation between test-subset rankings and the original ranking (expected ~0.82)

## How to apply

Load the crop_stimulated expression profile (36722 genes x 3259 cells) and perturbation information into MUSIC (install via devtools install_github("bm2-lab/MUSIC") on R>=3.4.1), ensuring the control sample is labeled 'CTRL'. Randomly relabel 20% of the control cells as a held-out control-test subset (e.g., via perturbation-label manipulation) and compute topic-distribution differences with Diff_topic_distri, then rank all perturbations including the relabeled group with Rank_overall, recording the test subset's rank and its correlation with the original full-data ranking. Repeat the relabeling and re-ranking 10 times with different random seeds, then summarize the average disturbance rate and average Pearson correlation against the original ranking. Optionally benchmark preprocessing sensitivity by comparing rankings produced under imputation only (SAVER), filtering only, or their combination, since each strategy altered rankings with average similarity ~0.6 across 14 datasets.

## Related tools

- **MUSIC** (Integrated pipeline whose Rank_overall and Diff_topic_distri functions produce the perturbation rankings and topic-distribution differences being stress-tested) — https://github.com/bm2-lab/MUSIC
- **R** (Runtime environment (>=3.4.1) in which the relabeling, ranking, and correlation computations are scripted)
- **devtools** (Used to install the MUSIC package from GitHub via install_github("bm2-lab/MUSIC"))
- **SAVER** (Imputation backend (Data_imputation step) whose optional use is one of the preprocessing variants whose effect on ranking stability can be compared) — https://github.com/mohuangx/SAVER

## Examples

```
library(MUSIC); crop_seq_list<-Input_preprocess(expression_profile,perturb_information); crop_seq_qc<-Cell_qc(crop_seq_list$expression_profile,crop_seq_list$perturb_information,species="Hs",plot=F)
```

## Evaluation signals

- Average disturbance rate of the control-test ranking across the 10 random relabelings is close to the reported ~0.06
- Average Pearson correlation between the 10 test-subset rankings and the original full-data ranking is close to the reported ~0.82
- Expression profile dimensions match the source data (36722 genes x 3259 cells) before preprocessing
- Perturbation label vector contains a 'CTRL' class whose cells are the only source of the relabeled test group
- Preprocessing ablation (imputation only, filtering only, both) yields ranking lists with average similarity around ~0.6 to the unprocessed ranking

## Limitations

- MUSIC's current filtering steps for data noise may need future refinement, so ranking stability is measured against a pipeline whose filters are acknowledged as improvable
- The relabeled control-test group is synthetic noise; results depend on the chosen relabeling fraction (20%) and the number of random repetitions (10)
- Preprocessing choices (SAVER imputation and cell filtering) only moderately preserve rankings (average similarity ~0.6), so absolute rank positions should not be over-interpreted across pipelines

## Evidence

- [discussion] MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation: "MUSIC adopted SAVER16, a R package for single-cell RNA-seq data imputation"
- [discussion] blank control Jurkat test: "we made the following test with stimulated Jurkat cell7 which offered cells without any induction of gRNAs (blank control)"
- [readme] install instruction: "You can install the **MUSIC** package from Github using **devtools** packages with R>=3.4.1."
- [readme] control label requirement: "The label of the control sample needs to be "CTRL"."
- [readme] input preprocessing: "crop_seq_list<-Input_preprocess(expression_profile,perturb_information)"
