---
name: roc-auc-method-comparison
description: Use when two or more interaction-scoring methods are applied to the same
  candidate enhancer-gene set and you must decide which performs better against perturbation-based
  ground truth (e.g., CRISPRi) via ROC/PR curves.
when_to_use_negative:
- The candidate enhancer-gene sets scored by the two methods differ — comparing ROC
  AUCs on non-shared interactions confounds score quality with candidate coverage.
- There is no perturbation-based ground truth (e.g., no CRISPRi measurements) to label
  positives, so ROC/PR evaluation is uninterpretable.
- The question is single-method calibration or threshold selection rather than comparing
  two competing scoring methods.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: STARE / stare-abc
  role: Produces the gABC-scored enhancer-gene interactions on each CRISPRi validation
    dataset that form one arm of the ROC/PR comparison
  repo: https://github.com/schulzlab/STARE
- name: pROC
  role: Computes and compares precision-recall and ROC curves and AUC values for gABC
    vs. Enformer ISM scores on each dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:bbbdd85ccce8de6d0b904c963f10fbca6cee3e54824354d4b44ad6b918938339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - STARE / stare-abc
  - pROC
  - stare-abc
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# roc-auc-method-comparison

## Summary

Head-to-head benchmarking of alternative enhancer-gene scoring methods (e.g., gABC vs. Enformer in-silico mutagenesis) by ranking shared candidate interactions under each score, labeling positives from CRISPR perturbation measurements, and comparing precision-recall curves and ROC AUCs per validation dataset with pROC.

## When to use

Use when two or more interaction-scoring methods are applied to the same candidate enhancer-gene set and you must decide which performs better against perturbation-based ground truth (e.g., CRISPRi) via ROC/PR curves.

## When NOT to use

- The candidate enhancer-gene sets scored by the two methods differ — comparing ROC AUCs on non-shared interactions confounds score quality with candidate coverage.
- There is no perturbation-based ground truth (e.g., no CRISPRi measurements) to label positives, so ROC/PR evaluation is uninterpretable.
- The question is single-method calibration or threshold selection rather than comparing two competing scoring methods.

## Inputs

- gABC-scored enhancer-gene interactions from STARE_ABCpp per validation dataset
- Enformer in-silico-mutagenesis interaction scores (2-kb replacement, 25 shuffle iterations, interactions <= 96 kb)
- CRISPR perturbation validation measurements used to label positive enhancer-gene pairs
- Shared candidate interaction set ranked by each scoring method

## Outputs

- Per-dataset precision-recall curves for each scoring method
- Per-dataset ROC AUC values for each scoring method
- Comparison table (gabc_enformer_pr_roc_comparison.csv) of gABC vs. Enformer ISM performance

## How to apply

Score the identical candidate enhancer-gene interactions with each method — here, run STARE_ABCpp per validation dataset with the candidate-region bed file, activity column (-n), Gencode gtf annotation (-a), chromatin contact folder (-f), default 5MB window and 0.02 ABC-score cutoff for gABC, and 2-kb neutral-sequence replacement averaged over 25 shuffle iterations (interactions <= 96 kb) for the Enformer ISM score. Restrict both methods to the shared candidate interactions, rank them by their respective scores, and label positives from the CRISPR perturbation validation measurements. For each dataset and method, compute precision-recall curves and ROC AUC in R with the pROC package, then assemble per-dataset values into a comparison table (gabc_enformer_pr_roc_comparison.csv). The decision rule in this study: a method wins if it attains higher accuracy (larger AUC) consistently across all three validation datasets, ideally with statistically significant AUC differences.

## Related tools

- **STARE / stare-abc** (Produces the gABC-scored enhancer-gene interactions on each CRISPRi validation dataset that form one arm of the ROC/PR comparison) — https://github.com/schulzlab/STARE
- **pROC** (Computes and compares precision-recall and ROC curves and AUC values for gABC vs. Enformer ISM scores on each dataset)

## Examples

```
library(pROC); roc_gabc <- roc(labels, gabc_scores); roc_ism <- roc(labels, ism_scores); roc.test(roc_gabc, roc_ism)
```

## Evaluation signals

- The two methods are ranked over the same shared candidate interaction set, so PR/ROC curves are computed on identical candidate counts per dataset.
- A per-dataset comparison table (gabc_enformer_pr_roc_comparison.csv) is produced with ROC AUC values for both methods on all three validation datasets.
- The gABC score attains the higher accuracy (larger AUC) on all three validation datasets, matching the reported finding.
- AUC differences between methods are tested for significance with pROC's ROC-curve comparison rather than reported from point estimates alone.

## Limitations

- The Enformer ISM arm is restricted to interactions <= 96 kb, so the comparison covers only the shared candidate range and not the full 5MB ABC window.
- IS scores require replacing each enhancer with a neutral 2-kb sequence averaged over 25 shuffle iterations, making the baseline computationally expensive.
- Conclusions hold for the three CRISPRi validation datasets; performance on other cell types or perturbation assays is not established by this comparison.

## Evidence

- [methods] Run STARE_ABCpp separately on each of the three validation datasets with the candidate-region bed file, activity column (-n), Gencode gtf annotation (-a), chromatin contact folder (-f), default 5MB window and 0.02 ABC-score cutoff: [evidence span withheld — anchor status: not_found]
- [supplementary] pROC package usage: "pROC: An open-source package for r and s+ to analyze and compare ROC curves"
- [results] gABC vs Enformer finding: [evidence span withheld — anchor status: not_found]
- [readme] STARE framework independence: "STARE is a framework that combines two tasks, which can also work independently"
