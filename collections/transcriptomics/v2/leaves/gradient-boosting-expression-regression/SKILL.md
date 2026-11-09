---
name: gradient-boosting-expression-regression
description: Use when benchmarking GRN inference methods (e.g., SCING vs ppcor, PIDC,
  GRNBOOST2) for overfitting by comparing goodness-of-fit on training cells versus
  generalization to held-out test cells.
when_to_use_negative:
- When the goal is perturbation-based validation (perturb-seq AUROC / TPR at FPR 0.05)
  rather than train/test generalization of expression prediction
- When the input network is built on all cells rather than a training split — the
  train/test cosine similarity contrast requires disjoint cell sets
- When spatial information is the target of evaluation; SCING does not utilize spatial
  information during network construction, so this skill does not assess spatial correctness
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: SCING
  role: GRN inference method whose output networks are evaluated for overfitting;
    trained per-gene GBRs use each gene's parents inferred by SCING on the training
    split
  repo: https://github.com/XiaYangLabOrg/SCING
- name: ppcor
  role: Baseline partial correlation-based GRN method compared against SCING in the
    cosine similarity overfitting benchmark
- name: PIDC
  role: Baseline mutual information-based GRN method compared against SCING in the
    overfitting evaluation
- name: GRNBOOST2
  role: Baseline gradient boosting GRN method compared against SCING; SCING outperformed
    it on test-data cosine similarity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:aafa2e2882e3f66aedd58612bfeb907899caf15238a5e6eef394dc1e27950007
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gradient-boosting-expression-regression

## Summary

Train per-gene gradient boosting regressors (GBR) on a GRN built from a training split of single-cell/snRNA-seq data, then predict held-out gene expression on a test split, using cosine similarity between predicted and true expression to quantify network overfitting.

## When to use

Use when benchmarking GRN inference methods (e.g., SCING vs ppcor, PIDC, GRNBOOST2) for overfitting by comparing goodness-of-fit on training cells versus generalization to held-out test cells.

## When NOT to use

- When the goal is perturbation-based validation (perturb-seq AUROC / TPR at FPR 0.05) rather than train/test generalization of expression prediction
- When the input network is built on all cells rather than a training split — the train/test cosine similarity contrast requires disjoint cell sets
- When spatial information is the target of evaluation; SCING does not utilize spatial information during network construction, so this skill does not assess spatial correctness

## Inputs

- Single-cell or single-nucleus RNA-seq expression matrix (e.g., Morabito human prefrontal cortex snRNA-seq, control patients) split 50/50 into train and test cells per cell type
- Inferred GRN (edge list of gene-gene regulatory links) from each method: SCING, ppcor, PIDC, GRNBOOST2
- Gene subsample sizes (1,000 / 3,000 / 5,000 genes) defining the feature space per run

## Outputs

- Per-gene cosine similarity of predicted vs. true expression on train and test splits for each method, cell type, and gene-subsample size
- Aggregated test/train cosine similarity ratio and absolute test/train cosine similarity per method (grn_cosine_similarity_benchmark.csv)

## How to apply

Split each cell type's cells 50/50 into train and test sets (e.g., Morabito control human prefrontal cortex snRNA-seq oligodendrocytes, astrocytes, and microglia), optionally drawing random gene subsamples of 1,000, 3,000, and 5,000 genes. Build the GRN on the training split with each method — for SCING this means supercell construction, GRN inference on 100 subsampled networks, merging, and edge pruning via conditional mutual information. For every gene, train a GBR using its inferred parents (regulators) in the training data, then predict that gene's expression on the held-out test cells. Compute cosine similarity between predicted and true expression separately on train and test splits, and aggregate the test/train cosine similarity ratio per method (saved as grn_cosine_similarity_benchmark.csv): a high training similarity with low test similarity indicates overfitting, so the method with the highest test/train ratio generalizes best.

## Related tools

- **SCING** (GRN inference method whose output networks are evaluated for overfitting; trained per-gene GBRs use each gene's parents inferred by SCING on the training split) — https://github.com/XiaYangLabOrg/SCING
- **ppcor** (Baseline partial correlation-based GRN method compared against SCING in the cosine similarity overfitting benchmark)
- **PIDC** (Baseline mutual information-based GRN method compared against SCING in the overfitting evaluation)
- **GRNBOOST2** (Baseline gradient boosting GRN method compared against SCING; SCING outperformed it on test-data cosine similarity)

## Examples

```
conda env create -n scing --file install/scing.environment.yml
```

## Evaluation signals

- The test/train cosine similarity ratio for SCING should exceed that of ppcor, PIDC, and GRNBOOST2 (SCING showed the least overfitting in the paper)
- Methods that overfit show higher cosine similarity on training data than on test data; check that train cosine similarity > test cosine similarity for the overfitting baselines
- Benchmark table must contain one row per method x cell type x gene-subsample size (1,000/3,000/5,000) with both ratio and absolute test/train cosine similarity columns
- SCING's test-data cosine similarity should perform similarly to ppcor and PIDC while outperforming GRNBOOST2
- Train/test split must be disjoint 50/50 per cell type — predictions on training cells must never contribute to the test cosine similarity

## Limitations

- Hyperparameter selection for network construction can be dataset dependent; optimization of hyperparameters is required per dataset
- Inferred GRN links are not necessarily causal or directional, so GBR prediction quality reflects associative regulatory structure, not causality
- SCING does not currently utilize spatial information during network construction, so this evaluation cannot certify spatially correct regulation

## Evidence

- [readme] Gene prediction validation, to determine network overfitting: "Gene prediction validation, to determine network overfitting (b). We split data into train and test sets and build a network on the train set."
- [readme] Per-gene GBR training and cosine similarity evaluation: "A GBR is trained for each gene based on its parents in the train data. We then predict the expression of each gene in the test set and determine the distance"
- [methods] Cosine similarity metric definition: "istance from the true expression through cosine similarity"
- [methods] SCING network construction steps feeding the benchmark: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges"
