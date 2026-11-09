---
name: density-index-computation-pc-space
description: 'Use when tuning DUBStepR feature selection on scRNA-seq data without
  reliable cell type labels: sweeping feature set sizes and needing a label-free criterion
  whose argmax agrees with the silhouette index.'
when_to_use_negative:
- Cell type labels are fully trusted and available, making direct silhouette-index
  maximization feasible without a label-free proxy
- The pipeline does not use DUBStepR-ordered feature sets or PC-space embeddings (k
  = 10, num.pcs = 20), since the DI-SI agreement was established in that setting
- The task is feature ranking itself rather than choosing the feature set size — use
  DUBStepR's getOptimalFeatureSet/elbow-point machinery instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DUBStepR
  role: Provides correlation-based feature ordering and PC-space embedding (DUBStepR(input.data,
    min.cells, species, optimise.features, k, num.pcs, error)) whose feature set sizes
    are swept for DI computation
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Recommended environment for building the Seurat object from hdf5r-loaded 10X
    data prior to feature selection and clustering
  repo: https://satijalab.org/seurat/
- name: R
  role: Implementation language; DUBStepR requires R version >= 3.5.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:d64a8760f3f43da45fb0381c393c11a3dce89c69e256ea78350dd12e05d6ee60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# density-index-computation-pc-space

## Summary

Computes the Density Index (DI), a label-free measure of clustering inhomogeneity in PCA space, as the root-mean-squared pairwise distance over the mean k-NN distance, and uses its maximum over feature set sizes as a proxy for the silhouette index to select the optimal DUBStepR feature set size.

## When to use

Use when tuning DUBStepR feature selection on scRNA-seq data without reliable cell type labels: sweeping feature set sizes and needing a label-free criterion whose argmax agrees with the silhouette index.

## When NOT to use

- Cell type labels are fully trusted and available, making direct silhouette-index maximization feasible without a label-free proxy
- The pipeline does not use DUBStepR-ordered feature sets or PC-space embeddings (k = 10, num.pcs = 20), since the DI-SI agreement was established in that setting
- The task is feature ranking itself rather than choosing the feature set size — use DUBStepR's getOptimalFeatureSet/elbow-point machinery instead

## Inputs

- Preprocessed scRNA-seq expression matrix (Seurat object; hdf5r-loadable 10X data such as pbmc_1k_v2) with cell type annotations for benchmarking
- DUBStepR correlation-ordered feature gene list and PC embeddings (k = 10, num.pcs = 20) per feature set size

## Outputs

- Table of Density Index values versus feature set size per dataset
- Table of Silhouette Index values versus feature set size per dataset
- di_si_agreement_record stating DI-argmax vs SI-argmax concordance per dataset (expected: 5 of 7 datasets agree)

## How to apply

For each of the seven benchmark scRNA-seq datasets, run DUBStepR correlation-based feature ordering and embed cells in PC space (DUBStepR defaults k = 10, num.pcs = 20) over a sweep of feature set sizes. At each size, compute the DI as the root-mean-squared pairwise cell distance over the mean k-NN distance in PC space, evaluating the RMS pairwise term via the Frobenius norm of the PC coordinate matrix; in parallel compute the silhouette index (SI) using known cell type labels on the same PC space. Tabulate DI and SI versus feature set size per dataset, identify each metric's argmax (reported maximum SI occurs at 200–500 feature genes), and record DI-argmax vs SI-argmax agreement. Correct application is judged by the DI-maximizing feature set coinciding with the SI-maximizing feature set in 5 of 7 benchmark datasets, consistent with Fig 5c,d.

## Related tools

- **DUBStepR** (Provides correlation-based feature ordering and PC-space embedding (DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)) whose feature set sizes are swept for DI computation) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Recommended environment for building the Seurat object from hdf5r-loaded 10X data prior to feature selection and clustering) — https://satijalab.org/seurat/
- **R** (Implementation language; DUBStepR requires R version >= 3.5.0)

## Examples

```
res <- DUBStepR(input.data = seu@assays$RNA@counts, min.cells = 10, species = 'human', optimise.features = TRUE, k = 10, num.pcs = 20, error = 1e-4)
```

## Evaluation signals

- DI curve reaches a maximum at approximately the same feature set size as the SI curve (positive correlation of DI and SI across sizes)
- Agreement record confirms DI-argmax feature set coincides with SI-argmax feature set in 5 of 7 benchmark datasets (cf. Fig 5c,d)
- Maximum SI falls in the reported 200–500 feature genes range on benchmark datasets
- DI computed with k = 10 neighbors and num.pcs = 20 PCs matches DUBStepR defaults used for the published benchmark
- Per-dataset DI/SI curves are tabulated over the full feature set size sweep, not at a single size

## Limitations

- Key supporting results are deferred to supplementary figures (Supp. Fig. S3, S4) and supplementary notes (Supp. Note 1B) rather than the main text
- The DI-vs-SI agreement was established on a stringently defined benchmark collection of scRNA-seq datasets with independently ascertainable cell type annotations (reference 28, not named or accessioned in main text); agreement of 5-of-7 is not perfect
- Elbow point computation used for feature set selection is not described in the main text

## Evidence

- [supplementary] The Density Index and silhouette index were positively correlated and reached maxima at approximately the same feature set size: [evidence span withheld — anchor status: not_found]
- [discussion] Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3): "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
- [methods] DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error): "DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)"
- [readme] Feature selection, i.e. determining the optimal subset of genes to cluster cells into cell types, is a critical step: "Feature selection, i.e. determining the optimal subset of genes to cluster cells into cell types, is a critical step in the unsupervised clustering of scRNA-seq data."
- [discussion] we used a stringently defined collection of single-cell datasets for which cell type annotations could be independently ascertained28: "we used a stringently defined collection of single-cell datasets for which cell type annotations could be independently ascertained28."
