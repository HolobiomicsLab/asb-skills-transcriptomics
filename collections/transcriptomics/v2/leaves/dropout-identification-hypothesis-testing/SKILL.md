---
name: dropout-identification-hypothesis-testing
description: Use when a single-cell expression matrix (e.g., CPM/TPM/FPKM values)
  contains zero entries that must be classified as dropout artifacts or true zeros
  before imputation or downstream clustering.
when_to_use_negative:
- The expression matrix has no zero entries to classify (e.g., data is already dense
  or previously imputed) — there are no dropouts for the hyper-geometric tests to
  flag
- The goal is direct clustering or analysis of raw scRNA-seq data without imputation
  — the testing step is only a precursor inside the scISR pipeline
- The analysis requires imputing every zero entry indiscriminately — scISR deliberately
  leaves under-representation-significant (trustworthy) zeros untouched, so it is
  the wrong tool when blanket imputation is desired
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Implements the hyper-geometric testing that segregates zero entries into training
    and imputable data, then imputes flagged dropouts via subspace regression; run
    as scISR(data = raw) in R
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the lm function used to construct the generalized linear regression
    model fitted on training data and applied to imputable entries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:3f1d041fdc41d53020e6a4213520c3085bc53a8821ec785ece5beb9955824115
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# dropout-identification-hypothesis-testing

## Summary

Identifies which zero-valued entries in a scRNA-seq expression matrix are true dropouts versus trustworthy biological zeros using hyper-geometric over- and under-representation tests with a 0.01 significance threshold, so that only dropout-affected zeros are passed to subspace-regression imputation. This selective targeting avoids the false signals introduced by methods that impute all zero entries.

## When to use

Use when a single-cell expression matrix (e.g., CPM/TPM/FPKM values) contains zero entries that must be classified as dropout artifacts or true zeros before imputation or downstream clustering.

## When NOT to use

- The expression matrix has no zero entries to classify (e.g., data is already dense or previously imputed) — there are no dropouts for the hyper-geometric tests to flag
- The goal is direct clustering or analysis of raw scRNA-seq data without imputation — the testing step is only a precursor inside the scISR pipeline
- The analysis requires imputing every zero entry indiscriminately — scISR deliberately leaves under-representation-significant (trustworthy) zeros untouched, so it is the wrong tool when blanket imputation is desired

## Inputs

- scRNA-seq expression matrix (e.g., Goolam E-MTAB-3321 CPM matrix: 124 cells, genes x cells)
- Optional ground-truth cell-type labels for ARI-based validation
- Genes pre-filtered: genes expressed in fewer than two cells and genes with less than one percent non-zero entries removed

## Outputs

- Per-zero-entry classification: imputable (over-representation p < 0.01) vs trustworthy (under-representation p < 0.01)
- Segregated matrices: training data and imputable data
- Imputed expression matrix with only flagged dropout zeros replaced via subspace regression on the 10 genes with highest Pearson correlation
- ARI values for raw vs imputed clustering (ari_results.csv)

## How to apply

Run the hypothesis-testing module of scISR on the (optionally log2-transformed, if the value range exceeds 100) expression matrix: for each zero value, perform an over-representation test and an under-representation test with significance threshold 0.01. A zero whose over-representation p-value is significant is flagged as 'imputable' (untrustworthy, likely a dropout); a zero whose under-representation p-value is significant is deemed 'trustworthy' and left unchanged. The matrix is thereby segregated into training data (non-dropout entries) and imputable data (dropout entries), and the training data is used to build the generalized linear regression model (via lm in the R stats package) that imputes only the flagged entries. After imputation, evaluate by PCA (50 components via irlba::prcomp_irlba), k-means clustering with k set to the true number of cell types (nstart = 2000, iter.max = 2000, set.seed(1)), and adjustedRandIndex (mclust) against known labels.

## Related tools

- **scISR** (Implements the hyper-geometric testing that segregates zero entries into training and imputable data, then imputes flagged dropouts via subspace regression; run as scISR(data = raw) in R) — https://github.com/duct317/scISR
- **R stats** (Provides the lm function used to construct the generalized linear regression model fitted on training data and applied to imputable entries)

## Examples

```
library(scISR); data('Goolam'); imputed <- scISR(data = Goolam$data)
```

## Evaluation signals

- On the Goolam dataset (124 cells, E-MTAB-3321), the ARI of k-means clusters on the scISR-imputed data should be approximately 0.641 versus approximately 0.533 for raw data, exceeding all compared methods (MAGIC 0.512, SAVER 0.590, scGNN 0.525, scImpute 0.291, scScope 0.1)
- Only zero-valued entries with a significant over-representation p-value (threshold 0.01) should be imputed; zeros with significant under-representation p-values must remain unchanged as trustworthy
- The clustering pipeline must be reproducible: set.seed(1), PCA with 50 components via irlba::prcomp_irlba, and k-means with nstart = 2000 and iter.max = 2000
- Validation criterion: the scISR-imputed ARI must exceed the raw-data ARI, written to ari_results.csv
- The imputed matrix must retain dimensions of the input (only zero entries replaced), confirming that non-dropout and non-zero values were not perturbed

## Limitations

- No version stated for the R stats package or the scISR software, limiting reproducibility
- The repository has no changelog ('_No changelog found._'), limiting reproducibility of the repository content
- Simulated-dataset generation parameters are only partially specified (illustrative example only), limiting reproducibility of the 116 simulated datasets used in the broader evaluation
- Key robustness evidence is deferred to supplementary figures (e.g., S19, S21, S24) not included in the main text

## Evidence

- [abstract] The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events: "The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events"
- [discussion] An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable).: "An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable)."
- [discussion] An entry with a significant p-value in the under-representation analysis is considered trustworthy.: "An entry with a significant p-value in the under-representation analysis is considered trustworthy."
- [readme] scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach.: "scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach. Based on the result obtained from hyper-geometric testing, the original dataset is segregated"
- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
