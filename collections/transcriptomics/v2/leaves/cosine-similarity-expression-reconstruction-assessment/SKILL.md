---
name: cosine-similarity-expression-reconstruction-assessment
description: Use when you need to measure how much a GRN inference method (e.g., SCING
  vs ppcor, PIDC, GRNBOOST2) overfits, by comparing per-gene expression reconstruction
  quality (cosine similarity) on train vs held-out test cells from snRNA-seq or scRNA-seq
  data.
when_to_use_negative:
- You lack a held-out test split — training and evaluating the GBR on the same cells
  only yields train cosine similarity and cannot reveal overfitting
- You are evaluating perturbation-target prediction (AUROC / TPR at FPR 0.05 from
  Perturb-seq guides), which is a separate validation scheme in the SCING paper
- Your network edges are needed as causal/directional regulatory claims — the paper
  notes links between genes are not necessarily causal or directional, so cosine-similarity
  reconstruction quality does not establish causality
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: GRN inference method whose networks are evaluated for overfitting via the
    cosine similarity test/train ratio; builds supercells, bagged subsample GRNs,
    merges and prunes edges
  repo: https://github.com/XiaYangLabOrg/SCING
- name: ppcor
  role: Baseline partial-correlation GRN method compared against SCING in the train/test
    cosine similarity benchmark
- name: PIDC
  role: Baseline information-theoretic GRN method compared against SCING in the train/test
    cosine similarity benchmark
- name: GRNBOOST2
  role: Baseline gradient-boosting GRN method compared against SCING; showed higher
    train than test cosine similarity (overfitting)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:76d01ed7ea9df5676a1ad45438ad2ee5d069548dec8b56acfb7b2a473c9bd276
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cosine-similarity-expression-reconstruction-assessment

## Summary

A gene-prediction validation protocol that quantifies GRN overfitting: a gradient boosting regressor (GBR) is trained per gene from its parents in a network built on a train split, expression is predicted on the held-out test split, and the cosine similarity to true expression is compared between train and test (test/train ratio).

## When to use

Use when you need to measure how much a GRN inference method (e.g., SCING vs ppcor, PIDC, GRNBOOST2) overfits, by comparing per-gene expression reconstruction quality (cosine similarity) on train vs held-out test cells from snRNA-seq or scRNA-seq data.

## When NOT to use

- You lack a held-out test split — training and evaluating the GBR on the same cells only yields train cosine similarity and cannot reveal overfitting
- You are evaluating perturbation-target prediction (AUROC / TPR at FPR 0.05 from Perturb-seq guides), which is a separate validation scheme in the SCING paper
- Your network edges are needed as causal/directional regulatory claims — the paper notes links between genes are not necessarily causal or directional, so cosine-similarity reconstruction quality does not establish causality

## Inputs

- snRNA-seq expression matrix (e.g., Morabito control human prefrontal cortex data) with oligodendrocyte, astrocyte, and microglia cell-type labels
- 50/50 train/test cell split per cell type
- Random gene subsample sizes (1,000, 3,000, 5,000 genes)
- Inferred GRN edge lists (one per method: SCING, ppcor, PIDC, GRNBOOST2) built on the training split

## Outputs

- Per-gene cosine similarity between predicted and true expression on train and test splits, per method, cell type, and gene-subsample size
- Test/train cosine similarity ratio per method (overfitting metric)
- Benchmark table saved as grn_cosine_similarity_benchmark.csv

## How to apply

Split each cell type's cells 50/50 into train and test sets, and draw random gene subsamples (e.g., 1,000, 3,000, and 5,000 genes) for each cell type. Build GRNs on the training split with each method — for SCING this means building supercells, building many GRNs on subsamples, merging to a final GRN, and pruning edges; for the baselines use ppcor, PIDC, and GRNBOOST2 directly. Following the paper's validation scheme, train a GBR for each gene based on its parents in the train-data network, then predict each gene's expression in the test cells and compute the cosine similarity between predicted and true expression for both train and test splits. Aggregate the absolute train/test cosine similarities and the test/train cosine similarity ratio per method, cell type, and gene-subsample size, saving the benchmark table as grn_cosine_similarity_benchmark.csv; a higher test/train ratio (as SCING showed vs ppcor, PIDC, and GRNBOOST2) indicates less overfitting.

## Related tools

- **SCING** (GRN inference method whose networks are evaluated for overfitting via the cosine similarity test/train ratio; builds supercells, bagged subsample GRNs, merges and prunes edges) — https://github.com/XiaYangLabOrg/SCING
- **ppcor** (Baseline partial-correlation GRN method compared against SCING in the train/test cosine similarity benchmark)
- **PIDC** (Baseline information-theoretic GRN method compared against SCING in the train/test cosine similarity benchmark)
- **GRNBOOST2** (Baseline gradient-boosting GRN method compared against SCING; showed higher train than test cosine similarity (overfitting))

## Examples

```
from scipy.spatial.distance import cosine; sim_test = 1 - cosine(y_test[g], gbr[g].predict(X_test[gParents]))  # per gene g, then ratio vs train split
```

## Evaluation signals

- The output grn_cosine_similarity_benchmark.csv contains rows for every combination of method (SCING, ppcor, PIDC, GRNBOOST2), cell type, and gene-subsample size (1,000 / 3,000 / 5,000)
- All cosine similarity values fall within the valid range [-1, 1] (absolute similarity reported per train and test split)
- The test/train cosine similarity ratio is computed per method and, as reported in the source finding, SCING's ratio is higher than ppcor, PIDC, and GRNBOOST2, whose train similarities exceed their test similarities
- The cell split is a 50/50 train/test partition within each cell type, and GRNs (and the per-gene GBRs) are fit exclusively on the training split before test prediction
- SCING's test-split reconstruction performs similarly to ppcor and PIDC and outperforms GRNBOOST2, providing a sanity check on relative rankings

## Limitations

- SCING hyperparameters require selection, and while robustness is shown across a range, optimization can be dataset dependent, which affects the reconstructed networks being evaluated
- Inferred GRN links are not necessarily causal or directional, so the cosine similarity metric reflects expression reconstruction, not causal correctness
- Results are stratified by gene-subsample size (1,000–5,000 genes) and cell type; conclusions may not transfer to full-transcriptome or other tissue contexts without re-running the benchmark

## Evidence

- [readme] GBR train/test validation scheme: "We split data into train and test sets and build a network on the train set. A GBR is trained for each gene based on its parents in the train data."
- [readme] cosine similarity metric on test predictions: "We then predict the expression of each gene in the test set and determine the distance from the true expression through cosine similarity."
- [methods] SCING applied to the benchmark-relevant dataset: "We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients"
