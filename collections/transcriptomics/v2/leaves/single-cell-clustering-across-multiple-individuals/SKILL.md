---
name: single-cell-clustering-across-multiple-individuals
description: Use when clustering raw UMI count matrices from droplet-based scRNA-seq
  (e.g., 10x Genomics Chromium) generated for several individuals or donors, and naive
  pooling would cause batch effects or Cell Ranger aggregation downsampling would
  lose information.
when_to_use_negative:
- The dataset consists of a single individual (use the DIMMSC mode explicitly instead
  of multi-individual BAMM-SC).
- The goal is quantifying cell cycle or developmental stages — each cell is assumed
  to belong to one distinct cluster and posterior probabilities only measure clustering
  uncertainty.
- The analysis requires identifying very rare cell types — filtering genes with excessive
  zeros and the supervised BAMMSC.predict strategy may miss rare clusters.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: BAMM-SC / BAMMSC R package
  role: Primary Bayesian mixture model for simultaneous multi-individual clustering
    of UMI counts
  repo: https://github.com/lichen-lab/BAMMSC
- name: DIMM-SC
  role: Single-individual Dirichlet multinomial clustering used when cells are pooled
    or only one individual is present
  repo: https://github.com/lichen-lab/BAMMSC
- name: mclust
  role: Provides adjustedRandIndex to evaluate clustering memberships against truth
    labels
- name: Cell Ranger
  role: Upstream 10x Genomics pipeline; aggregation downsampling is the information-loss
    problem BAMM-SC avoids
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:97d29ace48d269a34ec4fcdd4162b43d43ea13af697c07c45f8e2694743a323c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - BAMM-SC / BAMMSC R package
  - BAMMSC R package
  - Cell Ranger
  - DIMM-SC
  - SC3
  - TSCAN
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-clustering-across-multiple-individuals

## Summary

Cluster droplet-based scRNA-seq UMI count data from multiple individuals simultaneously using BAMM-SC, a Bayesian hierarchical Dirichlet multinomial mixture model that accounts for data heterogeneity and batch effects (e.g., unbalanced sequencing depths) in a unified framework.

## When to use

Use when clustering raw UMI count matrices from droplet-based scRNA-seq (e.g., 10x Genomics Chromium) generated for several individuals or donors, and naive pooling would cause batch effects or Cell Ranger aggregation downsampling would lose information.

## When NOT to use

- The dataset consists of a single individual (use the DIMMSC mode explicitly instead of multi-individual BAMM-SC).
- The goal is quantifying cell cycle or developmental stages — each cell is assumed to belong to one distinct cluster and posterior probabilities only measure clustering uncertainty.
- The analysis requires identifying very rare cell types — filtering genes with excessive zeros and the supervised BAMMSC.predict strategy may miss rare clusters.

## Inputs

- List of gene-by-cell UMI count matrices, one per individual (a single matrix triggers DIMMSC)
- Predefined number of clusters K (or AIC/BIC-based selection)
- Marker-gene-based approximated truth labels for ARI evaluation (optional)

## Outputs

- mem: list of per-individual cell clustering membership vectors
- loglik MCMC trace, AIC, and BIC for cluster-number selection
- alpha, pi, sigma2 parameter estimates and per-cell posterior cluster probabilities (delta)

## How to apply

Assemble the data as a list of gene-by-cell UMI count matrices (one element per individual; same number of genes, cell numbers may differ) and call BAMMSC(X, K) from the BAMMSC R package installed via install_github('lichen-lab/BAMMSC'). Keep default settings unless justified: K clusters, method_cluster_initial='kmeans', maxIter.BAMMSC=500, likTol.BAMMSC=1e-10, nBurn=400; the model characterizes variability at gene, cell-type, and individual levels. Predefine K from prior tissue knowledge or choose it using AIC/BIC returned by the model, and try a range of K values since K is treated as a continuum. For a single individual, or with option='DIMMSC' to pool cells, the package falls back to DIMMSC; for large datasets, optionally use BAMMSC.predict with a training proportion p (e.g., 0.5) to classify remaining cells by supervised classification. Filter out genes with excessive zeros before analysis, on the assumption that lowly-expressed genes contribute little to clustering.

## Related tools

- **BAMM-SC / BAMMSC R package** (Primary Bayesian mixture model for simultaneous multi-individual clustering of UMI counts) — https://github.com/lichen-lab/BAMMSC
- **DIMM-SC** (Single-individual Dirichlet multinomial clustering used when cells are pooled or only one individual is present) — https://github.com/lichen-lab/BAMMSC
- **mclust** (Provides adjustedRandIndex to evaluate clustering memberships against truth labels)
- **Cell Ranger** (Upstream 10x Genomics pipeline; aggregation downsampling is the information-loss problem BAMM-SC avoids)

## Examples

```
library(BAMMSC); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- Log-likelihood from MCMC converges within maxIter.BAMMSC=500 at likTol.BAMMSC=1e-10
- adjustedRandIndex(unlist(result$mem), unlist(truth)) computed with mclust is competitive; on the paper's real datasets BAMM-SC reached mean ARIs of ~0.487 (PBMC), 0.882 (mouse lung), 0.762 (skin)
- AIC and BIC returned by the run support the chosen K; a range of K values was tried rather than a single value
- Input X is a list whose elements share the same number of genes but may differ in cell number; a plain matrix implies DIMMSC was silently used
- Runtimes are consistent with expectations: about 1.5, 2.5, and 4.5 h for human PBMC, mouse lung, and human skin datasets

## Limitations

- Genes with excessive zeros are filtered out, which may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- Each cell is assumed to belong to one distinct cluster; posterior probabilities only measure clustering uncertainty, not cell cycle or developmental stage
- The supervised strategy for large datasets may potentially miss some rare clusters
- Further speed-up via parallel computing or GPU is noted as not yet implemented

## Evidence

- [abstract] BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model: "BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model"
- [methods] Install BAMMSC: "install_github("lichen-lab/BAMMSC")"
- [methods] Evaluate clustering with adjustedRandIndex: "adjustedRandIndex(unlist(Zpred),unlist(Z))"
- [readme] X argument description: "a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells"
- [discussion] Runtime discussion: "it takes about 1.5, 2.5, and 4.5 h when analyzing the 3 real datasets (human PBMC, mouse lung and human skin)"
- [discussion] Zero-filtering caveat: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
