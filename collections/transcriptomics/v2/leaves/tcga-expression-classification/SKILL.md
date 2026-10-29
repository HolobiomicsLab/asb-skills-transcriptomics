---
name: tcga-expression-classification
description: Use when classifying TCGA breast cancer RNA-seq samples with PAM50 and
  you must quantify how the reference subset (its size, ER+/ER− balance, or medoid
  recomputation) changes subtype-call concordance with published labels.
when_to_use_negative:
- The goal is single-sample classification with no cohort available to build a reference
  — use the article's single-sample alternatives (AIMS, mLR) instead
- You only need one classification run and are not varying or diagnosing reference-subset
  composition
- The input is microarray data with no log2-transformation/calibration strategy specified
  — AWCA-based PAM50 was validated on Affymetrix log2-transformed data (e.g. GSE4922),
  not raw RNA-seq-style inputs
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PAM50
  role: Nearest-centroid classifier (Spearman correlation to subtype medians) whose
    50-gene panel and medoid-based reference recomputation are the object of the sensitivity
    analysis
- name: AWCA_PAM50_BC_subtyping
  role: R implementation of the article's AWCA version of PAM50 that mitigates the
    reference-cohort bias this skill measures, with predefined external references
    for classifying independent RNA-seq profiles
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: mLR intrinsic subtyping
  role: Regularized multiclass logistic-regression single-sample caller used in the
    article as a reference-free comparison (90% concordance with PAM50 calls)
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:f45c894d8fafbbef0fde45afefcbe066c3a8e67c4468b09133d7d06756472bcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - AWCA_PAM50_BC_subtyping
  - PAM50
  - mLR intrinsic subtyping
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tcga-expression-classification

## Summary

Sensitivity analysis of PAM50 subtype calling on TCGA breast cancer RNA-seq: rebuild nearest-centroid references from ER+/ER−-controlled sample subsets and measure concordance against published TCGA PAM50 labels. Grounds the finding that reference-cohort composition (e.g. 262-sample 50/50 ER+/ER− subsets) is a dominant driver of classification reproducibility.

## When to use

Use when classifying TCGA breast cancer RNA-seq samples with PAM50 and you must quantify how the reference subset (its size, ER+/ER− balance, or medoid recomputation) changes subtype-call concordance with published labels.

## When NOT to use

- The goal is single-sample classification with no cohort available to build a reference — use the article's single-sample alternatives (AIMS, mLR) instead
- You only need one classification run and are not varying or diagnosing reference-subset composition
- The input is microarray data with no log2-transformation/calibration strategy specified — AWCA-based PAM50 was validated on Affymetrix log2-transformed data (e.g. GSE4922), not raw RNA-seq-style inputs

## Inputs

- TCGA breast cancer RNA-seq gene-expression matrix (samples × genes, including the 50 PAM50 panel genes)
- Published PAM50 intrinsic subtype labels per TCGA sample (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- ER status annotations per sample (ER+/ER−)
- Ciriello et al. technical control subset (262 samples, 50% ER+/50% ER−)

## Outputs

- Per-reference subtype calls for all TCGA samples under each of the 21 references
- Per-reference concordance values against published labels, averaged within reference groups, written to concordance_results.csv

## How to apply

Load the TCGA breast cancer RNA-seq expression matrix with published PAM50 subtype labels and ER status annotations via pandas. Draw randomized subsets of the 50-gene PAM50 panel cohort: ten 400-sample subsets at 60% ER+/40% ER− and ten 262-sample subsets at 50% ER+/50% ER−, plus the Ciriello et al. technical control subset, matching the control's dimensionality (262) and ER proportion. For each subset, build a reference by medoid-based centroid recomputation following the standard PAM50 reference-building procedure, then classify all TCGA samples with the PAM50 nearest-centroid rule using Spearman correlation to subtype medians. Average per-reference concordance (scikit-learn/pandas) within the three reference groups: the rationale, per the article, is that 'standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction', and composition-matched references recover published calls far better (95.45% vs 85.52% mean concordance).

## Related tools

- **PAM50** (Nearest-centroid classifier (Spearman correlation to subtype medians) whose 50-gene panel and medoid-based reference recomputation are the object of the sensitivity analysis)
- **AWCA_PAM50_BC_subtyping** (R implementation of the article's AWCA version of PAM50 that mitigates the reference-cohort bias this skill measures, with predefined external references for classifying independent RNA-seq profiles) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **mLR intrinsic subtyping** (Regularized multiclass logistic-regression single-sample caller used in the article as a reference-free comparison (90% concordance with PAM50 calls)) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
import pandas as pd; conc = pd.DataFrame({'group': g, 'concordance': c}); conc.to_csv('concordance_results.csv')
```

## Evaluation signals

- Mean concordance for the ten 50/50 ER+/ER− 262-sample references must reproduce the reported 95.45% against published TCGA calls
- Mean concordance for the ten 60/40 400-sample references must reproduce the reported 85.52%
- Concordance for the Ciriello et al. technical control reference must reproduce the reported 99.27%
- All random subsets must match the control's dimensionality (262 samples) and 50/50 ER+/ER− proportion before centroid recomputation
- All per-reference and averaged concordance values are present in concordance_results.csv

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction, limiting robustness and reproducibility of subtype calls
- Concordance metrics are biased because published TCGA labels were themselves obtained with the PAM50 assay on the same 50-gene panel
- Ambiguous samples (poorly or comparably correlated to more than one class centroid) destabilize classification, and the Normal-like class degrades performance because its clinical significance remains undetermined

## Evidence

- [abstract] Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction.: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [discussion] AWCA allows predefined external references for independent profiles.: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [discussion] AWCA-based classifications among themselves reach concordance beyond 98%.: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [readme] README describes the R-code for AWCA-PAM50 and mLR subtyping of the five intrinsic subtypes.: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method"
