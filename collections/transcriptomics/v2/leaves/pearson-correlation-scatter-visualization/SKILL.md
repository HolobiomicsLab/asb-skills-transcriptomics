---
name: pearson-correlation-scatter-visualization
description: Use when validating that clustering features (e.g., Spearman correlation
  features from sscClust) remain robust to subsampling fluctuations across multiple
  subsampling rates of an scRNA-seq dataset.
when_to_use_negative:
- The goal is to measure subsampling bias itself (e.g., comparing accuracy degradation
  across fractions) rather than to verify robustness via near-zero correlation
- Only a single subsampling rate is available, so no silhouette-vs-rate correlation
  can be computed
- The dataset lacks true cell-type labels, since silhouette values must be computed
  against known labels
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SSCC
  role: Spearman subsampling-clustering-classification framework whose Spearman-correlation
    features are evaluated for subsampling robustness
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R package used to construct Spearman-correlation features and run the subsampling-clustering
    pipeline on each subsample
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language for the pipeline (ssc.run calls with subsampling=T
    and sub.frac parameters)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:03f7149a7eecbed853c802ce6430dc64ed633d63ddca67a0b4215207ae4e2382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pearson-correlation-scatter-visualization

## Summary

A robustness-diagnostic visualization in which per-cell silhouette values (X axis) from Spearman-correlation-based SSCC features are plotted against cell-subsampling rates (Y axis, 10%–50%) as scatter plots, with a Pearson correlation coefficient computed per dataset. Near-zero correlations indicate the engineered features stay consistent with true cluster labels regardless of the subsampled cell fraction.

## When to use

Use when validating that clustering features (e.g., Spearman correlation features from sscClust) remain robust to subsampling fluctuations across multiple subsampling rates of an scRNA-seq dataset.

## When NOT to use

- The goal is to measure subsampling bias itself (e.g., comparing accuracy degradation across fractions) rather than to verify robustness via near-zero correlation
- Only a single subsampling rate is available, so no silhouette-vs-rate correlation can be computed
- The dataset lacks true cell-type labels, since silhouette values must be computed against known labels

## Inputs

- scRNA-seq count matrices for the five benchmark datasets
- true cell-type label vectors per dataset
- subsampling rate grid (10%, 20%, 30%, 40%, 50%)

## Outputs

- Per-dataset scatter plots of per-cell silhouette value (X) versus subsampling rate (Y)
- Pearson correlation coefficient between silhouette values and subsampling rate per dataset

## How to apply

For each benchmark scRNA-seq count matrix with true cell-type labels, generate subsampled datasets at five cell-subsampling rates: 10%, 20%, 30%, 40%, and 50%. For each subsample, construct Spearman-correlation features using the sscClust R package, cluster the subsample, and compute per-cell silhouette values against the true labels. Plot silhouette value (X) versus subsampling rate (Y) as scatter plots, one panel per dataset, and compute the Pearson correlation coefficient for each. Validation criterion: accept when the Pearson correlation coefficients between silhouette values and subsampling rates are near zero, indicating silhouette quality is nearly independent of subsampling rate. The sscClust README recommends R linked to an optimized BLAS library (ATLAS, MKL, or Microsoft R open on Windows) because subsampling-clustering calculations become expensive on large datasets.

## Related tools

- **SSCC** (Spearman subsampling-clustering-classification framework whose Spearman-correlation features are evaluated for subsampling robustness) — https://github.com/Japrin/sscClust
- **sscClust** (R package used to construct Spearman-correlation features and run the subsampling-clustering pipeline on each subsample) — https://github.com/Japrin/sscClust
- **R** (Implementation language for the pipeline (ssc.run calls with subsampling=T and sub.frac parameters))

## Examples

```
sce.sub <- ssc.run(sce, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Pearson correlation coefficient between per-cell silhouette values and subsampling rate is near zero for each of the five benchmark datasets
- Silhouette-value distributions are almost unchanged across the 10%, 20%, 30%, 40%, and 50% subsampling rates
- Each dataset produces its own silhouette-versus-rate scatter plot with the specified axis orientation (silhouette on X, subsampling rate on Y)
- Spearman-correlation features are built with sscClust for every subsample before clustering, and clustering is compared against the true cell-type labels

## Limitations

- Results were demonstrated on five benchmark scRNA-seq datasets; behavior on datasets outside this range of technologies, sequencing depths, and organisms is not established by this analysis
- The SCC strategy can suffer from biases introduced by subsampling that decrease clustering accuracy and robustness, so conclusions depend on the Spearman feature construction step performing as intended
- The sscClust README warns that calculation becomes expensive for large datasets unless R is linked to an optimized BLAS library (ATLAS, MKL) 

## Evidence

- [results] silhouette-vs-rate robustness check: [evidence span withheld — anchor status: not_found]
- [methods] validation criterion: [evidence span withheld — anchor status: not_found]
- [readme] feature construction tool: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient, clustering with subsampling and classfification"
- [readme] subsampling invocation: [evidence span withheld — anchor status: external_doc]
- [readme] performance caveat: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library"
