---
name: elastic-net-perturbation-target-regression
description: Use when you have an inferred GRN (from SCING, GRNBOOST2, PIDC, or ppcor)
  plus Perturb-seq data with guide-to-gene assignments, and you need to quantify how
  accurately the GRN predicts downstream perturbed genes.
when_to_use_negative:
- The scRNA-seq dataset has no Perturb-seq guide/perturbation annotations, so downstream
  perturbed targets and TPR/FPR cannot be defined
- 'You only need to build or prune the GRN itself (BuildNetwork.ipynb steps: supercells,
  subsampled GRNs, merging, edge pruning) rather than validate it against perturbation
  data'
- You are evaluating goodness of fit (cosine similarity to held-out test-set expression)
  or disease subnetwork modeling (random walk framework from Huang et al.) instead
  of perturbation target prediction
tools:
- name: SCING
  role: GRN under evaluation; its merged, pruned network provides the edges traversed
    from the perturbed gene for elastic net prediction, evaluated with undirected
    and directed edges
  repo: github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Baseline GRN inference method whose network edges are benchmarked with the
    same elastic net perturbation-target-prediction evaluation
- name: PIDC
  role: Baseline GRN inference method whose network edges are benchmarked with the
    same elastic net perturbation-target-prediction evaluation
- name: ppcor
  role: Baseline GRN inference method whose network edges are benchmarked with the
    same elastic net perturbation-target-prediction evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:4f1c1ea15ba0fc41bc0e857d1db7b0595011a77081148b2809f2fbe6e91d9f87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# elastic-net-perturbation-target-regression

## Summary

Validates an inferred gene regulatory network by fitting an elastic net regression to predict downstream perturbation effects in Perturb-seq data, identifying significantly perturbed targets via permutation testing and scoring with AUROC and TPR at FPR 0.05. This is the benchmark used in the SCING paper to compare GRN inference quality.

## When to use

Use when you have an inferred GRN (from SCING, GRNBOOST2, PIDC, or ppcor) plus Perturb-seq data with guide-to-gene assignments, and you need to quantify how accurately the GRN predicts downstream perturbed genes.

## When NOT to use

- The scRNA-seq dataset has no Perturb-seq guide/perturbation annotations, so downstream perturbed targets and TPR/FPR cannot be defined
- You only need to build or prune the GRN itself (BuildNetwork.ipynb steps: supercells, subsampled GRNs, merging, edge pruning) rather than validate it against perturbation data
- You are evaluating goodness of fit (cosine similarity to held-out test-set expression) or disease subnetwork modeling (random walk framework from Huang et al.) instead of perturbation target prediction

## Inputs

- Inferred GRN from SCING (100 subsampled networks merged into a final network with pruned edges and cycles via conditional mutual information)
- Inferred GRNs from baseline methods GRNBOOST2, PIDC, and ppcor on the same datasets
- Perturb-seq count matrices (Dixit et al., GSE90063; Papalexi et al., GSE153056) with guide-to-perturbed-gene assignments
- Definition of perturbed genes of guides for specific genes and their downstream genes at each depth in the network

## Outputs

- AUROC and TPR-at-FPR-0.05 scores for each method on each Perturb-seq dataset
- perturbseq_benchmark_results.csv keyed by method and dataset, with SCING directed evaluation reported separately
- Lists of significantly perturbed downstream targets per method identified by permutation testing

## How to apply

For each method's GRN (SCING, GRNBOOST2, ppcor, PIDC), traverse the network from the perturbed gene and, at each depth, fit an elastic net regression model to predict the perturbation effect on downstream genes, using the GRN edges as the feature/gene-selection structure. Apply permutation testing to identify significantly perturbed downstream targets. Compute the true positive rate and false positive rate at each depth in the network from the perturbed gene, and summarize performance with AUROC and TPR at an FPR of 0.05. Treat edges as undirected for all methods, and additionally evaluate SCING with directed edges as a separate condition. Compile results into a table keyed by method and dataset (e.g., Dixit et al. GSE90063 and Papalexi et al. GSE153056) and save as perturbseq_benchmark_results.csv. Validate that the table contains one AUROC and one TPR-at-FPR-0.05 entry for each of the four methods on each of the two Perturb-seq datasets, with the SCING directed evaluation reported separately.

## Related tools

- **SCING** (GRN under evaluation; its merged, pruned network provides the edges traversed from the perturbed gene for elastic net prediction, evaluated with undirected and directed edges) — github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Baseline GRN inference method whose network edges are benchmarked with the same elastic net perturbation-target-prediction evaluation)
- **PIDC** (Baseline GRN inference method whose network edges are benchmarked with the same elastic net perturbation-target-prediction evaluation)
- **ppcor** (Baseline GRN inference method whose network edges are benchmarked with the same elastic net perturbation-target-prediction evaluation)

## Evaluation signals

- perturbseq_benchmark_results.csv contains exactly one AUROC and one TPR-at-FPR-0.05 entry for each of the four methods (SCING, GRNBOOST2, PIDC, ppcor) on each of the two Perturb-seq datasets
- SCING directed-edge evaluation is reported separately from the undirected comparison applied uniformly to all methods
- TPR and FPR are computed at each depth in the network from the perturbed gene, and AUROC/TPR-at-FPR-0.05 are derived from these depth-wise curves
- Significantly perturbed downstream targets are flagged by permutation testing before scoring
- SCING outperforms all other GRN inference approaches at predicting perturbation effects when sparsity is adjusted, and outperforms select methods when all cells are used

## Limitations

- Links between genes in the inferred GRN are not necessarily causal or directional, which is why edges are treated as undirected for the main comparison
- Hyperparameter selection is dataset dependent; while robustness is shown across a range of hyperparameters, optimization is required per dataset
- Baseline methods are limited to thousands of pre-selected genes and cells because GRN inference from full transcriptomes is computationally intensive and intractable

## Evidence

- [readme] Perturb-seq validation depth-wise metrics: "We identified downstream perturbed genes of guides for specific genes. We then predict perturbed genes at each depth in the network from the perturbed gene."
- [readme] AUROC and TPR at FPR 0.05 metrics: "True positive rate, and false positive rate are determined at each depth in the network. We utilize AUROC and TPR at FPR 0.05 as metrics for evaluation."
- [other] Elastic net workflow from source card: [evidence span withheld — anchor status: not_found]
- [intro] Benchmark scope against baselines: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
