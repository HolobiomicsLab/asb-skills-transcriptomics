---
name: silhouette-width-evaluation
description: Use when a droplet-based scRNA-seq clustering has produced per-cell cluster
  memberships (e.g., the mem output of BAMM-SC) and you must report cluster tightness
  as average silhouette width, and cluster stability as APN under gene-perturbation
  reruns, to compare against competing methods.
when_to_use_negative:
- When true cell labels are known and you only need clustering accuracy - the paper
  uses adjustedRandIndex (e.g., adjustedRandIndex(unlist(result$mem), unlist(Z)))
  for that, not silhouette width
- When the goal is identifying rare cell types in data where filtering genes with
  excessive zeros would discard the relevant signal - the paper notes this filtering
  'may be problematic for rare cell type identification'
- When input is not droplet-based UMI count data from multiple individuals - BAMM-SC
  is designed for 10x Chromium-style population data; single-individual data routes
  to DIMMSC instead
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BAMM-SC
  role: Produces the cluster memberships (mem) on full and 100-gene-perturbed data
    from which APN and average silhouette width are computed; run with K per dataset
    and nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10
  repo: https://github.com/lichen-lab/BAMMSC
- name: SIMLR
  role: Competing scRNA-seq clustering method run on the same full and perturbed data
    for identical APN and silhouette computation
- name: SC3
  role: Competing scRNA-seq clustering method benchmarked identically for APN and
    silhouette width
- name: TSCAN
  role: Competing scRNA-seq clustering method benchmarked identically for APN and
    silhouette width
- name: DIMM-SC
  role: Competing scRNA-seq clustering method (also integrated into BAMM-SC for single-individual
    analysis) benchmarked identically
- name: CellTree
  role: Competing scRNA-seq clustering method benchmarked identically for APN and
    silhouette width
- name: scran
  role: Implements the mutual nearest neighbors (MNN) competing clustering approach
    benchmarked identically
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:156b289bef28fff7823af7d999c8f7ffbcbb316041b2d362871ba90f865a13f1
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
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# silhouette-width-evaluation

## Summary

Quantifies cluster tightness by computing the average silhouette width of clustering memberships (e.g., BAMM-SC's returned mem lists) across the three real droplet-based scRNA-seq datasets (human PBMC, mouse lung, human skin), and pairs it with average proportion of nonoverlap (APN) to reproduce the stability/tightness benchmarking of Supplementary Tables 7-8.

## When to use

Use when a droplet-based scRNA-seq clustering has produced per-cell cluster memberships (e.g., the mem output of BAMM-SC) and you must report cluster tightness as average silhouette width, and cluster stability as APN under gene-perturbation reruns, to compare against competing methods.

## When NOT to use

- When true cell labels are known and you only need clustering accuracy - the paper uses adjustedRandIndex (e.g., adjustedRandIndex(unlist(result$mem), unlist(Z))) for that, not silhouette width
- When the goal is identifying rare cell types in data where filtering genes with excessive zeros would discard the relevant signal - the paper notes this filtering 'may be problematic for rare cell type identification'
- When input is not droplet-based UMI count data from multiple individuals - BAMM-SC is designed for 10x Chromium-style population data; single-individual data routes to DIMMSC instead

## Inputs

- Per-individual UMI count matrices (list of genes x cells matrices; same genes across individuals, cell counts may differ), from the three real datasets human PBMC, mouse lung, human skin
- BAMM-SC mem output: list of per-individual cluster membership vectors for all cells
- Top 1000 highly variable gene list per dataset (after filtering genes with excessive zeros)
- Cluster memberships from the nine competing methods (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MNN/scran, etc.) on full and perturbed data

## Outputs

- Average silhouette width per dataset per method (cluster tightness)
- APN values between reference and perturbed clusterings per run/dataset/method (cluster stability)
- Assembled stability_metrics.csv of APN and silhouette values for BAMM-SC and all competitors across the three datasets

## How to apply

For each dataset (human PBMC, mouse lung, human skin), load the per-individual UMI count matrices, rank genes by variability, and select the top 1000 highly variable genes after filtering out genes with excessive zeros. For each of ten repeated runs, randomly remove 100 of those 1000 genes, rerun BAMM-SC with the dataset-appropriate K and default MCMC settings (nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10), and compute the average proportion of nonoverlap (APN) between reference and perturbed clusterings taken from the returned mem memberships. Compute the average silhouette width of the full-data mem memberships for each dataset to quantify cluster tightness. Run the nine competing methods (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MNN/scran, etc.) on the same full and perturbed data and compute APN and silhouette identically. The rationale is that low APN indicates high stability and high silhouette width indicates tight clusters, which is the criterion set used in Supplementary Tables 7-8.

## Related tools

- **BAMM-SC** (Produces the cluster memberships (mem) on full and 100-gene-perturbed data from which APN and average silhouette width are computed; run with K per dataset and nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10) — https://github.com/lichen-lab/BAMMSC
- **SIMLR** (Competing scRNA-seq clustering method run on the same full and perturbed data for identical APN and silhouette computation)
- **SC3** (Competing scRNA-seq clustering method benchmarked identically for APN and silhouette width)
- **TSCAN** (Competing scRNA-seq clustering method benchmarked identically for APN and silhouette width)
- **DIMM-SC** (Competing scRNA-seq clustering method (also integrated into BAMM-SC for single-individual analysis) benchmarked identically)
- **CellTree** (Competing scRNA-seq clustering method benchmarked identically for APN and silhouette width)
- **scran** (Implements the mutual nearest neighbors (MNN) competing clustering approach benchmarked identically)

## Examples

```
result = BAMMSC(data_BAMMSC, K=4)  # APN/silhouette computed from result$mem memberships
```

## Evaluation signals

- BAMM-SC attains high cluster stability (low APN) and high cluster tightness (high average silhouette width) in most dataset scenarios, consistent with Supplementary Tables 7-8
- Each stability run removes exactly 100 randomly chosen genes from the top 1000 highly variable genes, repeated ten times per dataset
- MCMC settings match the documented defaults: nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10
- mem output schema holds: a list with one membership vector [#cell] per individual, so silhouette/APN can be computed on a unified cross-individual label vector
- Competing methods (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MNN/scran, etc.) are evaluated on the same full and perturbed gene sets so APN/silhouette comparisons are apples-to-apples

## Limitations

- Filtering genes with excessive zeros prior to analysis may be problematic for rare cell type identification
- BAMM-SC does not explicitly model a zero-inflation pattern, which may or may not affect clustering accuracy and hence the tightness/stability metrics
- Each cell is assumed to belong to one distinct cluster; posterior probabilities measure clustering uncertainty rather than cell cycle or developmental stage
- In some scenarios (e.g., the 10% cluster-label disagreement case) BAMM-SC did not achieve the highest ARI, so tightness/silhouette wins do not guarantee accuracy wins in every scenario

## Evidence

- [intro] benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency: "benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency"
- [readme] mem output schema: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual. Length of list is #individuals"
- [discussion] zero-gene filtering rationale: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [discussion] rare cell type caveat of the filter step: "This may be problematic for rare cell type identification"
