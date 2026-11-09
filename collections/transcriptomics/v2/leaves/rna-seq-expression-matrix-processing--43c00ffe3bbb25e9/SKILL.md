---
name: rna-seq-expression-matrix-processing
description: Use when a pre-trained RNA-seq subtyping model (mLR with PAM50, limma50,
  or limma50_BWE features) must be applied to a wholly unseen external expression
  matrix, or when reproducing external testing accuracies like Table 2.
when_to_use_negative:
- When the input data are microarray profiles (e.g. Affymetrix log2-transformed GSE4922/GSE1456)
  rather than RNA-seq expression matrices
- When subtype labels for the cohort are already available and no external validation
  of classifiers is needed
- When the external matrix normalization does not match the training cohort's (RSEM
  vs FPKM mismatch), as gene feature values would not be comparable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Feature selection method underlying the limma50 and limma50_BWE gene signatures
    used by the mLR classifiers applied to the external matrices
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA_PAM50_BC_subtyping
  role: AWCA version of PAM50 used as a reference-free classification method and as
    a comparator for concordance on external datasets
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:eb923e0877f8a73c2c3bedc0e8a62838b765fb5f621ea4cb7abecbaf8818e117
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AWCA_PAM50_BC_subtyping
  - limma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# rna-seq-expression-matrix-processing

## Summary

Prepare external RNA-seq breast cancer expression matrices (e.g. PanCA, GSE81538) with the same normalization as training data (TCGA RSEM, GSE96058 FPKM) so pre-trained mLR and AWCA-PAM50 classifiers can be applied to unseen profiles for single-sample intrinsic subtyping.

## When to use

Use when a pre-trained RNA-seq subtyping model (mLR with PAM50, limma50, or limma50_BWE features) must be applied to a wholly unseen external expression matrix, or when reproducing external testing accuracies like Table 2.

## When NOT to use

- When the input data are microarray profiles (e.g. Affymetrix log2-transformed GSE4922/GSE1456) rather than RNA-seq expression matrices
- When subtype labels for the cohort are already available and no external validation of classifiers is needed
- When the external matrix normalization does not match the training cohort's (RSEM vs FPKM mismatch), as gene feature values would not be comparable

## Inputs

- External breast cancer RNA-seq expression matrix (e.g. PanCA, GSE81538) processed with matching normalization (RSEM or FPKM)
- Pre-trained mLR model objects for PAM50, limma50, and limma50_BWE feature sets from the BC_Intrinsic_subtyping repository
- Published PAM50 reference labels for the external cohort
- AWCA-PAM50 subtype calls for the external cohort (for concordance computation)

## Outputs

- Per-sample intrinsic subtype calls (five classes) from each mLR feature set
- Per-classifier accuracy against published PAM50 labels
- Per-classifier concordance with AWCA-PAM50 calls
- Compiled results table (accuracy and concordance per classifier per external dataset)

## How to apply

Load the pre-trained mLR model objects (PAM50, limma50, limma50_BWE feature sets) from the BC_Intrinsic_subtyping repository together with the external test expression matrix (e.g. PanCA for TCGA-trained models, GSE81538 for GSE96058-trained models). Ensure the external matrix is processed with the same normalization as the training cohort (TCGA RSEM or GSE96058 FPKM) so gene feature values are comparable. Apply each trained mLR to the matrix to produce single-sample intrinsic subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) for every sample. Score predictions against published PAM50 reference labels and compute concordance with AWCA-PAM50 calls. Compile per-classifier accuracy and AWCA-PAM50 concordance into a results table mirroring Table 2. Optionally apply AWCA_PAM50_BC_subtyping from the same repo for reference-free centroid classification on the same normalized profiles.

## Related tools

- **limma** (Feature selection method underlying the limma50 and limma50_BWE gene signatures used by the mLR classifiers applied to the external matrices) — github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA_PAM50_BC_subtyping** (AWCA version of PAM50 used as a reference-free classification method and as a comparator for concordance on external datasets) — github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
Rscript apply_mLR.R --models PAM50,limma50,limma50_BWE --expression PanCA_rsem.tsv --labels pam50_reference.tsv
```

## Evaluation signals

- Reproduced accuracy of the PAM50-mLR on PanCA should be ~90%, matching Table 2
- Reproduced accuracy of limma50-mLR on PanCA should be ~88% and limma50_BWE ~87%
- AWCA-PAM50 concordance for PAM50-mLR on PanCA should be ~90%, and ~91% for limma50_BWE
- Every external sample must receive exactly one of the five subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- Normalized external matrix gene values must be on the same scale as the training cohort's (RSEM for TCGA, FPKM for GSE96058) before prediction

## Limitations

- Published subtype labels were themselves obtained with the PAM50 assay, biasing accuracy estimates toward PAM50-based classifiers
- The Normal-like class has undetermined clinical significance and causes performance degradation
- AWCA-derived signatures (limma50, limma50_BWE) brought less convincing performances on testing compared to PAM50-gene classifiers
- Few training samples and a vast number of features limit subtyping capability for hard-to-recognize cases

## Evidence

- [results] external testing PanCA mLR accuracies: [evidence span withheld — anchor status: not_found]
- [abstract] mLR concordance without reference sample: "mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample"
- [readme] README classifier scope: [evidence span withheld — anchor status: not_found]
- [discussion] limma signature performance caveat: [evidence span withheld — anchor status: not_found]
