---
name: unsampled-cell-classification-projection
description: Use when a large-scale scRNA-seq matrix was clustered on a random subsample
  and the remaining unsampled cells must be assigned to clusters without rerunning
  the base clustering algorithm on the full dataset.
when_to_use_negative:
- The full dataset is small enough that the base clustering algorithm can run directly
  on all cells without subsampling (e.g., ssc.run with subsampling=F)
- Cells already have ground-truth labels or the analysis requires de novo clustering
  of the unsampled cells rather than nearest-neighbor assignment to subsampled clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSCC
  role: Computational framework defining the Spearman-correlation feature projection
    and the classification step for unsampled cells within the subsampling-clustering-classification
    workflow
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R implementation providing ssc.run with subsampling=T and Spearman-correlation-based
    projection used to execute the workflow
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Runtime environment for sscClust; an optimized BLAS-linked R build is recommended
    for large datasets
  repo: https://cran.r-project.org/doc/manuals/r-release/R-admin.html#BLAS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:c081df5f0fb177e6e70c20aa27c4b747d9c80949db7eefb8add6dcc7c879f7be
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# unsampled-cell-classification-projection

## Summary

Projects cells that were not included in the random subsample into the Spearman-correlation feature space built from subsampled cells and assigns each to the cluster of its nearest subsampled cells, completing the SSCC subsampling-clustering-classification workflow while keeping complexity at O(n).

## When to use

Use when a large-scale scRNA-seq matrix was clustered on a random subsample and the remaining unsampled cells must be assigned to clusters without rerunning the base clustering algorithm on the full dataset.

## When NOT to use

- The full dataset is small enough that the base clustering algorithm can run directly on all cells without subsampling (e.g., ssc.run with subsampling=F)
- Cells already have ground-truth labels or the analysis requires de novo clustering of the unsampled cells rather than nearest-neighbor assignment to subsampled clusters

## Inputs

- Gene expression matrix of single-cell RNA-seq count data loaded in R (sscClust data-preparation utilities)
- Cluster labels for the randomly subsampled cells produced in the Spearman-correlation feature space
- Subsample fraction (e.g., sub.frac=0.8) defining which cells are unsampled

## Outputs

- Cluster assignment for every unsampled cell (classification labels)
- Complete cluster assignment table combining subsampled and classified cells (e.g., sscc_clusters.csv)

## How to apply

After clustering the randomly subsampled cells in the constructed Spearman-correlation feature space, compute Spearman correlations between each unsampled cell and the subsampled cells to project it into the same feature space. Assign each projected cell to the cluster of its nearest subsampled cell(s) (classification step, Step 5 of the SSCC/SFCC workflow). This projection via Spearman correlation is the key extension over the plain SCC strategy, which clusters only the subsample and handles unsampled cells crudely, and it replaces direct use of raw gene expression for cell assignment. Combine the subsampled cells' cluster labels with the classified cells' labels to obtain the complete cluster assignment table (e.g., write to sscc_clusters.csv). Use the sscClust R package with subsampling enabled (e.g., sub.frac to control the subsample fraction) and an R build linked to an optimized BLAS library, as recommended in the README.

## Related tools

- **SSCC** (Computational framework defining the Spearman-correlation feature projection and the classification step for unsampled cells within the subsampling-clustering-classification workflow) — https://github.com/Japrin/sscClust
- **sscClust** (R implementation providing ssc.run with subsampling=T and Spearman-correlation-based projection used to execute the workflow) — https://github.com/Japrin/sscClust
- **R** (Runtime environment for sscClust; an optimized BLAS-linked R build is recommended for large datasets) — https://cran.r-project.org/doc/manuals/r-release/R-admin.html#BLAS

## Examples

```
sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Every cell in the input matrix receives a cluster label after combining subsampled and classified cells — no unsampled cell is left unassigned
- Unsampled cells are assigned via the same Spearman-correlation feature space used to cluster the subsample, not via raw gene expression
- End-to-end runtime and memory scale linearly with cell count (O(n)), consistent with the reported O(n2) to O(n) complexity reduction
- Cluster labels of unsampled cells agree with subsampled-cell clusters in the merged table (e.g., sscc_clusters.csv has one consistent label column)
- Accuracy on labeled benchmark datasets (e.g., 68,578-cell human blood dataset) should not degrade relative to clustering the full data directly

## Limitations

- Subsampling introduces biases that can decrease clustering accuracy and robustness; the Spearman feature projection is designed to mitigate but not eliminate this
- Assignment quality depends on the chosen subsample fraction (e.g., sub.frac=0.8) and nearest-neighbor assignment; very small subsamples may misclassify rare cell populations
- No parameter settings for how Spearman correlations are computed are given in the discussion section, limiting reproducibility of the projection details

## Evidence

- [discussion] with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [abstract] Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction: "Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [intro] SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering: "SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [readme] clustering with subsampling and classfification which make it feasible to process thousands cells' data.: "clustering with subsampling and classfification which make it feasible to process thousands cells' data."
