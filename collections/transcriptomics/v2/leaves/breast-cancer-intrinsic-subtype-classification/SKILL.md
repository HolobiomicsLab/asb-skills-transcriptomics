---
name: breast-cancer-intrinsic-subtype-classification
description: Use when you have processed breast cancer RNA-seq expression profiles
  (e.g., TCGA RSEM or GSE96058 FPKM-normalized) and need single-sample intrinsic subtype
  calls without building a cohort-specific reference.
when_to_use_negative:
- Expression data were normalized differently from the training cohorts (the mLRs
  were trained on TCGA RSEM and GSE96058 FPKM data and external testing requires the
  same normalization)
- You need de novo subtype discovery or a cohort-adaptive reference — the AWCA/mLR
  approach explicitly addresses the fact that standard PAM50 is affected by cohort
  composition, and relies on predefined external references instead
- Profiles are Affymetrix microarray data rather than RNA-seq without confirming applicability
  — the article discusses AWCA on Affymetrix log2-transformed data (GSE4922, GSE1456)
  separately from the RNA-seq mLR workflow
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: limma
  role: limma-derived feature selection supplies the limma50 and limma50_BWE gene
    signatures (the latter via backward elimination) used as alternative feature spaces
    for the regularized mLR classifiers
- name: BC_Intrinsic_subtyping
  role: Repository providing the R-code and pre-trained model objects for AWCA_PAM50
    and the mLR intrinsic subtyping classifiers
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:d067215ba2522655a65593faf75538798d76f77fc04e058e997c6e5c903bb402
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# breast-cancer-intrinsic-subtype-classification

## Summary

Assigns breast cancer RNA-seq profiles to the five prognosis-related intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) using regularized multiclass logistic regression (mLR) models with PAM50, limma50, or limma50_BWE gene signatures, or the reference-free AWCA_PAM50 strategy.

## When to use

Use when you have processed breast cancer RNA-seq expression profiles (e.g., TCGA RSEM or GSE96058 FPKM-normalized) and need single-sample intrinsic subtype calls without building a cohort-specific reference.

## When NOT to use

- Expression data were normalized differently from the training cohorts (the mLRs were trained on TCGA RSEM and GSE96058 FPKM data and external testing requires the same normalization)
- You need de novo subtype discovery or a cohort-adaptive reference — the AWCA/mLR approach explicitly addresses the fact that standard PAM50 is affected by cohort composition, and relies on predefined external references instead
- Profiles are Affymetrix microarray data rather than RNA-seq without confirming applicability — the article discusses AWCA on Affymetrix log2-transformed data (GSE4922, GSE1456) separately from the RNA-seq mLR workflow

## Inputs

- Normalized breast cancer RNA-seq expression matrix (e.g., RSEM-normalized TCGA profiles, FPKM-normalized GSE81538/GSE96058 profiles) processed with the same normalization as training data
- Pre-trained mLR model objects for PAM50, limma50, and limma50_BWE feature sets
- Published PAM50 reference subtype labels for the test cohort (for accuracy scoring)

## Outputs

- Per-sample intrinsic subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- Per-classifier accuracy and AWCA-PAM50 concordance values compiled into a results table (cf. Table 2)

## How to apply

Load the pre-trained mLR model objects (PAM50, limma50, limma50_BWE feature sets) from the BC_Intrinsic_subtyping repository and apply each to the target expression matrix to produce one of five subtype calls per sample. The mLRs combine ad-hoc gene selection from the global transcriptome with embedded Lasso and Ridge regularizers; the limma50_BWE signature additionally derives from a backward elimination strategy. For reference-free calling, use the AWCA versions of PAM50 that use predefined external references (Supplementary Material Table S3), built to mitigate the reference-cohort bias of standard PAM50. Validate by computing classification accuracy against published PAM50 reference labels and concordance with AWCA-PAM50 calls (Table 2 reports ~90% accuracy and ~90% AWCA-PAM50 concordance for PAM50-mLR on PanCA, ~88% for limma50, and ~87% accuracy / ~91% concordance for limma50_BWE).

## Related tools

- **limma** (limma-derived feature selection supplies the limma50 and limma50_BWE gene signatures (the latter via backward elimination) used as alternative feature spaces for the regularized mLR classifiers)
- **BC_Intrinsic_subtyping** (Repository providing the R-code and pre-trained model objects for AWCA_PAM50 and the mLR intrinsic subtyping classifiers) — github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Evaluation signals

- On PanCA external testing, TCGA-trained mLR accuracies should approach internal TCGA test-set results: ~90% with PAM50 features, ~88% with limma50, ~87% with limma50_BWE
- AWCA-PAM50 concordance on PanCA should be ~90% for PAM50-mLR and ~91% for limma50_BWE (Table 2 reference values)
- Concordance achieved by AWCA-based classifications among themselves should be beyond 98%
- Every sample must receive exactly one of the five subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)
- External-testing accuracy must be computed against the published PAM50 reference labels and cross-checked against AWCA-PAM50 calls on the same cohort

## Limitations

- Performances are biased because published subtype labels were themselves obtained with the PAM50 assay on the same gene panel, inflating apparent accuracy of PAM50-gene mLRs
- limma-derived signatures brought less convincing performances on testing compared with PAM50 genes
- Performance degradation is partly caused by the Normal-like class, whose clinical significance remains undetermined
- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction; the AWCA strategy is proposed to mitigate this

## Evidence

- [abstract] mLR best performer: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome"
- [abstract] external concordance: "On external test sets, mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample"
- [discussion] AWCA predefined references: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [readme] README subtyping scope: [evidence span withheld — anchor status: not_found]
- [discussion] limma signature limitation: "limma-derived signatures brought less convincing performances on testing"
