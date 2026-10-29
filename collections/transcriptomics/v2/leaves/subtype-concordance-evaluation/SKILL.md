---
name: subtype-concordance-evaluation
description: Use when subtype calls produced by AWCA or mLR models on RNA-seq expression
  matrices (TCGA RSEM, GSE96058 FPKM, PanCA, GSE81538) must be scored against published
  PAM50 labels or against another caller's calls via concordance/accuracy.
when_to_use_negative:
- Input is raw/unnormalized RNA-seq data — callers must be applied to expression matrices
  normalized the same way as the training cohorts (TCGA RSEM, GSE96058 FPKM)
- Data are Affymetrix microarray profiles (e.g. GSE4922, GSE1456) — AWCA-based classification
  there used log2-transformed microarray data, a distinct setting
- Subtype labels for the comparison were themselves not PAM50-derived — accuracy vs
  published PAM50 calls is acknowledged as biased toward PAM50-gene-based methods
tools:
- name: limma
  role: Used to derive the limma50 and limma50_BWE gene signatures whose mLR subtype
    calls are scored in the concordance/accuracy evaluation
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:53c142a8a23ab90ed2f1341c425a61b0a49ea882a259fb08f41195eba69c23a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - limma
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# subtype-concordance-evaluation

## Summary

Evaluates breast cancer intrinsic-subtyping callers (AWCA-PAM50 and regularized multiclass logistic regression, mLR) by computing concordance and classification accuracy of their subtype calls against published PAM50 reference labels across internal and external RNA-seq cohorts (TCGA, GSE96058, PanCA, GSE81538).

## When to use

Use when subtype calls produced by AWCA or mLR models on RNA-seq expression matrices (TCGA RSEM, GSE96058 FPKM, PanCA, GSE81538) must be scored against published PAM50 labels or against another caller's calls via concordance/accuracy.

## When NOT to use

- Input is raw/unnormalized RNA-seq data — callers must be applied to expression matrices normalized the same way as the training cohorts (TCGA RSEM, GSE96058 FPKM)
- Data are Affymetrix microarray profiles (e.g. GSE4922, GSE1456) — AWCA-based classification there used log2-transformed microarray data, a distinct setting
- Subtype labels for the comparison were themselves not PAM50-derived — accuracy vs published PAM50 calls is acknowledged as biased toward PAM50-gene-based methods

## Inputs

- Pre-trained mLR model objects for the PAM50, limma50, and limma50_BWE feature sets (from BC_Intrinsic_subtyping repo)
- External RNA-seq test expression matrices (PanCA, GSE81538) processed with the same normalization as training data (TCGA RSEM, GSE96058 FPKM)
- Published PAM50 reference subtype labels per sample
- Internal and external AWCA reference subtype calls per cohort

## Outputs

- Per-classifier accuracy values against published PAM50 reference labels (e.g. Table 2: 90% PAM50-mLR, 88% limma50-mLR, 87% limma50_BWE-mLR on PanCA)
- Per-classifier concordance with AWCA-PAM50 calls (e.g. ~90% PAM50-mLR, ~91% limma50_BWE on PanCA)
- Internal and external AWCA concordance table (awca_concordance_results.csv)

## How to apply

Load pre-trained mLR model objects (PAM50, limma50, and limma50_BWE feature sets) from the DEIB-GECO/BC_Intrinsic_subtyping repository together with the external test expression matrix (e.g. PanCA). Apply each classifier to every sample to produce single-sample calls for the five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like), then compute per-classifier accuracy against published PAM50 reference labels and concordance with AWCA-PAM50 calls. For AWCA evaluation, build internal references per cohort via Spearman-correlation nearest-centroid assignment and compute pairwise concordance; then apply external references (best TCGA AWCA to PanCA, best GSE96058 AWCA to GSE81538) and compute concordance of external-reference calls against internal-reference calls. Assemble per-classifier accuracy and concordance values into a results table (e.g. awca_concordance_results.csv).

## Related tools

- **limma** (Used to derive the limma50 and limma50_BWE gene signatures whose mLR subtype calls are scored in the concordance/accuracy evaluation)

## Evaluation signals

- Reproduced PAM50-mLR accuracy on PanCA ≈ 90% and AWCA-PAM50 concordance ≈ 90%, matching Table 2
- limma50_BWE-mLR on PanCA shows ~87% accuracy and ~91% AWCA-PAM50 concordance
- External AWCA reference concordance (TCGA→PanCA, GSE96058→GSE81538) exceeds 96%
- Internal AWCA concordance for GSE96058 is at least 95% and ~95% for PanCA and GSE81538
- Calls are restricted to the five prognosis-related intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)

## Limitations

- Accuracy measured against published PAM50 calls is biased in favor of mLRs using PAM50 genes or AWCA-PAM50, since those published labels came from the PAM50 assay on the same gene panel
- Performance degradation is partly caused by the Normal-like class, whose clinical significance remains undetermined
- Few training samples and a vast number of features limit subtyping capability for hard-to-recognize (ambiguous) samples
- limma-derived signatures gave less convincing performances on external testing than PAM50-gene or AWCA-PAM50 approaches

## Evidence

- [abstract] mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample: "mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample"
- [abstract] AWCA concordance threshold: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance"
- [discussion] External AWCA references: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [readme] Five intrinsic subtypes: "assign breast cancer (BC) samples to the five prognosis-related breast cancer _intrinsic_ subtypes (_Luminal A, Luminal B, Basal, Her2-enriched, or Normal-like_)"
