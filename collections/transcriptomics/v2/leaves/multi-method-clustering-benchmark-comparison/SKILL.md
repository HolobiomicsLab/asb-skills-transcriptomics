---
name: multi-method-clustering-benchmark-comparison
description: Use when comparing several single-cell clustering methods on the same
  scRNA-seq datasets and you need quantitative stability (APN) and tightness (silhouette
  width) metrics for a benchmark table.
when_to_use_negative:
- The analysis goal is supervised cell-type annotation (e.g., MetaNeighbor-style replication
  assessment) rather than unsupervised clustering benchmarking
- Data contain expected rare cell types, since filtering genes with excessive zeros
  may be problematic for rare cell type identification
- Only a single individual is present and no multi-individual batch-effect comparison
  is needed (BAMMSC falls back to DIMMSC; the population-study comparison is not meaningful)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BAMM-SC
  role: Primary Bayesian mixture model whose clustering memberships (mem) are benchmarked
    for APN and silhouette width
  repo: https://github.com/lichen-lab/BAMMSC
- name: SIMLR
  role: Competing scRNA-seq clustering method run on the same perturbed and full data
    for APN/silhouette comparison
- name: CellTree
  role: Competing scRNA-seq clustering method run on the same perturbed and full data
    for APN/silhouette comparison
- name: SC3
  role: Competing scRNA-seq clustering method run on the same perturbed and full data
    for APN/silhouette comparison
- name: TSCAN
  role: Competing scRNA-seq clustering method run on the same perturbed and full data
    for APN/silhouette comparison
- name: DIMM-SC
  role: Competing scRNA-seq clustering method (single-individual Dirichlet multinomial
    mixture) used in the benchmark
- name: scran
  role: Implementation of mutual nearest neighbors (MNN) batch correction used as
    a competing clustering pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:21318f774582dd483ed6a7c279b5408096e0a5423c6c2a2dd806046468828014
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - CellTree
  - DIMM-SC
  - SC3
  - SIMLR
  - TSCAN
  - scran
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multi-method-clustering-benchmark-comparison

## Summary

Benchmark BAMM-SC against nine competing scRNA-seq clustering methods (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MNN/scran, etc.) on the three real droplet-based datasets using cluster stability (average proportion of nonoverlap, APN) and cluster tightness (average silhouette width), reproducing Supplementary Tables 7-8.

## When to use

Use when comparing several single-cell clustering methods on the same scRNA-seq datasets and you need quantitative stability (APN) and tightness (silhouette width) metrics for a benchmark table.

## When NOT to use

- The analysis goal is supervised cell-type annotation (e.g., MetaNeighbor-style replication assessment) rather than unsupervised clustering benchmarking
- Data contain expected rare cell types, since filtering genes with excessive zeros may be problematic for rare cell type identification
- Only a single individual is present and no multi-individual batch-effect comparison is needed (BAMMSC falls back to DIMMSC; the population-study comparison is not meaningful)

## Inputs

- UMI count matrices (gene x cell) for each individual from three real droplet-based scRNA-seq datasets: human PBMC, mouse lung, human skin
- List of top 1000 highly variable genes per dataset (after filtering genes with excessive zeros)
- Dataset-appropriate number of clusters K per dataset
- Cluster membership vectors (mem) from reference and perturbed runs of each clustering method

## Outputs

- stability_metrics.csv with APN and average silhouette width for BAMM-SC and nine competing methods across the three datasets
- Reproduction of Supplementary Tables 7-8 (cluster stability and tightness values)
- Per-method cluster membership lists (result$mem) for reference and perturbed runs

## How to apply

Load the UMI count matrices for the human PBMC, mouse lung, and human skin droplet-based scRNA-seq datasets processed per the BAMMSC repository tutorials (10XGenomics.md, HumanPBMC.md, Mouselung.md, Humanskin.md), filter out genes with excessive zeros, and select the top 1000 highly variable genes per dataset. For ten repeated runs per dataset, randomly remove 100 of the top 1000 highly variable genes, rerun each method (BAMM-SC with dataset-appropriate K, nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10; plus SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MNN/scran and other competitors), and compute the average proportion of nonoverlap (APN) between the reference and perturbed clusterings from the returned mem memberships. Separately compute the average silhouette width of each method's cluster memberships on the full data to quantify cluster tightness. Assemble APN and silhouette values for all methods across the three datasets into stability_metrics.csv and validate that BAMM-SC attains low APN and high silhouette in most scenarios, consistent with Supplementary Tables 7-8.

## Related tools

- **BAMM-SC** (Primary Bayesian mixture model whose clustering memberships (mem) are benchmarked for APN and silhouette width) — https://github.com/lichen-lab/BAMMSC
- **SIMLR** (Competing scRNA-seq clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **CellTree** (Competing scRNA-seq clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **SC3** (Competing scRNA-seq clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **TSCAN** (Competing scRNA-seq clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **DIMM-SC** (Competing scRNA-seq clustering method (single-individual Dirichlet multinomial mixture) used in the benchmark)
- **scran** (Implementation of mutual nearest neighbors (MNN) batch correction used as a competing clustering pipeline)

## Examples

```
install_github("lichen-lab/BAMMSC"); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- APN computed between reference and perturbed clusterings should be low for BAMM-SC in most dataset scenarios, indicating high cluster stability
- Average silhouette width should be high for BAMM-SC, indicating tight clusters relative to competitors
- Values reproduce the pattern reported in Supplementary Tables 7-8 for the human PBMC, mouse lung, and human skin datasets
- Each method is run on identical perturbed inputs (same randomly removed 100 of the top 1000 highly variable genes) across ten repeated runs
- MCMC convergence is confirmed via the likTol.BAMMSC=1e-10 log-likelihood tolerance before memberships are compared

## Limitations

- Filtering genes with excessive zeros may be problematic for rare cell type identification
- BAMM-SC does not explicitly model zero-inflation, which may or may not affect clustering accuracy
- In the 10% cluster-label disagreement case BAMM-SC did not achieve the highest ARI in all scenarios, so 'most but not all' scenario wins must be reported honestly

## Evidence

- [intro] scRNA-seq tailored unsupervised methods, such as SIMLR9, CellTree10, SC311, TSCAN12, and DIMM-SC13: "scRNA-seq tailored unsupervised methods, such as SIMLR9, CellTree10, SC311, TSCAN12, and DIMM-SC13"
- [readme] mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual.: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual."
- [discussion] we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [readme] nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
