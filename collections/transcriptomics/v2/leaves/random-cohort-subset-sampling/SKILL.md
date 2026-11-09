---
name: random-cohort-subset-sampling
description: Use when evaluating whether subtype-calling references built from randomly
  sampled cohort subsets (e.g., ten 60/40 ER+/ER− TCGA splits) change PAM50 labels,
  stability, or concordance with published calls.
when_to_use_negative:
- The goal is single-sample classification of a new patient profile — use the pretrained
  AWCA-PAM50 or mLR single-sample callers instead of resampling the cohort.
- Comparing non-PAM50 classifiers such as AIMS or mLR feature-selected signatures,
  where reference-cohort resampling is not the mechanism being evaluated.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: AWCA
  role: 'Reference-construction strategy: averages of within-class averages computed
    on each random 60% training subset to build an external PAM50 reference'
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA-PAM50
  role: Nearest-centroid classifier that assigns full-cohort samples to the five intrinsic
    subtypes using each randomly constructed AWCA reference
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:80a044a731ee3257f8ed971a0dd92ed914172208023bd276e5493d3d575ad4d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - AWCA
  - AWCA-PAM50
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# random-cohort-subset-sampling

## Summary

Draws repeated random 60/40 ER+/ER− stratified training/test splits of a TCGA RNA-seq cohort, builds an AWCA-based PAM50 reference from each 60% training subset, and quantifies how much the resulting classifications vary in concordance and stability versus standard median-based references.

## When to use

Use when evaluating whether subtype-calling references built from randomly sampled cohort subsets (e.g., ten 60/40 ER+/ER− TCGA splits) change PAM50 labels, stability, or concordance with published calls.

## When NOT to use

- The goal is single-sample classification of a new patient profile — use the pretrained AWCA-PAM50 or mLR single-sample callers instead of resampling the cohort.
- Comparing non-PAM50 classifiers such as AIMS or mLR feature-selected signatures, where reference-cohort resampling is not the mechanism being evaluated.

## Inputs

- TCGA breast cancer RNA-seq expression matrix (genes x samples)
- Published standard PAM50 subtype calls for the TCGA cohort (from github.com/DEIB-GECO/BC_Intrinsic_subtyping)
- ER+/ER− status labels for stratifying the 60/40 splits
- Training subset sizes to sweep (400 down to 100 samples)

## Outputs

- awca_concordance.tsv: per-split concordance with published PAM50 calls (mean ± SD, target 91.17% ±0.87%)
- Pairwise concordance matrix among the ten AWCA classifications (mean 99.13% ±0.43%)
- Concordance-vs-training-size trend for subset sizes 400 to 100

## How to apply

Load the TCGA RNA-seq expression matrix and the published PAM50 subtype calls from the DEIB-GECO/BC_Intrinsic_subtyping repository. Draw ten random 60/40 splits stratified by ER status; for each split, run the AWCA pipeline on the 60% training subset to construct an external reference from its standard PAM50 labels, mirroring the predefined references of Supplementary Material Table S3. Classify the full cohort with each of the ten AWCA references via the AWCA-PAM50 nearest-centroid rule and compute concordance with the published calls, expecting ~91.17% ±0.87%. Also compute pairwise concordance among the ten classifications (~99.13% ±0.43%) and repeat with training sizes shrinking from 400 to 100 to verify the reported stability trend, writing all statistics to awca_concordance.tsv. The rationale is that standard PAM50 is profoundly affected by reference-cohort composition, and the AWCA reference strategy mitigates this bias.

## Related tools

- **AWCA** (Reference-construction strategy: averages of within-class averages computed on each random 60% training subset to build an external PAM50 reference) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA-PAM50** (Nearest-centroid classifier that assigns full-cohort samples to the five intrinsic subtypes using each randomly constructed AWCA reference) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
splits = [np.random.choice(n, size=int(0.6*n), replace=False) for _ in range(10)]  # ten random 60/40 training subsets of the TCGA cohort
```

## Evaluation signals

- Mean concordance of the ten AWCA classifications with published TCGA PAM50 calls falls near the reported 91.17% ±0.87%.
- Pairwise concordance among the ten AWCA-based classifications exceeds 98% (reported 99.13% ±0.43%).
- Concordance remains stable as training size shrinks from 400 to 100, matching the reported improvement trend over median-based references.
- awca_concordance.tsv exists and contains mean ± standard deviation statistics for all ten splits plus the size-sweep results.
- Each of the ten training subsets is stratified 60/40 by ER status and drawn without replacement.

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction, so results depend on the TCGA cohort used for resampling.
- Concordance is partly biased because the published subtype labels were themselves obtained from the PAM50 assay on the same gene panel.
- The Normal-like class, whose clinical significance remains undetermined, causes degradation of performance.
- Few training samples and a vast number of features limit subtyping capability, mainly for the hardest-to-recognize (ambiguous) cases.

## Evidence

- [abstract] standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [abstract] AWCA mitigates reference-cohort bias with over 90% concordance: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance, and prognostic ability"
- [discussion] AWCA constructs predefined external references for classifying independent RNA-seq profiles: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [discussion] Inter-classification concordance beyond 98% shows sampling stability: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [readme] Repository R-code implements the AWCA version of PAM50: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method ([AWCA_PAM50_BC_subtyping]"
