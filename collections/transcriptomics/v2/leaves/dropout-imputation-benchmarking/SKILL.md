---
name: dropout-imputation-benchmarking
description: 'Use when a scRNA-seq matrix with dropout zeros needs imputation quality
  assessment: compare imputed vs raw clustering (ARI), evaluate robustness across
  dropout rates, normalization (FPKM, CPM, TPM, RPM), or simulated sparsity (e.g.,
  Splatter negative-binomial, 84-93%).'
when_to_use_negative:
- The input is not single-cell expression data — e.g. bulk RNA-seq or already-clusted/feature-aggregated
  tables where cell-level dropout assessment is meaningless
- The zeros are known true biological zeros and imputing all zero entries would introduce
  false signals (the article notes both model-based and model-free methods risk altering
  true zeros)
- No ground-truth labels or simulation-derived labels exist to compute ARI, so imputation
  quality cannot be evaluated by clustering
edam_operation: http://edamontology.org/operation_4091
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scISR
  role: Primary imputation method under evaluation; hypothesis-testing dropout identification
    followed by subspace/generalized linear regression imputation
  repo: https://github.com/duct317/scISR
- name: scImpute
  role: Competitor imputation method included in the benchmark comparison
- name: SAVER
  role: Competitor imputation method included in the benchmark comparison
- name: MAGIC
  role: Competitor imputation method included in the benchmark comparison
- name: DrImpute
  role: Competitor imputation method included in the benchmark comparison
- name: scScope
  role: Competitor imputation method included in the benchmark comparison
- name: Splatter
  role: Simulator used to generate the 116 negative-binomial scRNA-seq datasets with
    varying dropout rates for robustness testing
- name: mclust
  role: Provides adjustedRandIndex used to score k-means clustering of imputed data
    against true labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:21e2f4793970df3c00b2a64d12e5a1f788b77a504f952aaa5fbb55a42a5712ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - DrImpute
  - MAGIC
  - SAVER
  - Splatter
  - mclust
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_4091
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# dropout-imputation-benchmarking

## Summary

Benchmarks single-cell RNA-seq dropout imputation methods by imputing raw count/F PKM-TPM-CPM matrices, clustering cells with k-means (true k), and scoring cluster quality via Adjusted Rand Index against known labels, as done to validate scISR against scImpute, SAVER, MAGIC, DrImpute, and scScope.

## When to use

Use when a scRNA-seq matrix with dropout zeros needs imputation quality assessment: compare imputed vs raw clustering (ARI), evaluate robustness across dropout rates, normalization (FPKM, CPM, TPM, RPM), or simulated sparsity (e.g., Splatter negative-binomial, 84-93%).

## When NOT to use

- The input is not single-cell expression data — e.g. bulk RNA-seq or already-clusted/feature-aggregated tables where cell-level dropout assessment is meaningless
- The zeros are known true biological zeros and imputing all zero entries would introduce false signals (the article notes both model-based and model-free methods risk altering true zeros)
- No ground-truth labels or simulation-derived labels exist to compute ARI, so imputation quality cannot be evaluated by clustering

## Inputs

- Raw scRNA-seq expression matrix (counts or FPKM/CPM/TPM/RPM), e.g. the Goolam dataset bundled with the scISR package
- Ground-truth cell-type labels for computing Adjusted Rand Index
- Optionally, Splatter-simulated negative-binomial datasets spanning dropout rates and sample sizes (116 simulated scenarios in the paper)

## Outputs

- Imputed expression matrices per method (scISR, scImpute, SAVER, MAGIC, DrImpute, scScope)
- ARI values per method per dataset/scenario, aggregated into a comparison table (e.g. simulation_ari_comparison.csv)
- ARI for raw (unimputed) data as baseline

## How to apply

Preprocess each dataset by removing genes expressed in fewer than two cells and genes with less than one percent non-zero entries; for datasets with a value range larger than 100, rescale using a base-2 log transformation. Run each imputation method — scISR first uses hyper-geometric over/under-representation testing (significance threshold 0.01) to split zeros into imputable (untrustworthy) vs trustworthy entries, then a generalized linear regression built on the 10 genes with highest Pearson correlation imputes the imputable data. For each method, cluster cells with k-means using the true number of cell types k as the number of clusters (README variant: PCA to 50 components via prcomp_irlba, kmeans with nstart=2000, iter.max=2000). Compute the Adjusted Rand Index of clusters vs known labels (mclust::adjustedRandIndex) and aggregate per method/scenario into a comparison table. For simulation-based robustness, generate datasets with Splatter varying dropout rate and sample size, and verify that scISR's mean ARI remains highest and stable across scenarios while competitors underperform or fail.

## Related tools

- **scISR** (Primary imputation method under evaluation; hypothesis-testing dropout identification followed by subspace/generalized linear regression imputation) — https://github.com/duct317/scISR
- **scImpute** (Competitor imputation method included in the benchmark comparison)
- **SAVER** (Competitor imputation method included in the benchmark comparison)
- **MAGIC** (Competitor imputation method included in the benchmark comparison)
- **DrImpute** (Competitor imputation method included in the benchmark comparison)
- **scScope** (Competitor imputation method included in the benchmark comparison)
- **Splatter** (Simulator used to generate the 116 negative-binomial scRNA-seq datasets with varying dropout rates for robustness testing)
- **mclust** (Provides adjustedRandIndex used to score k-means clustering of imputed data against true labels)

## Examples

```
data('Goolam'); raw <- Goolam$data; label <- Goolam$label; imputed <- scISR(data = raw)
```

## Evaluation signals

- ARI of k-means clusters from imputed data exceeds the ARI from raw data (README reports both, rounded to 3 decimals)
- scISR's mean ARI is highest and remains stable across dropout-rate, normalization, and quantification scenarios (FPKM, CPM, TPM, RPM) — the paper's reported robustness finding
- Preprocessing invariants hold: genes expressed in <2 cells and genes with <1% non-zero entries are removed; value range >100 triggers log2 rescaling
- scISR's two-tier split of zeros is respected: entries significant in the over-representation test are imputed (imputable), while entries significant in the under-representation test remain unchanged (trustworthy)
- Across all 116 Splatter-simulated datasets, scISR improves clustering quality of dropout data while competitor methods fail or underperform as sparsity increases from 84% to 93%

## Limitations

- Existing methods of both categories alter all zero-valued entries, including true zeros, potentially introducing false signals; scISR mitigates this by testing each zero, but the risk of over-smoothing meaningful biological variation remains for competitors
- Simulated dataset generation parameters are only partially specified (illustrative standard-normal example only), limiting exact reproducibility of the 116 simulated datasets
- Key robustness evidence (Supplementary Figures S19, S21, S24, Section 4.2) is deferred to supplementary material not included in the main text
- Model-based imputation depends on distribution-model validity and can require excessive computational power; model-free methods depend on many parameters and risk overfitting

## Evidence

- [readme] scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach: "scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach."
- [results] Filtering rule: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [results] Rescaling rule: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] Evaluation protocol: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [discussion] Dropout test threshold: "An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable)."
- [abstract] Robustness finding: "scISR consistently improves the quality of cluster analysis regardless of dropout rates, normalization techniques, and quantification schemes"
