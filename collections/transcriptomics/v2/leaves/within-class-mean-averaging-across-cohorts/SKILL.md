---
name: within-class-mean-averaging-across-cohorts
description: Use when building a predefined external gene-by-subtype centroid reference
  for PAM50-style classification of RNA-seq breast cancer samples, and you need the
  reference to be independent of the arbitrary composition of the training cohort.
when_to_use_negative:
- The goal is a single-sample classifier without any external reference (e.g. the
  paper's mLR approach reached 90% concordance with PAM50 'without need of reference
  sample')
- Input expression data are not normalized on the same platform/scale as the training
  cohorts (the authors applied AWCA separately to RNA-seq and to Affymetrix log2-transformed
  data such as GSE4922)
- A quick one-off within-cohort PAM50 run is acceptable and reference portability/reproducibility
  across cohorts is not a requirement
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BC_Intrinsic_subtyping
  role: Provides the PAM50-labeled training cohorts and the R-code implementing the
    AWCA_PAM50_BC_subtyping reference and classifier
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: pandas
  role: groupby aggregation over subtype labels to compute per-gene within-class means
    per cohort before cross-cohort averaging
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:abba1f18f93ac1bb3bf8bc0d857b9e34e9012154328e24ced1e57781c32c9294
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - BC_Intrinsic_subtyping
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# within-class-mean-averaging-across-cohorts

## Summary

Constructs a cohort-composition-independent PAM50 subtype reference by computing per-gene within-class (subtype) mean expression in each training cohort and averaging those class means across cohorts — the 'average of within-class averages' (AWCA) step of the AWCA-PAM50 method.

## When to use

Use when building a predefined external gene-by-subtype centroid reference for PAM50-style classification of RNA-seq breast cancer samples, and you need the reference to be independent of the arbitrary composition of the training cohort.

## When NOT to use

- The goal is a single-sample classifier without any external reference (e.g. the paper's mLR approach reached 90% concordance with PAM50 'without need of reference sample')
- Input expression data are not normalized on the same platform/scale as the training cohorts (the authors applied AWCA separately to RNA-seq and to Affymetrix log2-transformed data such as GSE4922)
- A quick one-off within-cohort PAM50 run is acceptable and reference portability/reproducibility across cohorts is not a requirement

## Inputs

- Subtype-labeled gene expression cohort(s), e.g. the PAM50-labeled RNA-seq training matrix from github.com/DEIB-GECO/BC_Intrinsic_subtyping
- Subtype label assignments (preliminary PAM50 classification) per sample
- Authors' precomputed reference table (Supplementary Material Table S3) for structural validation

## Outputs

- Gene-by-subtype AWCA reference centroid table (e.g. awca_reference_table.csv), formatted to match Supplementary Material Table S3

## How to apply

Start from a preliminary PAM50 classification of the subtype-labeled training cohort(s) (e.g. the PAM50-labeled matrix from the authors' repository github.com/DEIB-GECO/BC_Intrinsic_subtyping). For each cohort, compute per-gene mean expression within each of the five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) using a groupby aggregation over subtype labels in pandas (or the equivalent R implementation). Then average the within-subtype class means across all training cohorts per gene, so the final reference value does not scale with subtype numerosity — this mitigates the cohort-composition bias that makes standard PAM50 references unstable. Assemble the resulting gene-by-subtype centroid table to structurally match the authors' precomputed references in Supplementary Material Table S3 and save it (e.g. awca_reference_table.csv). Validate by checking structural agreement with Table S3 (same gene and subtype columns) and that downstream AWCA-based classifications built from it reach the reported beyond-98% concordance among AWCA-based classifications.

## Related tools

- **BC_Intrinsic_subtyping** (Provides the PAM50-labeled training cohorts and the R-code implementing the AWCA_PAM50_BC_subtyping reference and classifier) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **pandas** (groupby aggregation over subtype labels to compute per-gene within-class means per cohort before cross-cohort averaging)

## Examples

```
ref = df.groupby('pam50_label').mean().T; awca = ref.groupby(level=0, axis=1).mean(); awca.to_csv('awca_reference_table.csv')
```

## Evaluation signals

- The output centroid table has the same gene and subtype columns as the authors' precomputed references (Supplementary Material Table S3)
- Reference values are per-gene averages of within-subtype class means, not overall cohort means — no single cohort's numerosity dominates any subtype
- Classifications made with the constructed reference against AWCA-based classifications from the authors' precomputed references exceed 98% concordance
- The reference yields over 90% concordance and retained prognostic ability relative to standard PAM50 calls
- All five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) are represented as columns in the final table

## Limitations

- Standard PAM50 (and any reference built within-cohort) is profoundly affected by the composition of the sample cohort used for reference construction — this skill is the mitigation, but performance depends on the training cohorts available
- Published subtype labels used for training are themselves PAM50-derived on the same gene panel, biasing downstream accuracy estimates
- The Normal-like class has undetermined clinical significance and degrades classification performance
- Few training samples and a vast number of features limit subtyping capability for hard-to-recognize, ambiguous samples

## Evidence

- [discussion] the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [abstract] standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [discussion] The concordance achieved by AWCA-based classifications among them is beyond 98%: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method ([AWCA_PAM50_BC_subtyping]: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method"
