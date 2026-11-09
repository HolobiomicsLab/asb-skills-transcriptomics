---
name: awca-reference-construction
description: Use when classifying breast cancer RNA-seq samples into the five intrinsic
  subtypes (Luminal A/B, Basal, Her2-enriched, Normal-like) and the standard median-based
  PAM50 reference is sensitive to cohort composition or a single-sample, reference-independent
  caller is needed.
when_to_use_negative:
- Input profiles are microarray data without appropriate transformation — the article
  reports AWCA on Affymetrix log2-transformed data (e.g., GSE4922, GSE1456) and the
  porting workflow assumes comparable RNA-seq profiles
- The goal is a supervised multi-class learner with gene selection rather than nearest-centroid
  subtyping — the article's mLR (regularized multiclass logistic regression with Lasso/Ridge
  and limma50_BWE feature selection) is the alternative single-sample method
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: AWCA
  role: Strategy to build subtype references as averages of within-class averages
    from training PAM50 labels, mitigating reference-cohort bias
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA-PAM50
  role: Nearest-centroid single-sample classifier that assigns samples to the five
    intrinsic subtypes using the constructed AWCA reference
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: limma
  role: Source of limma-derived signatures (e.g., limma50_BWE) compared against AWCA-PAM50
    in the same subtyping framework
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:ef556eb1aaed55abcb0b52a62a0cb2a5f65aaf8eb1ada9d01f9fce762c66c832
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - AWCA
  - AWCA-PAM50
  - limma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# awca-reference-construction

## Summary

Constructs PAM50 subtype references using the AWCA (average of within-class averages) strategy to mitigate reference-cohort composition bias in breast cancer intrinsic subtyping of RNA-seq profiles, yielding predefined external references usable for single-sample classification with over 90% concordance.

## When to use

Use when classifying breast cancer RNA-seq samples into the five intrinsic subtypes (Luminal A/B, Basal, Her2-enriched, Normal-like) and the standard median-based PAM50 reference is sensitive to cohort composition or a single-sample, reference-independent caller is needed.

## When NOT to use

- Input profiles are microarray data without appropriate transformation — the article reports AWCA on Affymetrix log2-transformed data (e.g., GSE4922, GSE1456) and the porting workflow assumes comparable RNA-seq profiles
- The goal is a supervised multi-class learner with gene selection rather than nearest-centroid subtyping — the article's mLR (regularized multiclass logistic regression with Lasso/Ridge and limma50_BWE feature selection) is the alternative single-sample method

## Inputs

- RNA-seq expression matrix (e.g., TCGA RSEM or GSE96058 FPKM profiles), optionally restricted to genes shared across cohorts
- Published/standard PAM50 subtype calls for the training samples
- Training subset specification (e.g., random 60/40 ER+/ER− split)

## Outputs

- AWCA external subtype reference (per-subtype centroids) analogous to Supplementary Material Table S3
- Per-sample intrinsic subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- Concordance statistics table, e.g., awca_concordance.tsv or awca_concordance_results.csv (internal vs. published calls and pairwise stability)

## How to apply

Load a breast cancer RNA-seq expression matrix (e.g., TCGA RSEM or GSE96058 FPKM) together with standard PAM50 labels from github.com/DEIB-GECO/BC_Intrinsic_subtyping. Draw a training subset (e.g., a 60/40 ER+/ER− split, or subsets shrinking from 400 to 100 samples) and run the AWCA pipeline on it, computing for every sample Spearman correlations against the remaining cohort samples to build subtype centroids as averages of within-class averages, mirroring the predefined references of Supplementary Material Table S3. Classify the full cohort or an external cohort with the resulting AWCA reference using the AWCA-PAM50 nearest-centroid (nearest/adjacent-sample) assignment. Rationale: standard PAM50 references built arbitrarily from the dataset under study affect subtype calling; AWCA mitigates this bias and supports single-sample classification. Optionally repeat with multiple random splits to assess stability.

## Related tools

- **AWCA** (Strategy to build subtype references as averages of within-class averages from training PAM50 labels, mitigating reference-cohort bias) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA-PAM50** (Nearest-centroid single-sample classifier that assigns samples to the five intrinsic subtypes using the constructed AWCA reference) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **limma** (Source of limma-derived signatures (e.g., limma50_BWE) compared against AWCA-PAM50 in the same subtyping framework)

## Examples

```
Rscript AWCA_PAM50_BC_subtyping.R --expression tcga_rsem.csv --labels pam50_calls.csv --split 60/40 --output awca_concordance.tsv
```

## Evaluation signals

- Mean concordance of AWCA-based TCGA subtyping instances with published PAM50 calls should reach 91.17% ± 0.87%
- Pairwise concordance among ten AWCA classifications should be 99.13% ± 0.43% (article discussion notes >98% among AWCA-based classifications)
- Concordance trend should improve as training subset size shrinks from 400 to 100 samples, matching the reported improvement trend
- External application: concordance of the best TCGA AWCA reference on PanCA and the best GSE96058 FPKM reference on GSE81538 should exceed 96%
- Internal AWCA concordance on GSE96058 should be at least 95%, and approximately 95% for PanCA and GSE81538

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction; reference construction choices affect subsequent subtype calling and limit robustness and reproducibility
- Ambiguous samples, poorly or comparably correlated to more than one class centroid, reduce classification stability
- Performance is biased because published subtype labels were themselves obtained with the PAM50 assay on the same gene panel
- The Normal-like class has undetermined clinical significance and causes degradation of performance
- Small numbers of training samples and large feature spaces limit subtyping capability for hard-to-recognize cases

## Evidence

- [abstract] we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance"
- [abstract] standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [discussion] the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [discussion] The concordance achieved by AWCA-based classifications among them is beyond 98%: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method"
