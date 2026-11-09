---
name: classification-concordance-analysis
description: Use when replicating or validating a subtype/classification pipeline
  where novel calls must be compared against published labels and pairwise stability
  across repeated instances must be quantified as concordance percentages.
when_to_use_negative:
- Evaluating methods like mLR single-sample callers against the same PAM50-derived
  labels — the article notes such concordance is biased because published subtype
  calls were themselves obtained with the PAM50 assay
- Comparing subtype calls against clinical outcomes or prognostic value rather than
  against another set of subtype labels
- Input lacks a reference set of published labels to serve as ground truth for concordance
  computation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: AWCA
  role: Constructs external subtype references from training-subset PAM50 labels to
    mitigate reference-cohort bias before concordance is measured
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA-PAM50
  role: Nearest-centroid assignment method producing the AWCA-based subtype calls
    whose concordance and stability are evaluated
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:87600c0408533c542a43fe6919e71a810901fb3c96c8168c991697fcaf748371
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AWCA
  - AWCA-PAM50
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# classification-concordance-analysis

## Summary

Quantifies how faithfully independently constructed PAM50 subtype calls (e.g., AWCA-based references built from random 60/40 ER+/ER− subsets) agree with published reference labels and with each other on TCGA RNA-seq data, using mean ± SD concordance as the reproducibility metric.

## When to use

Use when replicating or validating a subtype/classification pipeline where novel calls must be compared against published labels and pairwise stability across repeated instances must be quantified as concordance percentages.

## When NOT to use

- Evaluating methods like mLR single-sample callers against the same PAM50-derived labels — the article notes such concordance is biased because published subtype calls were themselves obtained with the PAM50 assay
- Comparing subtype calls against clinical outcomes or prognostic value rather than against another set of subtype labels
- Input lacks a reference set of published labels to serve as ground truth for concordance computation

## Inputs

- TCGA RNA-seq breast cancer expression matrix (gene x sample)
- Published standard PAM50 subtype calls per sample
- Random 60/40 ER+/ER− train/test split definitions (10 instances)
- AWCA references constructed from training-subset PAM50 labels

## Outputs

- Per-instance concordance with published PAM50 calls (mean ± SD, e.g., 91.17% ± 0.87%)
- Pairwise concordance among AWCA classifications (mean ± SD, e.g., 99.13% ± 0.43%)
- Concordance trend across shrinking training sizes (400→100)
- awca_concordance.tsv summarizing all concordance statistics

## How to apply

Load the TCGA RNA-seq expression matrix and published PAM50 calls from github.com/DEIB-GECO/BC_Intrinsic_subtyping. For each of ten random 60/40 ER+/ER− splits, build an AWCA reference from the 60% training subset's standard PAM50 labels, then classify the full cohort with AWCA-PAM50 nearest-centroid assignment. Compute per-instance concordance with the published calls (expect ~91.17% ± 0.87%) and aggregate across instances as mean ± standard deviation. Additionally compute pairwise concordance among the ten AWCA classifications (expect ~99.13% ± 0.43%) to measure stability against reference-construction stochasticity. Optionally repeat with training subset sizes shrinking from 400 to 100 to verify the reported improvement trend, and write all statistics to a TSV.

## Related tools

- **AWCA** (Constructs external subtype references from training-subset PAM50 labels to mitigate reference-cohort bias before concordance is measured) — github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA-PAM50** (Nearest-centroid assignment method producing the AWCA-based subtype calls whose concordance and stability are evaluated) — github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
Rscript awca_pam50_concordance.R --expression tcga_rnaseq.tsv --labels published_pam50.tsv --splits 10 --train-frac 0.6 --er-stratified --out awca_concordance.tsv
```

## Evaluation signals

- Mean concordance with published TCGA PAM50 calls falls near the reported 91.17% with SD ≈ 0.87% across ten AWCA instances
- Pairwise concordance among the ten AWCA classifications exceeds 98%, matching the reported 99.13% ± 0.43%
- Concordance statistics for all ten instances and all pairwise comparisons are present and non-null in awca_concordance.tsv
- Subsample-size sweep from 400 down to 100 training samples reproduces the reported improvement trend in concordance
- Each AWCA reference is derived independently from a distinct 60/40 ER+/ER− split, not reused across instances

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction, so concordance against a single published reference conflates method agreement with cohort-specific labeling
- Ambiguous samples — poorly or comparably correlated to more than one class centroid — reduce classification stability and depress concordance
- The Normal-like class has undetermined clinical significance and causes degradation of classification performance
- Performance estimates are biased because published subtype labels were themselves obtained with the PAM50 assay on the same gene panel

## Evidence

- [abstract] AWCA concordance with published calls: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance, and prognostic ability"
- [discussion] Stability across AWCA instances: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [discussion] Predefined external references: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [abstract] Reference-cohort bias: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [readme] AWCA-PAM50 tool availability: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method ([AWCA_PAM50_BC_subtyping](https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/AWCA_PAM50_BC_subtyping))"
