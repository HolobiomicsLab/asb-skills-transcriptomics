---
name: train-test-holdout-evaluation
description: Use when benchmarking GRN inference methods on snRNA-seq data and you
  need to determine which method generalizes to held-out cells rather than merely
  fitting the training data, via a cosine similarity test/train ratio.
when_to_use_negative:
- When the validation target is true perturbation effects rather than generalization
  to held-out cells — use perturb-seq target-prediction validation (AUROC / TPR at
  FPR 0.05) instead
- When the goal is disease subnetwork or pathway interpretation of the GRN rather
  than method benchmarking — the random walk disease subnetwork framework or leiden/AUCell
  module workflows are the appropriate skills
- When you need causal or directional regulatory links — the inferred GRN links are
  not necessarily causal or directional
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: GRN inference on the training split (supercell construction, bagged GRNs from
    subsamples, merging, edge pruning); method whose generalization is being assessed
  repo: https://github.com/XiaYangLabOrg/SCING
- name: ppcor
  role: Baseline GRN inference method on the training split for overfitting comparison
- name: PIDC
  role: Baseline GRN inference method on the training split for overfitting comparison
- name: GRNBOOST2
  role: Gradient boosting baseline GRN method on the training split and the same GBR
    family used for per-gene expression prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:474a91519e337b45795813e28f7d39130df22f77a465dea01c49c293d159390d
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# train-test-holdout-evaluation

## Summary

Quantifies GRN overfitting by building networks on a training split of single-cell data, predicting held-out gene expression with per-gene gradient boosting regressors, and comparing test/train cosine similarity ratios across GRN inference methods (SCING, ppcor, PIDC, GRNBOOST2).

## When to use

Use when benchmarking GRN inference methods on snRNA-seq data and you need to determine which method generalizes to held-out cells rather than merely fitting the training data, via a cosine similarity test/train ratio.

## When NOT to use

- When the validation target is true perturbation effects rather than generalization to held-out cells — use perturb-seq target-prediction validation (AUROC / TPR at FPR 0.05) instead
- When the goal is disease subnetwork or pathway interpretation of the GRN rather than method benchmarking — the random walk disease subnetwork framework or leiden/AUCell module workflows are the appropriate skills
- When you need causal or directional regulatory links — the inferred GRN links are not necessarily causal or directional

## Inputs

- Single-cell/single-nucleus RNA-seq expression matrix (e.g. snRNA-seq counts from control human prefrontal cortex, subsetted by cell type)
- Cell type labels for subsetting populations (oligodendrocyte, astrocyte, microglia)
- Gene subsample sizes (e.g. 1,000 / 3,000 / 5,000 genes) per cell type

## Outputs

- GRNs built on the training split for each method (SCING, ppcor, PIDC, GRNBOOST2)
- Per-gene GBR predictions of held-out test-split expression
- Aggregated benchmark table of test/train cosine similarity ratios and absolute test/train cosine similarity per method, cell type, and gene-subsample size (grn_cosine_similarity_benchmark.csv)

## How to apply

Load the single-cell dataset (e.g. Morabito control human prefrontal cortex snRNA-seq), subset cell types such as oligodendrocytes, astrocytes, and microglia, and split each population 50/50 into train and test sets. Draw random gene subsamples (e.g. 1,000, 3,000, and 5,000 genes) and build GRNs on each training subset with each method — for SCING this means supercell construction, GRN inference on subsamples, merging many GRNs to a final GRN, and edge pruning. For each method's GRN, train one gradient boosting regressor (GBR) per gene using the gene's parents in the network on the train data. Predict each gene's expression on the held-out test cells and compute the cosine similarity between predicted and true expression on both splits. Aggregate the test/train cosine similarity ratio and absolute test/train cosine similarity per method, cell type, and gene-subsample size (saved as grn_cosine_similarity_benchmark.csv); a higher test/train ratio indicates less overfitting.

## Related tools

- **SCING** (GRN inference on the training split (supercell construction, bagged GRNs from subsamples, merging, edge pruning); method whose generalization is being assessed) — https://github.com/XiaYangLabOrg/SCING
- **ppcor** (Baseline GRN inference method on the training split for overfitting comparison)
- **PIDC** (Baseline GRN inference method on the training split for overfitting comparison)
- **GRNBOOST2** (Gradient boosting baseline GRN method on the training split and the same GBR family used for per-gene expression prediction)

## Examples

```
conda env create -n scing --file install/scing.environment.yml
```

## Evaluation signals

- A 50/50 train/test cell split exists per cell type before any network is built
- Cosine similarity between predicted and true expression is computed separately for train and test splits, and a test/train ratio is derivable for every method, cell type, and gene-subsample size (1,000 / 3,000 / 5,000)
- Overfitting methods show higher cosine similarity on training data than test data (test/train ratio < SCING's); in the source benchmark SCING's ratio was highest among SCING, ppcor, PIDC, GRNBOOST2
- The aggregated results are persisted in the expected schema as grn_cosine_similarity_benchmark.csv
- Test-split performance for SCING is similar to ppcor and PIDC and outperforms GRNBOOST2, serving as a sanity check against the published finding

## Limitations

- Many hyperparameters require selection, and while robustness was shown across a range, hyperparameter selection can be dataset dependent
- GRN links between genes are not necessarily causal or directional, so this overfitting metric does not validate causality
- SCING requires substantial setup (conda environment, pyitlib, optional pyscenic/enrichr) and its supercell/subsample/merge workflow is computationally intensive

## Evidence

- [readme] Gene prediction validation, to determine network overfitting: "Gene prediction validation, to determine network overfitting (b). We split data into train and test sets and build a network on the train set. A GBR is trained for each gene"
- [readme] Cosine similarity on held-out test predictions: "We then predict the expression of each gene in the test set and determine the distance from the true expression through cosine similarity."
- [intro] Comparison methods evaluated against SCING: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics"
- [methods] SCING applied to the source snRNA-seq data: "We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients"
- [discussion] Links are not causal/directional: "the links between genes are not necessarily causal or directional"
