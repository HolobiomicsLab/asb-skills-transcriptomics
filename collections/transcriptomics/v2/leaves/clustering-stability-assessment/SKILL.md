---
name: clustering-stability-assessment
description: Use when comparing unsupervised clustering methods on droplet-based scRNA-seq
  data and you need perturbation-based stability (APN) and tightness (silhouette width)
  metrics, as in BAMM-SC benchmarking on human PBMC, mouse lung, and human skin datasets.
when_to_use_negative:
- You have ground-truth cell labels — use clustering accuracy metrics such as adjustedRandIndex
  instead of perturbation-based APN/silhouette stability
- Your goal is rare cell type identification — the gene filtering step (removing genes
  with excessive zeros and dropping 100 HVGs) may be problematic for rare cell types
- Input is already a stability/tightness metrics table rather than raw UMI count matrices
  requiring clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BAMM-SC
  role: Primary clustering method whose memberships (result$mem) are rerun under gene-removal
    perturbations to compute APN and silhouette width
  repo: https://github.com/lichen-lab/BAMMSC
- name: SIMLR
  role: Competing scRNA-seq tailored unsupervised clustering method run on the same
    perturbed and full data for APN/silhouette comparison
- name: CellTree
  role: Competing scRNA-seq tailored unsupervised clustering method run on the same
    perturbed and full data for APN/silhouette comparison
- name: SC3
  role: Competing scRNA-seq tailored unsupervised clustering method run on the same
    perturbed and full data for APN/silhouette comparison
- name: TSCAN
  role: Competing scRNA-seq tailored unsupervised clustering method run on the same
    perturbed and full data for APN/silhouette comparison
- name: DIMM-SC
  role: Competing method for single-individual clustering, run identically for APN/silhouette
    comparison
  repo: https://github.com/lichen-lab/BAMMSC
- name: scran
  role: Implementation of mutual nearest neighbors (MNN) batch correction, one of
    the nine competitors evaluated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:c04ba97dcd2cdf31fe1ff7c11eef6d175737376f68439aa30429e90de3bf0e63
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# clustering-stability-assessment

## Summary

Quantifies cluster stability via average proportion of nonoverlap (APN) and cluster tightness via silhouette width by rerunning a clustering method (e.g., BAMM-SC) on perturbed gene sets and comparing memberships to a reference run. This reproduces the benchmarking logic of Supplementary Tables 7-8 comparing BAMM-SC against nine competing methods.

## When to use

Use when comparing unsupervised clustering methods on droplet-based scRNA-seq data and you need perturbation-based stability (APN) and tightness (silhouette width) metrics, as in BAMM-SC benchmarking on human PBMC, mouse lung, and human skin datasets.

## When NOT to use

- You have ground-truth cell labels — use clustering accuracy metrics such as adjustedRandIndex instead of perturbation-based APN/silhouette stability
- Your goal is rare cell type identification — the gene filtering step (removing genes with excessive zeros and dropping 100 HVGs) may be problematic for rare cell types
- Input is already a stability/tightness metrics table rather than raw UMI count matrices requiring clustering

## Inputs

- UMI count matrices (one per individual, genes x cells) for human PBMC, mouse lung, and human skin droplet-based scRNA-seq datasets
- Top 1000 highly variable genes per dataset (after filtering genes with excessive zeros)
- Reference clustering memberships (e.g., BAMMSC result$mem) and perturbed memberships from reruns with 100 randomly removed genes

## Outputs

- Average proportion of nonoverlap (APN) values per method per dataset over ten repeated gene-removal runs
- Average silhouette width values per method per dataset
- stability_metrics.csv assembling APN and silhouette values for BAMM-SC and all nine competitors across the three datasets

## How to apply

Load the UMI count matrices for each dataset (list of per-individual matrices, genes in rows and cells in columns) processed per the BAMMSC repository tutorials (10XGenomics.md, HumanPBMC.md, Mouselung.md, Humanskin.md). Rank genes by variability and select the top 1000 highly variable genes after filtering out genes with excessive zeros, under the assumption that lowly-expressed genes contribute little to clustering. For ten repeated runs per dataset, randomly remove 100 of the top 1000 genes, rerun the clustering (BAMMSC with dataset-appropriate K, nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10), and compute APN between reference and perturbed cluster memberships returned in the $mem output. Compute average silhouette width of the memberships to quantify tightness, run the nine competitors (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MNN/scran, etc.) identically, and assemble all values into stability_metrics.csv. Validation: BAMM-SC should attain low APN (high stability) and high silhouette width in most dataset scenarios, consistent with Supplementary Tables 7-8.

## Related tools

- **BAMM-SC** (Primary clustering method whose memberships (result$mem) are rerun under gene-removal perturbations to compute APN and silhouette width) — https://github.com/lichen-lab/BAMMSC
- **SIMLR** (Competing scRNA-seq tailored unsupervised clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **CellTree** (Competing scRNA-seq tailored unsupervised clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **SC3** (Competing scRNA-seq tailored unsupervised clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **TSCAN** (Competing scRNA-seq tailored unsupervised clustering method run on the same perturbed and full data for APN/silhouette comparison)
- **DIMM-SC** (Competing method for single-individual clustering, run identically for APN/silhouette comparison) — https://github.com/lichen-lab/BAMMSC
- **scran** (Implementation of mutual nearest neighbors (MNN) batch correction, one of the nine competitors evaluated)

## Examples

```
library(BAMMSC); result <- BAMMSC(X = data_list, K = 4, nBurn = 400, maxIter.BAMMSC = 500, likTol.BAMMSC = 1e-10); apn <- mean(unlist(result$mem) != unlist(reference$mem))
```

## Evaluation signals

- BAMM-SC attains low APN (high cluster stability) across the ten repeated 100-gene-removal runs in most dataset scenarios
- BAMM-SC attains high average silhouette width (tight clusters) in most of the three real datasets, matching Supplementary Tables 7-8
- The same gene set (top 1000 HVGs), K, and MCMC settings (nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10) are used for reference and perturbed runs
- All ten competitors (BAMM-SC + nine methods) are evaluated on identical perturbed and full data with identically computed APN and silhouette values
- stability_metrics.csv contains rows for each method x dataset combination with both APN and silhouette columns populated

## Limitations

- Filtering out genes with excessive zeros may be problematic for rare cell type identification
- BAMM-SC does not explicitly model zero-inflation, which may or may not affect clustering accuracy and hence stability estimates
- The number of clusters is treated as a continuum; the authors recommend trying a range of K values rather than a single value when defining the reference clustering
- MCMC-based reruns are computationally expensive (BAMM-SC takes about 1.5, 2.5, and 4.5 h on the three real datasets), and parallel/GPU speed-up is not yet implemented

## Evidence

- [intro] benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency: "benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency"
- [discussion] Gene filtering rationale: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [readme] mem output used for APN computation: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual. Length of list is #individuals"
- [readme] BAMMSC MCMC settings: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
- [abstract] Silhouette relevance to BAMM-SC clustering accuracy claim: "we develop a Bayesian mixture model for single-cell sequencing (BAMM-SC) method to cluster scRNA-seq data from multiple individuals simultaneously"
