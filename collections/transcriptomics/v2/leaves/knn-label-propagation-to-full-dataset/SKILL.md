---
name: knn-label-propagation-to-full-dataset
description: Use when clusters or labels have been assigned only to a sketched subset
  of cells (e.g., Sphetcher spherical or geometric sketch) and you need to infer labels
  for all cells of the original expression/PCA matrix.
when_to_use_negative:
- Labels already exist for every cell (no sketch-based subsampling was performed),
  so propagation would only add classifier noise.
- The sketch labels come from a method whose distance metric is incompatible with
  the features used for kNN (the paper notes the interplay of sketching distance metric
  with downstream algorithms was not systematically addressed).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: Produces the spherical sketch indicator vector whose sketched cells serve
    as labeled training set for the kNN classifier; used with Pearson correlation
    as distance metric.
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Community detection on the kNN graph of sketch cells provides the cluster
    labels that are propagated to the full dataset.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:d498a61368107a26b528b6803d11cb0b63a32ffa3aac23b9c5d79e54094c2bb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kNN label propagation to full dataset

## Summary

Extends cluster labels computed on a cell sketch (e.g., Louvain communities on Sphetcher's spherical sketch) back to every cell of the original scRNA-seq dataset by training a k-nearest-neighbor classifier on the sketched cells' features, enabling whole-dataset evaluation of sketch quality via Adjusted Rand Index.

## When to use

Use when clusters or labels have been assigned only to a sketched subset of cells (e.g., Sphetcher spherical or geometric sketch) and you need to infer labels for all cells of the original expression/PCA matrix.

## When NOT to use

- Labels already exist for every cell (no sketch-based subsampling was performed), so propagation would only add classifier noise.
- The sketch labels come from a method whose distance metric is incompatible with the features used for kNN (the paper notes the interplay of sketching distance metric with downstream algorithms was not systematically addressed).

## Inputs

- sketch_indicator_output.csv: indicator vector of n samples where 1 indicates the sample is in the sketch, 0 otherwise
- PCA feature matrix (CSV, rows are cells, columns are genes/PCs), e.g., zeisel_pca.csv
- Louvain cluster assignments for the sketched cells
- Ground-truth cell-type annotations for all cells

## Outputs

- Propagated cluster labels for every cell of the original dataset
- ari_results.csv table of ARI values per (dataset, sketching method, sketch size)
- Grouped bar chart comparing spherical vs. geometric sketch ARI (e.g., figure3_s4_ari.png)

## How to apply

After Louvain community detection assigns clusters to the sketched cells (extracted from sketch_indicator_output.csv rows with indicator 1), train a k-nearest-neighbor classifier using the sketch cells' feature vectors — typically the PCA feature matrix used as Sphetcher input, where rows are cells and columns are genes/PCs. Use the fitted classifier to predict cluster labels for all n cells of the original dataset. Then compare propagated labels against ground-truth cell-type annotations with a chance-adjusted agreement metric (Adjusted Rand Index) per dataset and sketch size (1%, 4%, 10%). For Sphetcher sketches the underlying metric is Pearson correlation, so kNN distance choices should be consistent with the sketching metric. Tabulate ARI values (dataset, sketching method, sketch size, ARI) so spherical vs. geometric sketches can be compared, validation passing when relative ARI trends match the published Figure 3/S4.

## Related tools

- **Sphetcher** (Produces the spherical sketch indicator vector whose sketched cells serve as labeled training set for the kNN classifier; used with Pearson correlation as distance metric.) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Community detection on the kNN graph of sketch cells provides the cluster labels that are propagated to the full dataset.)

## Examples

```
sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv  # then train kNN on rows with indicator 1 and predict labels for all cells
```

## Evaluation signals

- The indicator vector contains exactly sketch_size 1's (e.g., 1000 for the zeisel_pca.csv example), confirming the correct sketched subset is used for training.
- ARI between propagated labels and ground-truth annotations is computed for every (dataset, method, sketch size) combination and populated in ari_results.csv.
- Relative ARI trends across sketch sizes and methods reproduce the published comparison: spherical sketches better or comparable in most of 36 instances, geometric better in only three.
- Propagated label count equals the number of cells in the original dataset (e.g., 465,281 for Zeisel mouse nervous system).
- Rare cell types are not systematically missed after propagation, consistent with the spherical sketch's more even coverage of transcriptomic space.

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study.
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2.
- No software version numbers were stated for Sphetcher or the downstream tools, complicating exact reproduction.

## Evidence

- [readme] Output formats: "sketch_indicator_output.csv : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [methods] Input format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [discussion] Distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [discussion] Clustering tool: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering"
- [other] Metric: [evidence span withheld — anchor status: not_found]
