---
name: subspace-regression-gene-imputation
description: Use when a raw scRNA-seq expression matrix (e.g., CPM/TPM/FPKM values)
  contains zero entries suspected of being dropout events, and the goal is to improve
  downstream cell-type clustering (k-means ARI) before imputation-vulnerable analyses.
when_to_use_negative:
- The input contains no zero-valued entries or zeros are known to be true biological
  zeros — scISR's testing module is designed to distinguish dropouts, but imputing
  genuine zeros risks introducing false signals
- The analysis goal requires imputing all zero entries uniformly (e.g., some model-based
  baselines) — scISR intentionally alters only entries flagged as dropout-affected
  by hyper-geometric testing
- Downstream analysis is robust to dropouts or uses methods that internally handle
  sparsity (e.g., certain count-model tools), where over-smoothing by imputation could
  remove meaningful cell-to-cell stochasticity
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_0203
tools:
- name: scISR
  role: Core R package performing hyper-geometric dropout detection and subspace-regression
    imputation; installed from CRAN or GitHub
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Supplies the lm function used to fit the generalized linear regression model
    inside scISR's imputation module
- name: irlba
  role: Provides prcomp_irlba for PCA with 50 components on raw and imputed matrices
    prior to clustering
- name: mclust
  role: Supplies adjustedRandIndex to compute ARI of k-means clusters against known
    cell labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:d346f66287aa9115a4ec488482b2de64c4d80c8d6e1daeb351401f5a75560f8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - irlba
  - mclust
  - scISR
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# subspace-regression-gene-imputation

## Summary

Applies scISR (single-cell Imputation via Subspace Regression) to recover dropout-induced zero values in scRNA-seq count matrices: hyper-geometric over/under-representation testing (p < 0.01) segregates zeros into imputable vs. trustworthy sets, and a generalized linear regression built on the 10 most Pearson-correlated genes imputes the imputable entries.

## When to use

Use when a raw scRNA-seq expression matrix (e.g., CPM/TPM/FPKM values) contains zero entries suspected of being dropout events, and the goal is to improve downstream cell-type clustering (k-means ARI) before imputation-vulnerable analyses.

## When NOT to use

- The input contains no zero-valued entries or zeros are known to be true biological zeros — scISR's testing module is designed to distinguish dropouts, but imputing genuine zeros risks introducing false signals
- The analysis goal requires imputing all zero entries uniformly (e.g., some model-based baselines) — scISR intentionally alters only entries flagged as dropout-affected by hyper-geometric testing
- Downstream analysis is robust to dropouts or uses methods that internally handle sparsity (e.g., certain count-model tools), where over-smoothing by imputation could remove meaningful cell-to-cell stochasticity

## Inputs

- scRNA-seq expression matrix (genes x cells) with numeric values, e.g. CPM/TPM/FPKM/RPKM counts (Goolam E-MTAB-3321 provides CPM values)
- Ground-truth cell-type label vector (one label per cell) for ARI-based validation
- Number of clusters k set to the true number of cell types

## Outputs

- Imputed expression matrix with only dropout-affected zero entries replaced (raw matrix segregated into training and imputable data)
- ARI values for k-means clusters derived from raw vs. imputed data (e.g., written to ari_results.csv), expected ~0.533 raw vs ~0.641 scISR on Goolam (124 cells)

## How to apply

First preprocess the expression matrix: if the value range exceeds 100, rescale with log2 transformation, and remove genes expressed in fewer than two cells or with less than one percent non-zero entries. Run scISR, which first performs hyper-geometric testing to identify zero-valued entries most likely affected by dropout events — an entry with a significant over-representation p-value (threshold 0.01) is deemed imputable, while a significant under-representation p-value marks it trustworthy — then groups genes with similar patterns via data perturbation and imputes dropout-affected values with a generalized linear (subspace) regression model that uses the 10 genes with highest Pearson correlation to each target gene. Unlike many competing methods, scISR alters only dropout-affected zeros rather than all zero-valued entries. Assess the gain by running PCA (e.g., 50 components via irlba::prcomp_irlba) on raw and imputed matrices, clustering with k-means using k equal to the true number of cell types (nstart = 2000, iter.max = 2000), and comparing adjusted Rand Index (adjustedRandIndex from mclust) against known labels.

## Related tools

- **scISR** (Core R package performing hyper-geometric dropout detection and subspace-regression imputation; installed from CRAN or GitHub) — https://github.com/duct317/scISR
- **R stats** (Supplies the lm function used to fit the generalized linear regression model inside scISR's imputation module)
- **irlba** (Provides prcomp_irlba for PCA with 50 components on raw and imputed matrices prior to clustering)
- **mclust** (Supplies adjustedRandIndex to compute ARI of k-means clusters against known cell labels)

## Examples

```
library(scISR); data('Goolam'); imputed <- scISR(data = Goolam$data); pca_imputed <- irlba::prcomp_irlba(t(imputed), n = 50)$x; cluster_imputed <- kmeans(pca_imputed, length(unique(Goolam$label)), nstart = 2000, iter.max = 2000)$cluster
```

## Evaluation signals

- scISR-imputed ARI should exceed raw-data ARI; on Goolam (124 cells, CPM) expect ~0.641 (scISR) vs ~0.533 (raw), with scISR highest among compared methods (MAGIC 0.512, scImpute 0.291, SAVER 0.590, scScope 0.1, scGNN 0.525)
- Preprocessing invariant: genes expressed in fewer than two cells or with <1% non-zero entries are removed before imputation; log2 rescaling applied when value range > 100
- Only entries flagged by significant over-representation tests (p < 0.01) are imputed; entries with significant under-representation tests remain untouched (trustworthy)
- Fixed-seed reproducibility check: set.seed(1) before prcomp_irlba(t(data), n = 50) and kmeans(k = unique cell types, nstart = 2000, iter.max = 2000) should yield stable ARI values
- Each target gene's regression subspace uses exactly the 10 genes with highest Pearson correlation

## Limitations

- No version stated for the R stats package or the scISR software, limiting exact reproducibility
- Key robustness evidence is deferred to supplementary figures/sections (S19, S21, S24) not always included
- Simulated dataset generation parameters (116 simulated datasets) are only partially specified, limiting reproducibility of simulation-based validation
- Existing methods (and by extension any imputation) risk over-smoothing meaningful cell-to-cell stochasticity; scISR mitigates but does not eliminate this by imputing only flagged zeros

## Evidence

- [abstract] hyper-geometric testing to identify dropout-affected zeros and subspace-regression imputation: "The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events"
- [abstract] imputation via subspace regression: "and then estimates the dropout values using a subspace regression model"
- [results] preprocessing filters for gene retention: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [results] log2 rescaling rule: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] k-means with true number of cell types: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [discussion] over-representation test threshold and imputability rule: "An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable)."
- [discussion] top-10 Pearson-correlated genes for regression: [evidence span withheld — anchor status: not_found]
- [readme] README: package installation and Goolam example invocation: [evidence span withheld — anchor status: not_found]
- [readme] README: PCA and k-means assessment snippet: "pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x"
