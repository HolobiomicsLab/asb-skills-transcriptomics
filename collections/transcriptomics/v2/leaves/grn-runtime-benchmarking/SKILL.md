---
name: grn-runtime-benchmarking
description: Use when comparing GRN inference methods on snRNA-seq/scRNA-seq data
  and you need quantitative topology fit (scale-free R-squared), reproducibility of
  inferred edges, and computational runtime benchmarks per method.
when_to_use_negative:
- You need causal or directional regulatory relationships — inferred GRN links are
  not necessarily causal or directional
- Your goal is functional module discovery or pathway annotation (leiden partitioning
  + AUCell + enrichr) rather than method benchmarking
- The question is perturbation-target prediction accuracy (perturb-seq AUROC / TPR
  at FPR 0.05), which is a separate validation modality
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: SCING
  role: Gradient boosting and mutual information-based GRN inference method being
    benchmarked; uses supercells and bagged subsample GRNs merged and pruned via conditional
    mutual information
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Gradient boosting-based GRN inference baseline method included in the topology
    and runtime comparison
- name: ppcor
  role: Partial correlation-based GRN inference baseline method included in the topology
    and runtime comparison
- name: PIDC
  role: Information-theory-based GRN inference baseline method included in the topology
    and runtime comparison
- name: networkx
  role: Python library used to compute network topology metrics (scale-free fit R-squared,
    edge/gene counts, betweenness centrality variance) on each inferred GRN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:15808828940051a5677ad7360de901000b746e2e17a268d5fb9aef9f17fdfb73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - networkx
  - ppcor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# grn-runtime-benchmarking

## Summary

Benchmark single-cell gene regulatory network (GRN) inference methods (SCING, GRNBOOST2, ppcor, PIDC) by computing network topology metrics — scale-free fit R-squared on the degree distribution, edge/gene counts, betweenness centrality variance, and a normalized edge overlap score between independent cell splits — alongside runtime (seconds per GRN) across gene- and cell-subset scaling tests.

## When to use

Use when comparing GRN inference methods on snRNA-seq/scRNA-seq data and you need quantitative topology fit (scale-free R-squared), reproducibility of inferred edges, and computational runtime benchmarks per method.

## When NOT to use

- You need causal or directional regulatory relationships — inferred GRN links are not necessarily causal or directional
- Your goal is functional module discovery or pathway annotation (leiden partitioning + AUCell + enrichr) rather than method benchmarking
- The question is perturbation-target prediction accuracy (perturb-seq AUROC / TPR at FPR 0.05), which is a separate validation modality

## Inputs

- snRNAseq/scRNA-seq expression matrices per cell type (e.g., oligodendrocyte, astrocyte, microglia)
- Sampled gene subsets (1,000/2,000/3,000/4,000 genes) per cell type
- 50/50 cell-split subsets per cell type
- GRN edge lists produced by SCING, GRNBOOST2, ppcor, and PIDC

## Outputs

- grn_benchmark_metrics.csv with scale-free R-squared, edge counts, gene counts, betweenness centrality variance, normalized edge overlap score, and runtime (seconds per GRN) for all method x cell type x replicate combinations
- 120 inferred GRNs (4 methods x 3 cell types x 10 replicates)

## How to apply

Load snRNAseq expression matrices per cell type (e.g., oligodendrocyte, astrocyte, microglia), sample 3,000 random genes per cell type, and split cells 50/50 into two independent subsets. Infer 10 replicate GRNs per cell type per method (SCING, GRNBOOST2, ppcor, PIDC) using each method's native implementation for 120 networks total. Compute topology metrics with networkx: scale-free fit R-squared on the degree distribution (excluding genes with expression sparsity above 0.7), edge counts, gene counts, and betweenness centrality variance. Quantify reproducibility with a normalized edge overlap score between GRNs built on the two 50/50 cell-split halves, and time GRN construction across gene subsets (1,000/2,000/4,000 genes) and cell subsets (250/500/1,000 cells) recording seconds per GRN. Aggregate all topology, overlap, and runtime measurements into a single table (e.g., grn_benchmark_metrics.csv).

## Related tools

- **SCING** (Gradient boosting and mutual information-based GRN inference method being benchmarked; uses supercells and bagged subsample GRNs merged and pruned via conditional mutual information) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Gradient boosting-based GRN inference baseline method included in the topology and runtime comparison)
- **ppcor** (Partial correlation-based GRN inference baseline method included in the topology and runtime comparison)
- **PIDC** (Information-theory-based GRN inference baseline method included in the topology and runtime comparison)
- **networkx** (Python library used to compute network topology metrics (scale-free fit R-squared, edge/gene counts, betweenness centrality variance) on each inferred GRN)

## Examples

```
conda env create -n scing --file install/scing.environment.yml
```

## Evaluation signals

- The metrics table contains complete entries for all 4 methods x 3 cell types x 10 replicates with no missing method or cell-type rows
- SCING networks show a significantly higher scale-free fit R-squared than PIDC and GRNBOOST2 (and trending higher than ppcor), confirming the expected finding
- Every GRN has scale-free R-squared computed only after excluding genes with expression sparsity above 0.7
- Normalized edge overlap scores are present for both 50/50 cell-split halves per method and cell type
- Runtime (seconds per GRN) is recorded for every combination of gene subset (1,000/2,000/4,000) and cell subset (250/500/1,000) per method

## Limitations

- Inferred GRN links are not necessarily causal or directional, so topology fit does not imply causal correctness
- Many hyperparameters require selection and selection can be dataset dependent, which affects cross-method comparability
- Existing GRN tools are limited to thousands of pre-selected genes and cells because GRN inference from full transcriptomes in large scRNA-seq datasets is computationally intensive — runtime scaling tests may not cover full-transcriptome regimes for all methods

## Evidence

- [intro] We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy.: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [intro] Existing GRN tools limited to pre-selected genes: "Existing GRN tools utilize scRNA-seq data with thousands of pre-select genes and cells, because GRNs from full transcriptomes in large scRNA-seq datasets are often computationally intensive and"
- [intro] GRNBOOST2 computing efficiency: "top performing methods in recent benchmarking studies, with GRNBOOST2 showing superior computing efficiency than GENIE3"
- [methods] SCING environment setup: "conda env create -n scing --file install/scing.environment.yml"
- [readme] Bagged subsample GRNs per README: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
