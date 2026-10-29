---
name: imputation-robustness-analysis-across-dropout-scenarios
description: Use when comparing scRNA-seq imputation methods for robustness across
  dropout rates, normalization techniques, or quantification scenarios — e.g. Splatter-simulated
  negative-binomial datasets where ARI against known cell-type labels must be aggregated
  per method and scenario.
when_to_use_negative:
- When only a single real scRNA-seq dataset needs imputing without a robustness benchmark
  — apply the imputation method directly instead of the scenario sweep
- When datasets lack known cell-type labels, since ARI against true labels is the
  sole evaluation metric
- When the data are already feature tables or bulk RNA-seq matrices rather than sparse
  single-cell count data with dropout zeros
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_2269
tools:
- name: scISR
  role: Imputation method under evaluation; identifies dropouts via hypothesis testing
    then imputes via subspace regression
  repo: https://github.com/duct317/scISR
- name: Splatter
  role: Simulator generating the negative-binomial scRNA-seq datasets with varied
    dropout rates and sample sizes
- name: scImpute
  role: Competitor imputation method benchmarked in the comparison
- name: SAVER
  role: Competitor imputation method benchmarked in the comparison
- name: MAGIC
  role: Competitor imputation method benchmarked in the comparison
- name: DrImpute
  role: Competitor imputation method benchmarked in the comparison
- name: scScope
  role: Competitor imputation method benchmarked in the comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:762277b743fe7afb8f27041730c0e13a221f4f0a7ba057252266e6a587122f8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - DrImpute
  - MAGIC
  - SAVER
  - Splatter
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# imputation-robustness-analysis-across-dropout-scenarios

## Summary

Benchmark imputation methods for scRNA-seq across increasing dropout/sparsity scenarios by clustering imputed data with k-means (using the true cell-type count k) and scoring Adjusted Rand Index against known labels, as done for scISR versus scImpute, SAVER, MAGIC, DrImpute, and scScope on 116 Splatter-simulated datasets.

## When to use

Use when comparing scRNA-seq imputation methods for robustness across dropout rates, normalization techniques, or quantification scenarios — e.g. Splatter-simulated negative-binomial datasets where ARI against known cell-type labels must be aggregated per method and scenario.

## When NOT to use

- When only a single real scRNA-seq dataset needs imputing without a robustness benchmark — apply the imputation method directly instead of the scenario sweep
- When datasets lack known cell-type labels, since ARI against true labels is the sole evaluation metric
- When the data are already feature tables or bulk RNA-seq matrices rather than sparse single-cell count data with dropout zeros

## Inputs

- Splatter-simulated scRNA-seq count matrices (negative-binomial simulation with varying dropout rates and sample sizes, e.g. 116 datasets)
- Ground-truth simulated cell-type labels per dataset
- True number of cell types k per dataset

## Outputs

- ARI value per imputation method per simulated dataset
- Aggregated ARI comparison table per method and scenario (e.g. simulation_ari_comparison.csv)

## How to apply

Generate simulated scRNA-seq datasets with Splatter while varying dropout rate (sparsity from 84% to 93%) and sample size (up to 200,000 cells), plus normalization technique and quantification scenario. Preprocess each dataset by removing genes expressed in fewer than two cells and genes with less than one percent non-zero entries, and rescale with log2 transformation any dataset whose value range exceeds 100. Apply the imputation methods — scISR (hypothesis-testing dropout identification followed by subspace regression imputation) and competitors — to each dataset, then cluster cells with k-means using the true number of cell types k as the number of clusters. Compute the Adjusted Rand Index (ARI) of each method's clustering against the known simulated labels, aggregate ARI values per method and scenario, and compare mean ARI stability across scenarios to judge robustness.

## Related tools

- **scISR** (Imputation method under evaluation; identifies dropouts via hypothesis testing then imputes via subspace regression) — https://github.com/duct317/scISR
- **Splatter** (Simulator generating the negative-binomial scRNA-seq datasets with varied dropout rates and sample sizes)
- **scImpute** (Competitor imputation method benchmarked in the comparison)
- **SAVER** (Competitor imputation method benchmarked in the comparison)
- **MAGIC** (Competitor imputation method benchmarked in the comparison)
- **DrImpute** (Competitor imputation method benchmarked in the comparison)
- **scScope** (Competitor imputation method benchmarked in the comparison)

## Examples

```
install.packages("scISR"); data(Goolam); raw <- Goolam$data; label <- Goolam$label; imputed <- scISR(data = raw)
```

## Evaluation signals

- scISR's mean ARI remains highest and stable across dropout-rate and normalization scenarios, matching the paper's robustness finding
- Filtering removed genes expressed in fewer than two cells and genes with less than one percent non-zero entries before imputation
- k-means was run with the true number of cell types k as the number of clusters for every dataset
- ARI values were computed for all 116 simulated datasets and aggregated per method and scenario into the comparison table
- Competitor methods that fail or underperform as sparsity increases (84%–93%) confirm the benchmark discriminates robustness

## Limitations

- Simulated dataset generation parameters are only partially specified (an illustrative example is given), limiting reproducibility of the 116 simulated datasets
- Competitor methods may fail or underperform on the most sparse scenarios, so per-method failures must be recorded rather than averaged over
- Some robustness evidence is deferred to supplementary figures (e.g. S19, S21, S24) not included in the main text

## Evidence

- [abstract] scISR method name and purpose: "we develop a novel imputation method, named single-cell Imputation via Subspace Regression (scISR), that can reliably recover the dropout values of scRNA-seq data"
- [results] log2 rescaling threshold: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] clustering protocol: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [discussion] dropout identification module: "This section describes the first module in scISR which aims at determining whether each zero value observed is the result of dropouts."
- [readme] hyper-geometric testing in package: "scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach."
