---
name: external-cohort-validation-of-classifiers
description: Use when pre-trained RNA-seq subtype classifiers must be benchmarked
  on an independent external cohort processed with the same normalization, reproducing
  Table 2 external-testing results.
when_to_use_negative:
- The expression matrix was generated with a different platform or normalization than
  the training cohort (e.g. Affymetrix microarray log2 data) — performance is not
  guaranteed by the article's external-testing setup
- The goal is to train or re-fit classifiers rather than score an unseen cohort with
  existing frozen models
- Reference labels are unavailable and no AWCA-PAM50 calls can be generated, leaving
  no ground truth for accuracy/concordance computation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: limma
  role: limma-derived feature selection underlies the limma50 and limma50_BWE signatures
    used by the mLR classifiers being externally validated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:a5c55022fcc9ade7d80ef469e636929b3e79e41cf4f39be6428512b9a985af20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - limma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# external-cohort-validation-of-classifiers

## Summary

Applies pre-trained mLR intrinsic subtype classifiers (PAM50, limma50, limma50_BWE feature sets) trained on TCGA (RSEM) and GSE96058 (FPKM) to wholly unseen external RNA-seq cohorts (PanCA, GSE81538) to measure generalization accuracy against published PAM50 reference labels and AWCA-PAM50 concordance.

## When to use

Use when pre-trained RNA-seq subtype classifiers must be benchmarked on an independent external cohort processed with the same normalization, reproducing Table 2 external-testing results.

## When NOT to use

- The expression matrix was generated with a different platform or normalization than the training cohort (e.g. Affymetrix microarray log2 data) — performance is not guaranteed by the article's external-testing setup
- The goal is to train or re-fit classifiers rather than score an unseen cohort with existing frozen models
- Reference labels are unavailable and no AWCA-PAM50 calls can be generated, leaving no ground truth for accuracy/concordance computation

## Inputs

- Pre-trained mLR model objects (PAM50, limma50, limma50_BWE feature sets) from the BC_Intrinsic_subtyping repository
- External test RNA-seq expression matrix (e.g. PanCA, GSE81538) processed with the same normalization as the training cohort (RSEM or FPKM)
- Published PAM50 reference subtype labels for the external cohort
- AWCA-PAM50 subtype calls on the external cohort (for concordance scoring)

## Outputs

- Per-sample single-sample intrinsic subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) for each external sample and each classifier
- Results table of per-classifier accuracy vs published PAM50 labels and concordance with AWCA-PAM50 calls

## How to apply

Load the pre-trained mLR model objects for the three feature sets (PAM50, limma50, limma50_BWE) from the DEIB-GECO/BC_Intrinsic_subtyping repository together with the external test expression matrix (e.g. PanCA or GSE81538) normalized identically to the training cohort (RSEM for TCGA-trained models, FPKM for GSE96058-trained models). Apply each classifier sample-by-sample to produce single-sample intrinsic subtype calls across the five classes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like). Compute classification accuracy against published PAM50 reference labels and concordance with AWCA-PAM50 calls. Compile per-classifier accuracy and concordance into a results table and compare against Table 2 reference values. Note the article's caveat that accuracies on PAM50-gene and AWCA-PAM50-based comparisons are biased because published labels themselves derive from the PAM50 assay.

## Related tools

- **limma** (limma-derived feature selection underlies the limma50 and limma50_BWE signatures used by the mLR classifiers being externally validated)

## Evaluation signals

- Reproduced accuracy on PanCA should be ~90% for PAM50-mLR, ~88% for limma50-mLR, and ~87% for limma50_BWE-mLR
- AWCA-PAM50 concordance on PanCA should be ~90% for PAM50-mLR and ~91% for limma50_BWE-mLR
- External-cohort accuracies should be nearly identical to internal TCGA test-set accuracies for the TCGA-trained models
- Every external sample must receive exactly one of the five intrinsic subtype calls per classifier (no missing/ambiguous single-sample outputs)
- GSE96058-trained mLR models applied to GSE81538 should reproduce the Table 2 accuracy and concordance values via the same scoring procedure

## Limitations

- Accuracies on PAM50-gene and AWCA-PAM50-based comparisons are biased because published subtype labels were themselves obtained with the PAM50 assay
- The Normal-like class, whose clinical significance remains undetermined, degrades classification performance
- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction, complicating reference-label comparisons

## Evidence

- [abstract] External-testing accuracy and concordance targets: "mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample"
- [discussion] Bias in PAM50-based comparison labels: "the higher accuracies reached with mLRs using PAM50 genes or with the AWCA-PAM50 method are biased by the nature of the published subtype calls, obtained in their turn from the PAM50 assay"
- [discussion] Normal-like performance degradation: "degradation is caused also by the Normal-like class, whose clinical significance remains undetermined"
- [abstract] Reference-cohort bias motivating robust external evaluation: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [readme] Repository provides pre-trained models and workflow entry points: "our regularized multiclass Logistic Regression models ([mLR intrinsic subtyping](https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/mLR_BC_subtyping)) to assign breast cancer (BC) samples to"
