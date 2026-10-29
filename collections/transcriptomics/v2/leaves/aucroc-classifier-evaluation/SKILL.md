---
name: aucroc-classifier-evaluation
description: 'Use when a set of high-weight pathways from a disease-related module
  must be validated as a predictive signature: bulk RNA-seq expression matrices with
  binary phenotype labels (e.g., MMR/MSI status) are available and held-out + external-cohort
  AUROC is the acceptance criterion.'
when_to_use_negative:
- The study has fewer than 15 samples — the article notes scPAFA requires at least
  15 samples for MOFA-based module identification, so no high-weight module features
  can be extracted
- The analysis goal is single-cell per-cell scoring (PAS) rather than bulk-cohort
  signature validation — use scPAFA's PAS/MOFA steps instead
- A trained model or precomputed per-sample feature table already exists and only
  reporting/metrics are needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides the L1-regularized LinearSVC used to train the module-pathway classifier
    (C=0.2) and Platt scaling of decision values
- name: GSVA
  role: Computes per-sample pathway variation scores for the ~30 high-weight module
    pathways on bulk expression matrices
- name: MOFA
  role: Supplies the factor weights used to select the top-30 pathways that become
    classifier features
  repo: https://github.com/bioFAM/mofapy2
- name: scPAFA
  role: Upstream tool whose multicellular pathway modules (pseudobulk PAS + MOFA)
    yield the disease-related pathways validated by this classifier
  repo: https://github.com/ZhuoliHuang/scPAFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:f3ba436989fa4b9e88edc1071531b5512327b523ec85c21906e758797a944c69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioFAM/mofapy2
  tools:
  - GSVA
  - MOFA
  - Python
  - scPAFA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# aucroc-classifier-evaluation

## Summary

Validates disease-related multicellular pathway modules by converting high-weight MOFA factor pathways into bulk-RNA features and training an L1-regularized LinearSVC whose AUROC on held-out and independent cohorts (GSE39582, TCGA COAD-READ) must match the published ~0.91.

## When to use

Use when a set of high-weight pathways from a disease-related module must be validated as a predictive signature: bulk RNA-seq expression matrices with binary phenotype labels (e.g., MMR/MSI status) are available and held-out + external-cohort AUROC is the acceptance criterion.

## When NOT to use

- The study has fewer than 15 samples — the article notes scPAFA requires at least 15 samples for MOFA-based module identification, so no high-weight module features can be extracted
- The analysis goal is single-cell per-cell scoring (PAS) rather than bulk-cohort signature validation — use scPAFA's PAS/MOFA steps instead
- A trained model or precomputed per-sample feature table already exists and only reporting/metrics are needed

## Inputs

- List of high-weight pathways (pathway name + gene set) extracted from the MOFA factor/module weights
- Bulk RNA-seq expression matrix for the discovery cohort (e.g., GSE39582) with binary phenotype labels (MMR/MSI status)
- Bulk RNA-seq expression matrix for an independent test cohort (e.g., TCGA COAD-READ) with phenotype labels

## Outputs

- Trained L1-regularized LinearSVC model (C selected by 4-fold CV)
- Per-cohort AUROC values written to mmr_signature_auroc_results.csv
- Platt-scaled decision values / predicted probabilities for held-out test and independent-cohort samples

## How to apply

Extract the ~30 pathways carrying the highest MOFA factor weights from the disease-related multicellular pathway module identified by scPAFA (pseudobulk PAS analyzed via MOFA). Compute GSVA pathway scores for these pathways on the bulk expression matrix (e.g., GSE39582) and on an independent cohort (TCGA COAD-READ). Split the discovery cohort 70/30 into train/test, then tune an L1-regularized LinearSVC by 4-fold cross-validation — the paper's MMR-status classifier selected C=0.2. Train on the training split, predict decision values on the held-out test set and the independent cohort, convert decision values to probabilities via Platt scaling, and compute AUROC per cohort (reported ~0.91 in both). Write per-cohort AUROC values to a results CSV (mmr_signature_auroc_results.csv) and compare against the reported figure (Fig. 3d–g).

## Related tools

- **scikit-learn** (Provides the L1-regularized LinearSVC used to train the module-pathway classifier (C=0.2) and Platt scaling of decision values)
- **GSVA** (Computes per-sample pathway variation scores for the ~30 high-weight module pathways on bulk expression matrices)
- **MOFA** (Supplies the factor weights used to select the top-30 pathways that become classifier features) — https://github.com/bioFAM/mofapy2
- **scPAFA** (Upstream tool whose multicellular pathway modules (pseudobulk PAS + MOFA) yield the disease-related pathways validated by this classifier) — https://github.com/ZhuoliHuang/scPAFA

## Examples

```
from sklearn.svm import LinearSVC; clf = LinearSVC(penalty='l1', C=0.2, dual=False, max_iter=5000); clf.fit(X_train, y_train); scores = clf.decision_function(X_test)
```

## Evaluation signals

- Held-out GSE39582 test-set AUROC matches the reported ~0.91 shown in Fig. 3d-g
- Independent TCGA COAD-READ AUROC also matches ~0.91, confirming the signature generalizes
- Regularization parameter equals C=0.2, as selected by 4-fold cross-validation on the training split
- mmr_signature_auroc_results.csv exists and contains one AUROC row per evaluated cohort
- Feature matrix row count equals the number of GSVA-scored samples and column count equals the ~30 high-weight pathways

## Limitations

- Pathway selection is a supervised process; using a redundant pathway set may increase the complexity of results, while too few or biologically misaligned pathways may cause information loss
- Module features derive from pseudobulk integration of the PAS matrix, which leads to information loss since PAS of multiple cells is aggregated
- MOFA is a linear model — interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states

## Evidence

- [discussion] These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers.: "These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers."
- [intro] we benchmarked scPAFA’s performance on colorectal cancer (CRC) datasets22,23 together with a large-scale lupus atlas3: "we benchmarked scPAFA’s performance on colorectal cancer (CRC) datasets22,23 together with a large-scale lupus atlas3"
- [readme] In step 4, together with sample-level clinical metadata, disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by statistical analysis.: "In step 4, together with sample-level clinical metadata, disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by"
