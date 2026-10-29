---
name: baseline-method-benchmarking-in-scrnaseq
description: Use when clustering droplet-based multi-individual scRNA-seq UMI count
  data and you need to quantify whether a proposed method outperforms established
  baselines via adjusted Rand index against approximated truth labels.
when_to_use_negative:
- Data comes from a single individual — DIMM-SC (callable via BAMMSC on a matrix input)
  is the appropriate model rather than the multi-individual benchmark against MNN/CCA
  alignment pipelines
- The goal is rare cell-type discovery rather than benchmarking clustering accuracy
  — the article notes filtering genes with excessive zeros 'may be problematic for
  rare cell type identification'
- Truth labels cannot be approximated (no marker genes known) — ARI comparison against
  baselines is not well defined without a reference clustering
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BAMM-SC
  role: proposed Bayesian hierarchical Dirichlet multinomial mixture method being
    benchmarked; clusters multiple individuals simultaneously from raw UMI counts
  repo: https://github.com/lichen-lab/BAMMSC
- name: DIMM-SC
  role: single-individual Dirichlet multinomial baseline run on pooled cells across
    individuals
  repo: https://github.com/lichen-lab/BAMMSC
- name: SC3
  role: unsupervised scRNA-seq clustering baseline included in the ten-method comparison
- name: TSCAN
  role: scRNA-seq clustering baseline included in the ten-method comparison
- name: scran
  role: provides mutual nearest neighbors (MNN) batch correction used with K-means
    as a baseline
- name: mclust
  role: provides adjustedRandIndex used to score every clustering run against approximated
    truth labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:cd79a079d623cd90f682b5dce88253f1196b2c41e352983cbf37e6b2045b4426
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - BAMMSC R package
  - DIMM-SC
  - SC3
  - TSCAN
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# baseline-method-benchmarking-in-scrnaseq

## Summary

Benchmark BAMM-SC against nine scRNA-seq clustering baselines (MNN/CCA + K-means, TSCAN, SC3, Seurat, DIMM-SC) on multi-individual droplet UMI count datasets, scoring each run with mclust::adjustedRandIndex against marker-gene-based approximated truth labels and reporting mean/SD/range ARI per method per dataset.

## When to use

Use when clustering droplet-based multi-individual scRNA-seq UMI count data and you need to quantify whether a proposed method outperforms established baselines via adjusted Rand index against approximated truth labels.

## When NOT to use

- Data comes from a single individual — DIMM-SC (callable via BAMMSC on a matrix input) is the appropriate model rather than the multi-individual benchmark against MNN/CCA alignment pipelines
- The goal is rare cell-type discovery rather than benchmarking clustering accuracy — the article notes filtering genes with excessive zeros 'may be problematic for rare cell type identification'
- Truth labels cannot be approximated (no marker genes known) — ARI comparison against baselines is not well defined without a reference clustering

## Inputs

- list of gene-by-cell UMI count matrices, one per individual (same genes, variable cell counts), e.g., human PBMC from 5 donors, mouse lung from 4 mice, human skin from 5 donors
- marker-gene-based approximated truth cell-type labels for each individual
- number of clusters K per dataset (from prior knowledge or AIC/BIC)

## Outputs

- per-run adjusted Rand index values computed on the union of cell memberships across individuals (mclust::adjustedRandIndex)
- summary table of mean, standard deviation, and range ARI per method per dataset (e.g., ari_benchmark_results.csv)
- best-run t-SNE clusterings with ARI values for visual comparison

## How to apply

Load each dataset as a list of gene-by-cell UMI count matrices (e.g., human PBMC from 5 donors, mouse lung from 4 mice, human skin from 5 donors). Run BAMM-SC (install_github('lichen-lab/BAMMSC')) with default Bayesian hierarchical Dirichlet multinomial settings (maxIter.BAMMSC=500, likTol.BAMMSC=1e-10, nBurn=400) and each of the nine baselines ten times per dataset, computing mclust::adjustedRandIndex on the union of cell memberships across individuals against marker-gene-derived truth labels each run. Aggregate the ten replicate ARIs per method per dataset into mean, standard deviation, and range, then write the summary table (e.g., ari_benchmark_results.csv). Correct application is verified when BAMM-SC mean ARIs land near the reported benchmarks (0.487 PBMC, 0.882 mouse lung, 0.762 skin) and exceed all baselines on every dataset.

## Related tools

- **BAMM-SC** (proposed Bayesian hierarchical Dirichlet multinomial mixture method being benchmarked; clusters multiple individuals simultaneously from raw UMI counts) — https://github.com/lichen-lab/BAMMSC
- **DIMM-SC** (single-individual Dirichlet multinomial baseline run on pooled cells across individuals) — https://github.com/lichen-lab/BAMMSC
- **SC3** (unsupervised scRNA-seq clustering baseline included in the ten-method comparison)
- **TSCAN** (scRNA-seq clustering baseline included in the ten-method comparison)
- **scran** (provides mutual nearest neighbors (MNN) batch correction used with K-means as a baseline)
- **mclust** (provides adjustedRandIndex used to score every clustering run against approximated truth labels)

## Examples

```
library(BAMMSC); library(mclust); result=BAMMSC(data_BAMMSC,K=4); adjustedRandIndex(unlist(result$mem),unlist(Z))
```

## Evaluation signals

- BAMM-SC mean ARI over ten replicates matches the reported Table 1 values: ~0.487 (human PBMC), ~0.882 (mouse lung), ~0.762 (human skin)
- BAMM-SC achieves the highest mean ARI among the ten methods on all three datasets
- Best-run t-SNE ARIs are approximately 0.532, 0.910, and 0.843 for PBMC, mouse lung, and skin respectively
- Every method-dataset cell in the summary table contains mean, standard deviation, and range computed from exactly ten replicate runs
- BAMM-SC runtimes fall near the reported ~1.5, 2.5, and 4.5 h for PBMC, mouse lung, and skin, indicating default settings (maxIter.BAMMSC=500, nBurn=400) were used

## Limitations

- Filtering genes with excessive zeros may be problematic for rare cell type identification
- BAMM-SC does not explicitly model zero-inflation, which may or may not affect clustering accuracy
- In the 10% cluster-label disagreement simulation scenario, BAMM-SC did not achieve the highest ARI in all scenarios (only most)
- Each cell is assumed to belong to one distinct cluster; posterior probabilities measure clustering uncertainty, not developmental stage
- The number of clusters is treated as a continuum, so a range of K values should be tried rather than a single value

## Evidence

- [abstract] Table 1 benchmark ARIs: "BAMM-SC outperformed existing clustering methods with considerable improved clustering accuracy, particularly in the presence of heterogeneity among individuals"
- [abstract] BAMM-SC input model: "BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model framework"
- [methods] ARI evaluation: "adjustedRandIndex(unlist(Zpred),unlist(Z))"
- [intro] benchmark scope: "benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency"
- [discussion] runtime expectations: "it takes about 1.5, 2.5, and 4.5 h when analyzing the 3 real datasets (human PBMC, mouse lung and human skin)"
- [readme] installation and calling convention: "install_github("lichen-lab/BAMMSC")"
- [readme] output memberships used for ARI: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual. Length of list is #individuals"
