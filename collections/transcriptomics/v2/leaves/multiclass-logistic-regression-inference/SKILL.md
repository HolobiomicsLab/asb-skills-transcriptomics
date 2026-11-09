---
name: multiclass-logistic-regression-inference
description: Use when an already-normalized external RNA-seq breast cancer expression
  matrix (e.g. RSEM/FPKM processed like TCGA/GSE96058) must be subtype-called without
  a cohort-derived reference, using the published mLR models.
when_to_use_negative:
- When training a new classifier from scratch on an internal cohort — this skill is
  inference with fixed pre-trained models, not model fitting
- When expression profiles are microarray data or normalized with a scheme different
  from the training cohorts (RSEM for TCGA-trained, FPKM for GSE96058-trained mLRs)
- When the goal is the AWCA-PAM50 centroid method rather than supervised mLR subtype
  calling
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: limma
  role: limma-derived gene signatures (limma50, limma50_BWE) define the feature spaces
    of the pre-trained mLR classifiers applied here
- name: BC_Intrinsic_subtyping
  role: Repository supplying the pre-trained mLR model objects and R code for assigning
    the five BC intrinsic subtypes
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:448bfb92e20c8602a63ff21c6ea10f5d8d38d479f8b06d641d7fa52aba402d1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BC_Intrinsic_subtyping
  - limma
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# multiclass-logistic-regression-inference

## Summary

Apply pre-trained regularized multiclass logistic regression (mLR) models (PAM50, limma50, limma50_BWE feature sets) from the BC_Intrinsic_subtyping repository to RNA-seq breast cancer expression matrices to assign single-sample intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like), then score accuracy against published PAM50 references and concordance with AWCA-PAM50 calls.

## When to use

Use when an already-normalized external RNA-seq breast cancer expression matrix (e.g. RSEM/FPKM processed like TCGA/GSE96058) must be subtype-called without a cohort-derived reference, using the published mLR models.

## When NOT to use

- When training a new classifier from scratch on an internal cohort — this skill is inference with fixed pre-trained models, not model fitting
- When expression profiles are microarray data or normalized with a scheme different from the training cohorts (RSEM for TCGA-trained, FPKM for GSE96058-trained mLRs)
- When the goal is the AWCA-PAM50 centroid method rather than supervised mLR subtype calling

## Inputs

- Pre-trained mLR model objects (PAM50, limma50, limma50_BWE feature sets) from github.com/DEIB-GECO/BC_Intrinsic_subtyping
- External RNA-seq expression matrix processed with the training-cohort normalization (e.g. PanCA RSEM-like, GSE81538 FPKM-like)
- Published PAM50 reference subtype labels for the external cohort (for accuracy scoring)
- AWCA-PAM50 subtype calls on the same cohort (for concordance scoring)

## Outputs

- Per-sample intrinsic subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) from each mLR feature set
- Per-classifier classification accuracy table vs PAM50 reference labels (e.g. 90% PAM50, 88% limma50, 87% limma50_BWE on PanCA)
- Per-classifier AWCA-PAM50 concordance values (e.g. ~90% for PAM50-mLR, ~91% for limma50_BWE on PanCA)

## How to apply

Load the pre-trained mLR model objects (PAM50, limma50, and limma50_BWE feature sets) from the DEIB-GECO/BC_Intrinsic_subtyping repository together with the external test expression matrix (e.g. PanCA, GSE81538), confirming the matrix is normalized with the same scheme used in training (RSEM for TCGA-trained models, FPKM for GSE96058-trained models). Apply each classifier to every sample to emit one of the five intrinsic subtype calls per sample per classifier. Score classification accuracy against published PAM50 reference labels and compute concordance with AWCA-PAM50 calls on the same cohort. Note that PAM50-gene-based mLR performance is partly biased because published labels themselves derive from the PAM50 assay, so AWCA-PAM50 concordance is reported alongside accuracy. Validate against Table 2 reference values, e.g. ~90% accuracy and ~90% AWCA-PAM50 concordance for PAM50-mLR on PanCA, and ~87% accuracy with ~91% concordance for limma50_BWE on PanCA.

## Related tools

- **limma** (limma-derived gene signatures (limma50, limma50_BWE) define the feature spaces of the pre-trained mLR classifiers applied here)
- **BC_Intrinsic_subtyping** (Repository supplying the pre-trained mLR model objects and R code for assigning the five BC intrinsic subtypes) — github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Evaluation signals

- Reproduced PanCA accuracy for TCGA-trained mLRs matches Table 2: 90% (PAM50), 88% (limma50), 87% (limma50_BWE)
- AWCA-PAM50 concordance on PanCA is ~90% for PAM50-mLR and ~91% for limma50_BWE
- External-test accuracies are nearly identical to internal TCGA test-set accuracies, indicating no normalization/cohort drift
- Every sample receives exactly one of the five subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) per classifier
- GSE96058-trained mLRs applied to GSE81538 reproduce the same prediction-and-scoring workflow and reference concordance values

## Limitations

- Accuracy against PAM50-based published labels is biased because those labels themselves derive from the PAM50 assay on the same gene panel
- Small number of training samples and large feature count limit subtyping of hard-to-recognize cases
- The Normal-like class has undetermined clinical significance and degrades performance

## Evidence

- [full_text] On external testing (PanCA), the TCGA-trained mLRs reached accuracies nearly identical to internal TCGA test-set results: [evidence span withheld — anchor status: not_found]
- [abstract] regularized multiclass logistic regression (mLR) displayed the best performance: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome."
- [abstract] mLR classifications reached 90% concordance with PAM50-based calls: [evidence span withheld — anchor status: not_found]
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method ([AWCA_PAM50_BC_subtyping]"
