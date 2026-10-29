---
name: grn-benchmark-auroc-tpr-evaluation
description: Use when you have one or more inferred GRNs and Perturb-seq count matrices
  (e.g., Dixit GSE90063 or Papalexi GSE153056) and need a quantitative, comparable
  ranking of network accuracy via perturbation-target prediction.
when_to_use_negative:
- You do not have Perturb-seq data with known perturbation targets — use goodness-of-fit
  (cosine similarity on held-out train/test splits) or disease subnetwork random-walk
  validation instead.
- The GRN was inferred on a subset of genes/cells inconsistent with the perturbation
  dataset, so downstream genes of the perturbed gene are not represented in the network.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: GRN inference method under benchmark evaluation; builds supercells, 100 subsampled
    GRNs merged into a final pruned network (BuildNetwork.ipynb)
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Baseline gradient-boosting GRN inference method compared against SCING in
    the perturbation-prediction benchmark
- name: PIDC
  role: Baseline information-theoretic GRN inference method compared against SCING
- name: ppcor
  role: Baseline partial-correlation GRN inference method compared against SCING
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:d679f4f047009b80f19ec7cbde339fe38bd2e8beb292421a05f8886464e36c06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# grn-benchmark-auroc-tpr-evaluation

## Summary

Benchmark single-cell GRN inference methods (SCING, GRNBOOST2, PIDC, ppcor) by predicting downstream perturbed genes in Perturb-seq datasets, scoring each method with AUROC and TPR at FPR 0.05 to quantify how accurately inferred networks capture real perturbation effects.

## When to use

Use when you have one or more inferred GRNs and Perturb-seq count matrices (e.g., Dixit GSE90063 or Papalexi GSE153056) and need a quantitative, comparable ranking of network accuracy via perturbation-target prediction.

## When NOT to use

- You do not have Perturb-seq data with known perturbation targets — use goodness-of-fit (cosine similarity on held-out train/test splits) or disease subnetwork random-walk validation instead.
- The GRN was inferred on a subset of genes/cells inconsistent with the perturbation dataset, so downstream genes of the perturbed gene are not represented in the network.

## Inputs

- Perturb-seq count matrices from GEO: Dixit et al. (GSE90063) and Papalexi et al. (GSE153056)
- Inferred GRNs for each method (SCING, GRNBOOST2, PIDC, ppcor) built on the same preprocessed datasets
- Guide/perturbation assignments linking perturbed genes to target guides

## Outputs

- perturbseq_benchmark_results.csv: AUROC and TPR-at-FPR-0.05 for each of the four methods on each of the two datasets
- Per-method lists of significantly perturbed downstream targets from permutation testing
- Separate SCING directed-edge evaluation results alongside the undirected comparison

## How to apply

Preprocess the Perturb-seq count matrices from GEO, then build GRNs for each method: for SCING, cluster cells into supercells, build 100 GRNs on subsamples of the data in parallel, merge them into a final network, and prune edges and cycles using conditional mutual information; for the baselines, run GRNBOOST2 (gradient boosting), PIDC, and ppcor on the same datasets. For each method's GRN, identify downstream perturbed genes of guides for specific genes, then fit an elastic net regression to predict the perturbation effect on downstream genes and use permutation testing to call significantly perturbed downstream targets. Predict perturbed genes at each depth in the network from the perturbed gene, computing true-positive and false-positive rates at each depth, and aggregate these into AUROC and TPR at FPR 0.05. Treat edges as undirected for all methods so comparison is fair, and additionally evaluate SCING with directed edges as a separate condition. Compile results into a table keyed by method and dataset (e.g., perturbseq_benchmark_results.csv) with one AUROC and one TPR-at-FPR-0.05 entry per method per dataset.

## Related tools

- **SCING** (GRN inference method under benchmark evaluation; builds supercells, 100 subsampled GRNs merged into a final pruned network (BuildNetwork.ipynb)) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Baseline gradient-boosting GRN inference method compared against SCING in the perturbation-prediction benchmark)
- **PIDC** (Baseline information-theoretic GRN inference method compared against SCING)
- **ppcor** (Baseline partial-correlation GRN inference method compared against SCING)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git && conda env create -n scing --file install/scing.environment.yml && conda activate scing
```

## Evaluation signals

- The results table contains exactly one AUROC and one TPR-at-FPR-0.05 entry for each of the four methods on each of the two datasets, with the SCING directed evaluation reported separately.
- Edges are treated as undirected for all methods in the head-to-head comparison; a directed SCING score appears only as an additional condition.
- SCING is expected to outperform GRNBOOST2, PIDC, and ppcor when sparsity is adjusted; deviations should be checked against sparsity-handling settings before trusting the ranking.
- TPR is reported at a fixed FPR of 0.05, so the FPR threshold used must be verifiable in the scoring code rather than re-tuned post hoc.
- Significantly perturbed downstream targets should be defined by permutation testing, not by nominal correlation or regression coefficients alone.

## Limitations

- Inferred GRN links are not necessarily causal or directional, which motivates evaluating all methods with undirected edges.
- Many hyperparameters require selection, and selection can be dataset dependent; hyperparameter optimization can affect benchmark outcomes.
- SCING outperforms other methods at predicting perturbation effects when sparsity is adjusted, but only outperforms select methods when all cells are used.

## Evidence

- [intro] intro: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [readme] readme: "We identified downstream perturbed genes of guides for specific genes. We then predict perturbed genes at each depth in the network from the perturbed gene."
- [methods] methods: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
- [discussion] discussion: "the links between genes are not necessarily causal or directional"
