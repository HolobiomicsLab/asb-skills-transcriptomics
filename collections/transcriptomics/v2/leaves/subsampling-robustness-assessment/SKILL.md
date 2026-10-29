---
name: subsampling-robustness-assessment
description: Use when validating that features engineered from scRNA-seq data (e.g.,
  Spearman-correlation features from sscClust) are robust to subsampling fluctuations
  before trusting a subsampling-based clustering strategy.
when_to_use_negative:
- When the goal is a single full-data clustering run rather than a robustness assessment
  across subsampling rates.
- When no ground-truth cell-type labels are available, since silhouette values must
  be computed against true labels.
- When evaluating subsampling bias of methods that do not use a subsampling-clustering-classification
  (SCC) strategy, as the rationale is specific to SCC-style biases.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSCC
  role: Framework whose Spearman-correlation features are the object of the robustness
    assessment (subsampling-clustering-classification with random projection and feature
    construction).
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R package used to construct Spearman-correlation features and run the subsampling
    clustering pipeline on each subsample.
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation environment for sscClust and for computing silhouette values
    and Pearson correlation coefficients.
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:57347eeeb27bc2836f305f4a0ddfb6d29724f523fafe7423ec52ee063ef569d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# subsampling-robustness-assessment

## Summary

A validation procedure from the SSCC paper that tests whether Spearman correlation-based engineered features remain consistent with true cell-type labels across cell-subsampling rates (10%–50%), using per-cell silhouette values against true labels as the robustness metric.

## When to use

Use when validating that features engineered from scRNA-seq data (e.g., Spearman-correlation features from sscClust) are robust to subsampling fluctuations before trusting a subsampling-based clustering strategy.

## When NOT to use

- When the goal is a single full-data clustering run rather than a robustness assessment across subsampling rates.
- When no ground-truth cell-type labels are available, since silhouette values must be computed against true labels.
- When evaluating subsampling bias of methods that do not use a subsampling-clustering-classification (SCC) strategy, as the rationale is specific to SCC-style biases.

## Inputs

- scRNA-seq count matrices for benchmark datasets
- True cell-type labels for each cell in each dataset
- List of cell-subsampling rates (e.g., 10%, 20%, 30%, 40%, 50%)

## Outputs

- Per-cell silhouette values computed against true labels for each subsample
- Scatter plots of silhouette value (X) versus subsampling rate (Y), one per dataset
- Pearson correlation coefficient between silhouette values and subsampling rate per dataset

## How to apply

Load benchmark scRNA-seq count matrices together with their true cell-type labels, then generate subsampled datasets at five cell-subsampling rates (10%, 20%, 30%, 40%, 50%). For each subsample, construct Spearman-correlation features with the sscClust R package (recommended with an optimized BLAS library such as ATLAS or MKL for large data). Cluster each subsample and compute per-cell silhouette values against the true labels. Plot silhouette value versus subsampling rate as scatter plots, one per dataset, and compute the Pearson correlation coefficient for each. Accept the result when the Pearson correlation coefficients between silhouette values and subsampling rates are near zero, indicating silhouette quality is nearly independent of subsampling rate.

## Related tools

- **SSCC** (Framework whose Spearman-correlation features are the object of the robustness assessment (subsampling-clustering-classification with random projection and feature construction).) — https://github.com/Japrin/sscClust
- **sscClust** (R package used to construct Spearman-correlation features and run the subsampling clustering pipeline on each subsample.) — https://github.com/Japrin/sscClust
- **R** (Implementation environment for sscClust and for computing silhouette values and Pearson correlation coefficients.) — https://github.com/Japrin/sscClust

## Examples

```
sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Pearson correlation coefficient between per-cell silhouette values and subsampling rate is near zero for each benchmark dataset.
- Per-cell silhouette values of Spearman-correlation features are almost unchanged across the 10%–50% subsampling rates for all five benchmark datasets.
- Scatter plots of silhouette value versus subsampling rate show no systematic trend for every dataset.
- Each subsample's constructed features remain consistent with the true cell-type labels across all tested fractions.

## Limitations

- The SCC strategy can suffer from biases introduced by subsampling which may greatly decrease clustering accuracy and robustness, even though it reduces computational complexity from O(n^2) to O(n).
- Computation becomes expensive as datasets grow; sscClust strongly recommends R linked to an optimized BLAS library (e.g., ATLAS, MKL) and Microsoft R open for Windows users.
- Requires datasets with trusted true cell-type labels to compute silhouette values against ground truth.

## Evidence

- [abstract] SSCC framework: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [discussion] SCC subsampling bias rationale: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [readme] sscClust subsampling functionality: "clustering with subsampling and classfification which make it feasible to process thousands cells' data."
- [readme] subsample parameter in ssc.run: [evidence span withheld — anchor status: external_doc]
