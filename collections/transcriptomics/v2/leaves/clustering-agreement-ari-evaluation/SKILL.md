---
name: clustering-agreement-ari-evaluation
description: Use when benchmarking a co-expression or clustering method on simulated
  data with known latent gene clusters (e.g., four co-expressed clusters, n=2000 cells,
  p=100 genes) and you need ARI of cluster recovery plus PCA subspace agreement per
  method.
when_to_use_negative:
- Applied to real scRNA-seq data where true latent cluster labels or a true co-expression
  matrix are unknown — ARI against ground truth is undefined without a simulation
  with known structure
- Evaluating differential co-expression between conditions (use diff_CSCORE donor-level
  permutation testing instead of cluster-agreement ARI)
- Comparing co-expression P-value calibration or type-I error under the null, which
  requires null-simulation designs rather than planted-cluster ARI metrics
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CSCORE (R package)
  role: Estimates the cell-type-specific co-expression network whose recovered clusters
    are scored by ARI and PCA subspace distance against the simulation ground truth
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: WGCNA
  role: Builds adjacency and topological overlap matrices from estimated co-expressions
    so that gene modules/clusters can be recovered for ARI comparison
- name: dynamicTreeCut
  role: Cuts the hierarchical clustering dendrogram (cutreeDynamic) to assign cluster
    labels whose agreement with truth is measured by ARI
- name: locCSN
  role: Comparison co-expression method whose network and cluster output are benchmarked
    against CS-CORE in the Fig. 4 evaluation
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Comparison co-expression method evaluated on the simulated four-cluster data
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:082417036c0e2b9f0fd75f200a5b5695118e660ee91ad7bc34f33d8ad8d7162b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE (R package)
  - WGCNA
  - dynamicTreeCut
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# clustering-agreement-ari-evaluation

## Summary

Evaluates whether estimated co-expression networks recover known cluster structure by scoring cluster-label agreement with the adjusted Rand index (ARI) and comparing estimated vs. true principal components via subspace distance, as done in the CS-CORE Fig. 4 simulation benchmark.

## When to use

Use when benchmarking a co-expression or clustering method on simulated data with known latent gene clusters (e.g., four co-expressed clusters, n=2000 cells, p=100 genes) and you need ARI of cluster recovery plus PCA subspace agreement per method.

## When NOT to use

- Applied to real scRNA-seq data where true latent cluster labels or a true co-expression matrix are unknown — ARI against ground truth is undefined without a simulation with known structure
- Evaluating differential co-expression between conditions (use diff_CSCORE donor-level permutation testing instead of cluster-agreement ARI)
- Comparing co-expression P-value calibration or type-I error under the null, which requires null-simulation designs rather than planted-cluster ARI metrics

## Inputs

- Simulated latent true expression matrix for n=2000 cells and p=100 genes with genes assigned to four correlated co-expression clusters
- Observed UMI count matrix generated from the latent expression via a Poisson measurement model with varying sequencing depths
- Estimated co-expression matrices from CS-CORE and comparison methods
- True gene cluster labels and true co-expression matrix for reference

## Outputs

- Per-method adjusted Rand index (ARI) of co-expression cluster recovery
- Subspace distance between top four singular vectors of estimated and true co-expression matrices
- Correlation of estimated vs. true mean expression and co-expression values
- ARI of expression-cluster recovery per method
- Results table matching Fig. 4a-e panels (e.g., figure4_simulation_benchmark.csv)

## How to apply

Simulate latent true expression in which genes form known co-expression clusters (e.g., four clusters, n=2000 cells, p=100 genes) and generate observed UMI counts via the Poisson measurement model with varying sequencing depths. Estimate networks with each method (CS-CORE via the CSCORE R package, plus comparison methods such as WGCNA, locCSN, NoiseRegularization, normalisr). Recover clusters from each estimated network (e.g., WGCNA adjacency/TOM followed by dynamicTreeCut::cutreeDynamic on the hierarchical clustering) and compute the adjusted Rand index against the true cluster labels. Additionally compute the subspace distance between the top four singular vectors of estimated and true co-expression matrices (matching Fig. 4's four-cluster design), plus correlations of estimated vs. true means and co-expression values. Assemble per-method metrics into a table (e.g., figure4_simulation_benchmark.csv) whose panels match Fig. 4a-e.

## Related tools

- **CSCORE (R package)** (Estimates the cell-type-specific co-expression network whose recovered clusters are scored by ARI and PCA subspace distance against the simulation ground truth) — https://github.com/ChangSuBiostats/CS-CORE
- **WGCNA** (Builds adjacency and topological overlap matrices from estimated co-expressions so that gene modules/clusters can be recovered for ARI comparison)
- **dynamicTreeCut** (Cuts the hierarchical clustering dendrogram (cutreeDynamic) to assign cluster labels whose agreement with truth is measured by ARI)
- **locCSN** (Comparison co-expression method whose network and cluster output are benchmarked against CS-CORE in the Fig. 4 evaluation) — https://github.com/xuranw/locCSN
- **normalisr** (Comparison co-expression method evaluated on the simulated four-cluster data) — https://github.com/lingfeiwang/normalisr

## Examples

```
CSCORE_result <- CSCORE(sim_counts, genes = genes_selected); memb = dynamicTreeCut::cutreeDynamic(dendro = hclust_dist, distM = as.dist(1 - abs(CSCORE_coexp))); mclust::adjustedRandIndex(memb, true_clusters)
```

## Evaluation signals

- CS-CORE should be the only method with an ARI near 1 for recovering the four truly co-expressed gene clusters, while other methods yield strikingly inaccurate cluster labels
- Subspace distance between the top four singular vectors of the CS-CORE estimated and true co-expression matrices should be near zero relative to comparison methods
- Correlation of CS-CORE estimated vs. true co-expression values should be markedly higher than for biased comparison methods
- Output CSV must contain one row per method with ARI, subspace distance, and correlation columns matching the Fig. 4a-e panel structure
- Cluster-label counts from cutreeDynamic should recover exactly four modules matching the planted cluster sizes when the method is unbiased

## Limitations

- The benchmark relies on the Poisson measurement model; CS-CORE's own authors note adapting to other count distributions such as the negative binomial is deferred to future work
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- ARI and PCA subspace metrics require simulated ground truth; they cannot be computed for real datasets without known latent cluster structure

## Evidence

- [results] In simulations with n=2000 cells, p=100 genes and four co-expressed gene clusters, CS-CORE was the only method that accurately estimated co-expressions and recovered the truly co-expressed gene clusters: [evidence span withheld — anchor status: not_found]
- [results] Compute evaluation metrics: ARI of co-expression cluster recovery, subspace distance between top four singular vectors, correlation of estimated vs true values: [evidence span withheld — anchor status: not_found]
- [abstract] Most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [methods] cutreeDynamic assigns cluster labels from the hierarchical clustering dendrogram for module recovery: "memb = dynamicTreeCut::cutreeDynamic(dendro = hclust_dist, "
- [readme] CS-CORE is an R package for cell-type-specific co-expression inference installed from GitHub: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
- [discussion] Measurement model discussion and negative binomial extension deferred to future work: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution and we leave it to future work."
