---
name: formula-verification-against-reference-implementation
description: 'Use when reproducing or validating a published scoring/ranking method:
  you have the paper''s formula definitions, the reference package (e.g. MUSIC''s
  Rank_overall/Rank_specific), and a shared test dataset (e.g. crop_stimulated).'
when_to_use_negative:
- When the goal is a first-pass analysis of new screening data rather than validating
  a re-implementation of the scoring formulas
- When the reference implementation's functions cannot be run on the same intermediate
  object (e.g. no access to the distri_diff object), since numerical matching is the
  acceptance criterion
- When the perturbation-level scoring method differs (e.g. non-topic-model CRISPR
  screen analysis); this skill is specific to reproducing MUSIC's topic-distribution-based
  scores
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MUSIC
  role: Reference implementation providing Rank_overall() and Rank_specific() whose
    outputs serve as ground truth for verification
  repo: https://github.com/bm2-lab/MUSIC
- name: R
  role: Language in which the independent re-implementation of TPD/TPDS/TPDR/CS formulas
    and bootstrap resampling is written (R>=3.4.1 required for MUSIC)
- name: topicmodels
  role: R package used to fit the graded LDA topic models underlying the TPD computation
- name: Diff_topic_distri
  role: MUSIC function producing the per-cell topic distribution differences (distri_diff)
    that both the custom and reference scoring must consume identically
  repo: https://github.com/bm2-lab/MUSIC
- name: Select_topic_number
  role: Selects the optimal topic number (within 4–6) by scoring topic specificity,
    purity, and their combination, fixing the model on which scores are verified
  repo: https://github.com/bm2-lab/MUSIC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:166519413d7b2c19b5a07671bcbad73d4447718f80bb6d5ea59ce0c3a6eedbe0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - Diff_topic_distri
  - Get_topics
  - MUSIC
  - R
  - Select_topic_number
  - topicmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Formula verification against reference implementation

## Summary

Independently re-implements a paper's quantitative scoring formulas (here MUSIC's TPD, TPDS, TPDR, and CS for topic-model-based perturbation ranking) and verifies correctness by numerically matching the re-implementation's values and perturbation orderings against the reference implementation's functions on the same input object.

## When to use

Use when reproducing or validating a published scoring/ranking method: you have the paper's formula definitions, the reference package (e.g. MUSIC's Rank_overall/Rank_specific), and a shared test dataset (e.g. crop_stimulated).

## When NOT to use

- When the goal is a first-pass analysis of new screening data rather than validating a re-implementation of the scoring formulas
- When the reference implementation's functions cannot be run on the same intermediate object (e.g. no access to the distri_diff object), since numerical matching is the acceptance criterion
- When the perturbation-level scoring method differs (e.g. non-topic-model CRISPR screen analysis); this skill is specific to reproducing MUSIC's topic-distribution-based scores

## Inputs

- distri_diff object of per-cell topic distributions per perturbation vs. control, from Diff_topic_distri() on the optimal graded LDA model
- crop_stimulated expression profile and perturbation_information (control label must be 'CTRL')
- Formula definitions for TPD, TPDS, TPDR, CS from the MUSIC paper (DOI 10.1038/s41467-019-10216-x)

## Outputs

- music_topic_scores_rankings.csv with independently implemented TPD/TPDS/TPDR/CS scores and perturbation rankings
- Ranking comparison report showing identical perturbation ordering between custom implementation and Rank_overall()/Rank_specific() outputs

## How to apply

First compute per-cell topic distribution differences (TPD) between each perturbed sample and control cells on the optimal LDA model selected via Select_topic_number() over topic_number = c(4:6). Then implement the scoring stack: z-normalize TPD, correct case/control sample imbalance with 1000x bootstrap resampling, aggregate into the overall (TPDS) and topic-specific (TPDR) statistics, and compute the efficiency score CS by sweeping random thresholds. Run the reference functions Rank_overall() and Rank_specific() on the identical distri_diff object, then diff both the score values and the perturbation rankings. Acceptance is met only when the independently computed TPD/TPDS/TPDR/CS values and the ordering of perturbations numerically match the reference outputs, after which the verified table is written to music_topic_scores_rankings.csv.

## Related tools

- **MUSIC** (Reference implementation providing Rank_overall() and Rank_specific() whose outputs serve as ground truth for verification) — https://github.com/bm2-lab/MUSIC
- **R** (Language in which the independent re-implementation of TPD/TPDS/TPDR/CS formulas and bootstrap resampling is written (R>=3.4.1 required for MUSIC))
- **topicmodels** (R package used to fit the graded LDA topic models underlying the TPD computation)
- **Diff_topic_distri** (MUSIC function producing the per-cell topic distribution differences (distri_diff) that both the custom and reference scoring must consume identically) — https://github.com/bm2-lab/MUSIC
- **Select_topic_number** (Selects the optimal topic number (within 4–6) by scoring topic specificity, purity, and their combination, fixing the model on which scores are verified) — https://github.com/bm2-lab/MUSIC

## Examples

```
crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Independently computed TPD/TPDS/TPDR/CS values numerically match Rank_overall() and Rank_specific() outputs on the same distri_diff object
- Identical ordering of perturbations between custom ranking and the reference implementation
- Bootstrap resampling uses 1000 replicates as specified for the case/control sample-imbalance correction
- Score verification performed with the optimal topic number chosen by Select_topic_number() within the candidate range 4:6, not an arbitrary model
- Verified table is written and reproducible (set.seed-controlled, e.g. the README's set.seed(234) pattern for stochastic steps)

## Limitations

- MUSIC recommends only a narrow scope of topic numbers (4 to 6), constraining the models over which verification is meaningful
- Reference verification depends on exact intermediate objects (distri_diff); discrepancies in upstream filtering (e.g. cell_num_threshold in Cell_filtering, control label 'CTRL') break numeric matching
- Current filtering steps for data noise may need future refinement, which can shift the cells entering the scored distributions

## Evidence

- [other] MUSIC evaluates the overall effect of a perturbation using an impact score based on the topic distribution (topic probability profile) computed for that perturbation, denoted TPDS.: [evidence span withheld — anchor status: not_found]
- [readme] The distribution of topics between cases and controls is affected by the ratio of their sample numbers, and such a sample imbalance issue is addressed by the bootstrapping strategy: "The distribution of topics between cases and controls is affected by the ratio of their sample numbers, and such a sample imbalance issue is addressed by the bootstrapping strategy"
- [discussion] we recommended a reasonable scope (now 4 to 6) of topic model number to be tried: "we recommended a reasonable scope (now 4 to 6) of topic model number to be tried"
- [readme] You can apply function "Input_preprocess()" to handle this data format. You can format your own data according to the provided example files.: "You can apply function "Input_preprocess()" to handle this data format. You can format your own data according to the provided example files."
- [discussion] MUSIC applies an automatic strategy to select the optimal topic number: "MUSIC applies an automatic strategy to select the optimal topic number"
