---
name: scale-free-degree-distribution-fitting
description: Use when comparing GRN inference methods (e.g., SCING vs. GRNBOOST2,
  ppcor, PIDC) on topology quality, or when judging whether inferred single-cell network
  structures approximate a scale-free degree distribution.
when_to_use_negative:
- When the goal is perturbation target prediction accuracy (e.g., Perturb-seq AUROC
  evaluation) rather than network topology goodness of fit
- When the network is a known heterogeneous, hierarchical biological network for which
  scale-free structure is not an expected model
- When evaluating disease subnetwork modeling via the random walk framework of Huang
  et al. — a different validation modality
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: GRN inference method whose networks are fit to a scale-free structure; networks
    were found to follow scale-free structure more closely than PIDC, GRNBOOST2, and
    ppcor
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Gradient boosting-based comparison GRN method included in the topology benchmark
- name: ppcor
  role: Partial correlation-based comparison GRN method included in the topology benchmark
- name: PIDC
  role: Mutual information-based comparison GRN method included in the topology benchmark
- name: networkx
  role: Python library used to compute degree distribution fits, edge/gene counts,
    and betweenness centrality variance on each GRN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:ad0e238b73c3999e4c0fcf37f1eb2a91a3cc6f783802c1374b673db8b789dcca
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scale-free-degree-distribution-fitting

## Summary

Quantify how closely inferred gene regulatory networks (GRNs) follow a scale-free network structure by fitting the degree distribution and reporting R-squared, alongside edge counts, gene counts, and betweenness centrality variance, as done when benchmarking SCING against ppcor, PIDC, and GRNBOOST2.

## When to use

Use when comparing GRN inference methods (e.g., SCING vs. GRNBOOST2, ppcor, PIDC) on topology quality, or when judging whether inferred single-cell network structures approximate a scale-free degree distribution.

## When NOT to use

- When the goal is perturbation target prediction accuracy (e.g., Perturb-seq AUROC evaluation) rather than network topology goodness of fit
- When the network is a known heterogeneous, hierarchical biological network for which scale-free structure is not an expected model
- When evaluating disease subnetwork modeling via the random walk framework of Huang et al. — a different validation modality

## Inputs

- Inferred GRN objects/edge lists (e.g., from SCING, ppcor, PIDC, GRNBOOST2) built on snRNA-seq expression matrices
- snRNA-seq expression matrices per cell type (e.g., oligodendrocyte, astrocyte, microglia) with ~3,000 sampled genes
- Gene-level expression sparsity values (to exclude genes with sparsity above 0.7)

## Outputs

- Scale-free fit R-squared per GRN degree distribution
- Edge counts, gene counts, and betweenness centrality variance per GRN
- Normalized edge overlap score between networks built on 50/50 cell-split halves
- Aggregated metrics table (e.g., grn_benchmark_metrics.csv) covering method x cell type x replicate

## How to apply

For each replicate GRN built from snRNA-seq expression matrices (e.g., oligodendrocyte, astrocyte, and microglia cell types with ~3,000 random genes per cell type), load the network into networkx and fit the degree distribution to a scale-free model, excluding genes with expression sparsity above 0.7 from the degree computation. Record the scale-free fit R-squared per network, along with edge counts, gene counts, and betweenness centrality variance, computing these for every replicate (e.g., 10 replicates per cell type per method). Build networks on two independent 50/50 cell-split halves to additionally compute a normalized edge overlap score as a reproducibility companion metric. Aggregate topology, overlap, and per-GRN runtime measurements (timed across gene subsets like 1,000/2,000/4,000 genes and cell subsets like 250/500/1,000 cells) into a single metrics table such as grn_benchmark_metrics.csv. Validate completeness by confirming every method x cell type x replicate combination has no missing values.

## Related tools

- **SCING** (GRN inference method whose networks are fit to a scale-free structure; networks were found to follow scale-free structure more closely than PIDC, GRNBOOST2, and ppcor) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Gradient boosting-based comparison GRN method included in the topology benchmark)
- **ppcor** (Partial correlation-based comparison GRN method included in the topology benchmark)
- **PIDC** (Mutual information-based comparison GRN method included in the topology benchmark)
- **networkx** (Python library used to compute degree distribution fits, edge/gene counts, and betweenness centrality variance on each GRN)

## Examples

```
conda env create -n scing --file install/scing.environment.yml && conda activate scing  # then compute networkx degree-distribution metrics on the inferred GRNs
```

## Evaluation signals

- The metrics table (grn_benchmark_metrics.csv) contains complete entries for all 4 methods x 3 cell types x 10 replicates (120 networks) with no missing method or cell-type rows
- SCING scale-free R-squared is significantly higher than PIDC and GRNBOOST2 and trending higher than ppcor, as expected from the published finding
- Genes with expression sparsity above 0.7 are excluded from the degree distribution fit
- Edge overlap scores are computed on networks built from two independent 50/50 cell splits, confirming reproducibility of inferred edges
- Runtime rows exist for each method across gene subsets (1,000/2,000/4,000) and cell subsets (250/500/1,000) in seconds per GRN

## Limitations

- Inferred GRN links are not necessarily causal or directional, so a good scale-free fit does not imply correct edge direction
- Many hyperparameters require selection, and selection can be dataset dependent and may need optimization per dataset
- Expression sparsity and cell-to-cell heterogeneity in single-cell data limit existing GRN tool accuracy

## Evidence

- [intro] intro: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [readme] readme: "We utilize bagging through subsamples of supercells to keep robust edges in the final GRN. For each subsample, the genes are clustered to only operate on likely regulatory edges."
- [discussion] discussion: "the links between genes are not necessarily causal or directional"
