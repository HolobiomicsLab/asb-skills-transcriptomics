---
name: reference-vector-validation
description: Use when a PAM50-style centroid reference must be rebuilt from subtype-labeled
  RNA-seq cohorts so that classification no longer depends on cohort composition,
  or when an existing reference table (e.g., Supplementary Table S3 format) must be
  structurally verified before downstream classification.
when_to_use_negative:
- The goal is single-sample classification without reference construction — in that
  case the regularized multiclass Logistic Regression (mLR) approach reaches 90% concordance
  with PAM50-based calls without needing a reference sample.
- Input is already a validated precomputed external reference (e.g., Supplementary
  Material Table S3); rebuilding it from scratch is unnecessary.
tools:
- name: AWCA_PAM50_BC_subtyping
  role: R implementation in the authors' repository that consumes the AWCA reference
    to assign samples to the five intrinsic subtypes; the reference table must match
    its expected format
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/AWCA_PAM50_BC_subtyping
- name: BC_Intrinsic_subtyping
  role: Repository hosting the R-code for AWCA-PAM50 classification and the training
    data from which the reference is derived
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:73e5267d3ad10c8be37e11c8cecf5c5d445a45e6711243f898de692541c5f95b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AWCA_PAM50_BC_subtyping
  - BC_Intrinsic_subtyping
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# reference-vector-validation

## Summary

Builds and validates a subtype-balanced AWCA reference table for PAM50-based breast cancer intrinsic subtyping by averaging within-class expression means per gene across training cohorts, removing the dependence of subtype calls on the arbitrary composition of the cohort used to build the reference.

## When to use

Use when a PAM50-style centroid reference must be rebuilt from subtype-labeled RNA-seq cohorts so that classification no longer depends on cohort composition, or when an existing reference table (e.g., Supplementary Table S3 format) must be structurally verified before downstream classification.

## When NOT to use

- The goal is single-sample classification without reference construction — in that case the regularized multiclass Logistic Regression (mLR) approach reaches 90% concordance with PAM50-based calls without needing a reference sample.
- Input is already a validated precomputed external reference (e.g., Supplementary Material Table S3); rebuilding it from scratch is unnecessary.

## Inputs

- PAM50-labeled gene expression training cohort(s) (gene x sample matrix with intrinsic subtype labels per sample), e.g., from the authors' repository github.com/DEIB-GECO/BC_Intrinsic_subtyping
- Authors' precomputed reference table (Supplementary Material Table S3) for structural comparison

## Outputs

- awca_reference_table.csv: gene-by-subtype AWCA centroid table matching the gene and subtype columns of Supplementary Material Table S3

## How to apply

Load the subtype-labeled gene expression matrix(es) (e.g., the PAM50-labeled training data from github.com/DEIB-GECO/BC_Intrinsic_subtyping) with pandas, then compute the per-gene mean expression within each of the five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) via a groupby aggregation over subtype labels for each training cohort. The AWCA ('average of within-class averages') step then averages these within-subtype class means per gene across all training cohorts, producing a reference value independent of subtype numerosity. Assemble the resulting gene-by-subtype centroid table to match the precomputed references of Supplementary Material Table S3 (same gene and subtype columns) and save it as awca_reference_table.csv. Validate by (1) structural comparison against the authors' precomputed reference table and (2) confirming that classifications built from it reproduce the reported beyond-98% concordance among AWCA-based classifications. The rationale is that the standard PAM50 reference is profoundly affected by cohort composition, limiting robustness and reproducibility of subtype calls.

## Related tools

- **AWCA_PAM50_BC_subtyping** (R implementation in the authors' repository that consumes the AWCA reference to assign samples to the five intrinsic subtypes; the reference table must match its expected format) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/AWCA_PAM50_BC_subtyping
- **BC_Intrinsic_subtyping** (Repository hosting the R-code for AWCA-PAM50 classification and the training data from which the reference is derived) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
import pandas as pd; awca_ref = df.groupby(['cohort','pam50_label']).mean().groupby(level='pam50_label').mean(); awca_ref.to_csv('awca_reference_table.csv')
```

## Evaluation signals

- Structural check: awca_reference_table.csv has the same gene and subtype columns as the authors' precomputed references in Supplementary Material Table S3.
- Downstream AWCA-based classifications built from the table achieve the reported beyond-98% concordance among AWCA-based classifications.
- Each gene's reference value equals the across-cohort average of within-subtype class means, confirming independence from subtype numerosity.
- Applying the reference via the repository's R-code assigns samples across all five subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like).
- Per-cohort groupby means reproduce per-gene, per-subtype averages consistent with the pooled AWCA table.

## Limitations

- The AWCA strategy mitigates but does not eliminate cohort-composition bias; standard PAM50 remains profoundly affected by the composition of the reference-building cohort.
- Performance is biased when published subtype labels were themselves obtained with the PAM50 assay on the same gene panel.
- AWCA-based classifications on non-RNA-seq platforms require adaptation: on Affymetrix data, log2-transformed profiles (e.g., GSE4922, GSE1456) were used.
- The Normal-like class has undetermined clinical significance and degrades classification performance.

## Evidence

- [abstract] standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [discussion] The concordance achieved by AWCA-based classifications among them is beyond 98%: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [discussion] the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3): "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method ([AWCA_PAM50_BC_subtyping]"
- [readme] to assign breast cancer (BC) samples to the five prognosis-related breast cancer _intrinsic_ subtypes: "to assign breast cancer (BC) samples to the five prognosis-related breast cancer _intrinsic_ subtypes (_Luminal A, Luminal B, Basal, Her2-enriched, or Normal-like_)"
