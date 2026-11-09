---
name: average-proportion-of-nonoverlap-computation
description: Use when comparing clustering methods on droplet-based scRNA-seq data
  and you need a quantitative cluster-stability metric from repeated perturbation
  runs.
when_to_use_negative:
- When the analysis goal is cluster tightness rather than stability — use average
  silhouette width instead
- When no perturbation scheme (e.g., random gene removal) can be defined because the
  data cannot be reclustered repeatedly
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BAMM-SC
  role: Primary clustering method whose memberships (mem output) are compared via
    APN between reference and perturbed runs
  repo: https://github.com/lichen-lab/BAMMSC
- name: SIMLR
  role: Competing clustering method run on the same perturbed and full data for APN
    comparison
- name: CellTree
  role: Competing clustering method run on the same perturbed and full data for APN
    comparison
- name: SC3
  role: Competing clustering method run on the same perturbed and full data for APN
    comparison
- name: TSCAN
  role: Competing clustering method run on the same perturbed and full data for APN
    comparison
- name: DIMM-SC
  role: Competing clustering method run on the same perturbed and full data for APN
    comparison
- name: scran
  role: Implements mutual nearest neighbors (MNN), one of the nine competing methods
    benchmarked for APN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:f64b11e3c60a6af93f3bf6b3bb8cb2b8d8d7b6afc763b7915c4f76a225161be8
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# average-proportion-of-nonoverlap-computation

## Summary

Computes the average proportion of nonoverlap (APN) to quantify cluster stability by comparing a reference clustering against clusterings obtained after perturbing the data (e.g., removing 100 of the top 1000 highly variable genes), as used to benchmark BAMM-SC against nine competing methods on three real droplet-based scRNA-seq datasets.

## When to use

Use when comparing clustering methods on droplet-based scRNA-seq data and you need a quantitative cluster-stability metric from repeated perturbation runs.

## When NOT to use

- When the analysis goal is cluster tightness rather than stability — use average silhouette width instead
- When no perturbation scheme (e.g., random gene removal) can be defined because the data cannot be reclustered repeatedly

## Inputs

- UMI count matrices for each individual (list of genes x cells matrices) for the human PBMC, mouse lung, and human skin droplet-based scRNA-seq datasets
- Reference clustering memberships from the full data (top 1000 highly variable genes)
- Perturbed cluster memberships from repeated runs with 100 highly variable genes randomly removed

## Outputs

- Per-run APN values comparing reference vs. perturbed clusterings
- Aggregated APN values per clustering method per dataset (assembled into stability_metrics.csv alongside silhouette widths)

## How to apply

First obtain a reference clustering of cells, e.g., by running BAMM-SC on the full data with the top 1000 highly variable genes (selected after filtering out genes with excessive zeros) and dataset-appropriate K. Then, for ten repeated runs per dataset, randomly remove 100 of the top 1000 highly variable genes, rerun clustering on the perturbed data, and collect the cluster memberships from the returned mem objects. Compute the average proportion of nonoverlap (APN) between the reference and each perturbed clustering, then average over the ten runs. Low APN indicates high stability; compare BAMM-SC's APN against competitors such as SIMLR, CellTree, SC3, TSCAN, and DIMM-SC run identically, and pair with average silhouette width to assess cluster tightness.

## Related tools

- **BAMM-SC** (Primary clustering method whose memberships (mem output) are compared via APN between reference and perturbed runs) — https://github.com/lichen-lab/BAMMSC
- **SIMLR** (Competing clustering method run on the same perturbed and full data for APN comparison)
- **CellTree** (Competing clustering method run on the same perturbed and full data for APN comparison)
- **SC3** (Competing clustering method run on the same perturbed and full data for APN comparison)
- **TSCAN** (Competing clustering method run on the same perturbed and full data for APN comparison)
- **DIMM-SC** (Competing clustering method run on the same perturbed and full data for APN comparison)
- **scran** (Implements mutual nearest neighbors (MNN), one of the nine competing methods benchmarked for APN)

## Examples

```
result=BAMMSC(data_BAMMSC,K=4); adjustedRandIndex(unlist(result$mem),unlist(Z))
```

## Evaluation signals

- APN is low (high stability) for BAMM-SC in most dataset scenarios, consistent with Supplementary Tables 7-8
- Ten repeated perturbation runs per dataset each remove exactly 100 of the top 1000 highly variable genes
- APN computed from mem memberships matches the reference-vs-perturbed clustering comparison identically across all methods
- BAMM-SC's APN outperforms (is lower than) all nine competing clustering methods on the three real datasets
- Log-likelihood convergence criteria used in reruns match defaults: nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10

## Limitations

- Filtering out genes with excessive zeros prior to gene selection may be problematic for rare cell type identification
- APN alone does not measure cluster tightness; must be paired with average silhouette width for a full stability/tightness assessment
- 10% cluster-label disagreement case where BAMM-SC did not achieve the highest ARI in some scenarios — stability metrics may not always favor BAMM-SC in every scenario

## Evidence

- [supplementary] How do BAMM-SC and the nine competing clustering methods compare on cluster stability (average proportion of nonoverlap, APN): [evidence span withheld — anchor status: not_found]
- [supplementary] Perturbation scheme: [evidence span withheld — anchor status: not_found]
- [supplementary] MCMC settings: [evidence span withheld — anchor status: not_found]
- [supplementary] APN computed from mem memberships: [evidence span withheld — anchor status: not_found]
- [discussion] Gene filtering step: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [discussion] Runtime confirmation: "it takes about 1.5, 2.5, and 4.5 h when analyzing the 3 real datasets (human PBMC, mouse lung and human skin)"
- [readme] README install instructions: "install_github("lichen-lab/BAMMSC")"
- [readme] BAMMSC function output: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual"
