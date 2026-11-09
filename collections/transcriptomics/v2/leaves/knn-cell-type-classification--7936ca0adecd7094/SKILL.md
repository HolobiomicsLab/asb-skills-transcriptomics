---
name: knn-cell-type-classification
description: Use when clustering very large scRNA-seq datasets (e.g., tens of thousands
  of cells like the 49,300-cell Macosko retina set) where clustering all cells is
  too expensive, so labels must be propagated from a subsample (e.g., 5% or 10%) to
  the rest via k-NN.
when_to_use_negative:
- Dataset is small enough that direct clustering of all cells (e.g., ssc.run with
  subsampling=F) is computationally feasible — subsampling plus k-NN adds unnecessary
  indirection.
- You need cluster labels only for the subsampled cells, not for the full cell population.
- 'You lack a reliable reference clustering on the subsample: k-NN classification
  propagates whatever bias the subsample clustering introduces.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SSCC
  role: Framework whose classification step assigns k-means cluster labels from a
    Spearman-correlation feature space (after random projection) to non-subsampled
    cells via k-NN
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language; sscClust R package provides ssc.run with subsampling=T
    to perform subsample clustering plus k-NN classification
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:969debd47db8148c59f919b5ec01e247dcb863ca0ac69397ed80c16d46a219c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# knn-cell-type-classification

## Summary

The classification step of the subsampling-clustering-classification (SCC/SSCC) strategy: k-means clusters are learned on a random cell subsample, then k-nearest-neighbor classification transfers those labels to all remaining cells, reducing complexity from O(n^2) to O(n). Within SSCC, k-NN operates on a Spearman-correlation feature space after random projection, which also improves cross-scheme consistency.

## When to use

Use when clustering very large scRNA-seq datasets (e.g., tens of thousands of cells like the 49,300-cell Macosko retina set) where clustering all cells is too expensive, so labels must be propagated from a subsample (e.g., 5% or 10%) to the rest via k-NN.

## When NOT to use

- Dataset is small enough that direct clustering of all cells (e.g., ssc.run with subsampling=F) is computationally feasible — subsampling plus k-NN adds unnecessary indirection.
- You need cluster labels only for the subsampled cells, not for the full cell population.
- You lack a reliable reference clustering on the subsample: k-NN classification propagates whatever bias the subsample clustering introduces.

## Inputs

- scRNA-seq cell-by-gene expression matrix (e.g., 49,300-cell Macosko retina dataset) preprocessed for the sscClust R framework
- Random cell subsample at a specified fraction (e.g., 5% or 10%)
- k-means cluster labels computed on the subsample

## Outputs

- Cluster/class label assigned to every cell outside the subsample via k-NN classification
- Per-cell cluster label table for the full dataset (SCC vs SSCC)
- Consistency metrics between subsampling schemes (silhouette difference, NMI, Pearson correlation of per-cell silhouette values)

## How to apply

Load the scRNA-seq expression matrix (e.g., the 49,300-cell Macosko retina dataset) and preprocess it within the sscClust R framework. Draw a random subsample at a chosen fraction (e.g., 5% or 10% of cells) and run k-means clustering on it; then apply k-nearest-neighbor classification to assign the learned cluster labels to the cells not in the subsample. Within the full SSCC framework, first project the data into a feature space built from Spearman correlations and apply random projection before clustering and classification, which reduces subsampling bias. Decision point: plain SCC introduces subsampling biases that lower accuracy and robustness, so use the SSCC feature-engineering + random-projection variant when consistency across subsampling schemes matters. Validate by comparing metrics between subsampling schemes (e.g., silhouette difference, NMI, and Pearson correlation of per-cell silhouettes).

## Related tools

- **SSCC** (Framework whose classification step assigns k-means cluster labels from a Spearman-correlation feature space (after random projection) to non-subsampled cells via k-NN) — https://github.com/Japrin/sscClust
- **R** (Implementation language; sscClust R package provides ssc.run with subsampling=T to perform subsample clustering plus k-NN classification) — https://github.com/Japrin/sscClust

## Examples

```
sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Average silhouette difference between the 5% and 10% subsampling schemes should be near-negligible for SSCC (~0.01) versus ~0.29 for plain SCC on the Macosko retina dataset.
- NMI between the two subsampling schemes should increase from ~0.60 (SCC) to ~0.69 (SSCC).
- Pearson correlation of per-cell silhouette values between the two schemes should rise from ~0.47 (SCC) to ~0.58 (SSCC).
- All metrics should be written to scc_sscc_macosko_benchmark.csv for reproducible comparison.
- Every non-subsampled cell must receive exactly one k-NN label, i.e., full label coverage of the dataset.

## Limitations

- The plain SCC strategy suffers from biases introduced by subsampling which may greatly decrease clustering accuracy and robustness; k-NN inherits any bias in the subsample's k-means clustering.
- README recommends linking R to optimized BLAS (ATLAS, MKL; Microsoft R Open on Windows) because computation becomes expensive as datasets grow.
- sscClust requires the companion sscVis package to be installed.
- Classification results depend on the chosen subsampling fraction (e.g., 5% vs 10%), so consistency checks across fractions are needed.

## Evidence

- [results] switching from SCC to SSCC made the average silhouette difference between the two subsampling schemes negligible (0.01 vs 0.29) and raised the NMI between the two schemes from 0.60 to 0.69: [evidence span withheld — anchor status: not_found]
- [methods] SCC: k-means clustering on the subsample followed by k-NN classification to assign labels to the remaining cells: [evidence span withheld — anchor status: not_found]
- [methods] SSCC, which adds Spearman-correlation-based feature engineering and random projection before clustering and classification: [evidence span withheld — anchor status: not_found]
- [discussion] the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness"
- [readme] clustering with subsampling and classfification which make it feasible to process thousands cells' data: "clustering with subsampling and classfification which make it feasible to process thousands cells' data"
