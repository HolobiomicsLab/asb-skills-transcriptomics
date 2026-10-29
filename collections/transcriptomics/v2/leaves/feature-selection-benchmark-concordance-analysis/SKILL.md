---
name: feature-selection-benchmark-concordance-analysis
description: Use when determining the optimal number of DUBStepR-ordered feature genes
  for clustering scRNA-seq data, and for testing whether a label-free criterion (Density
  Index) agrees with label-based silhouette index across benchmark datasets.
when_to_use_negative:
- When gold-standard cell type labels are available for the actual analysis goal and
  a supervised label-based metric alone suffices — the DI proxy is intended for label-free
  feature set size selection
- When the input is already a reduced feature table or PC embedding rather than a
  raw gene-by-cell count matrix, since DUBStepR requires raw expression data to construct
  the gene-gene correlation (GGC) matrix
- When clustering features were selected by a method other than DUBStepR correlation
  ordering, since the DI-SI concordance (5 of 7 datasets) was established for DUBStepR-ordered
  feature sets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: Performs correlation-based feature ordering (GGC matrix, correlation range
    z-scores, stepwise regression seed features) and provides getOptimalFeatureSet
    to select the optimal feature set; default parameters k = 10, num.pcs = 20
  repo: github.com/prabhakarlab/DUBStepR
- name: R
  role: Runtime environment for the DI/SI sweep; DUBStepR requires R >= 3.5.0
- name: Seurat
  role: Seurat object infrastructure for loading scRNA-seq data and supporting PC-space
    embedding used by DI and SI computation
- name: hdf5r
  role: Loads 10X HDF5 data (e.g., pbmc_1k_v2) into the Seurat object
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:9261dd7b6685f004504f9d0d1a170e635bd6c00a1874952aa47aa56095813298
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - hdf5r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# feature-selection-benchmark-concordance-analysis

## Summary

Reproduces the DUBStepR Density Index (DI)-guided choice of optimal feature set size for single-cell clustering and validates it against the label-based silhouette index (SI), which peaks at 200–500 feature genes. The skill sweeps feature set sizes in PC space, tabulates DI and SI curves per benchmark dataset, and records agreement between the DI- and SI-argmax feature sets.

## When to use

Use when determining the optimal number of DUBStepR-ordered feature genes for clustering scRNA-seq data, and for testing whether a label-free criterion (Density Index) agrees with label-based silhouette index across benchmark datasets.

## When NOT to use

- When gold-standard cell type labels are available for the actual analysis goal and a supervised label-based metric alone suffices — the DI proxy is intended for label-free feature set size selection
- When the input is already a reduced feature table or PC embedding rather than a raw gene-by-cell count matrix, since DUBStepR requires raw expression data to construct the gene-gene correlation (GGC) matrix
- When clustering features were selected by a method other than DUBStepR correlation ordering, since the DI-SI concordance (5 of 7 datasets) was established for DUBStepR-ordered feature sets

## Inputs

- Seven benchmark scRNA-seq datasets with cell type annotations (e.g., 10X Genomics pbmc_1k_v2 PBMC dataset)
- Raw gene-by-cell count matrices loadable into a Seurat object (e.g., via the hdf5r package for 10X HDF5 files)
- Known cell type label annotations per cell for silhouette index computation
- DUBStepR-ordered feature gene lists and PC embeddings (k = 10, num.pcs = 20)

## Outputs

- Per-dataset table of Density Index vs feature set size
- Per-dataset table of silhouette index vs feature set size
- Agreement record (di_si_agreement_record) of DI-argmax vs SI-argmax feature set sizes, expected to show concordance in 5 of 7 datasets
- DI-optimal feature set selected via getOptimalFeatureSet-style evaluation

## How to apply

Load each benchmark scRNA-seq dataset with cell type annotations in R (>= 3.5.0) using the DUBStepR/Seurat environment. Run DUBStepR correlation-based feature ordering (filtering mitochondrial, ribosomal, spike-in, and pseudogenes; GGC matrix with correlation range z-scores) and embed cells in PC space with the DUBStepR defaults k = 10 and num.pcs = 20 over a sweep of feature set sizes. At each size, compute the Density Index as the root-mean-squared pairwise distance over the mean k-NN distance in PC space (RMS pairwise term via the Frobenius norm of the PC coordinate matrix), and compute the silhouette index using known cell type labels in the same PC space. Identify the feature set size maximizing each metric (SI is expected near 200–500 feature genes per the article), then record per-dataset DI-argmax vs SI-argmax agreement. Success is an agreement record showing the DI-maximizing feature set coincides with the SI-maximizing feature set in 5 of the 7 benchmark datasets (Fig 5c,d).

## Related tools

- **DUBStepR** (Performs correlation-based feature ordering (GGC matrix, correlation range z-scores, stepwise regression seed features) and provides getOptimalFeatureSet to select the optimal feature set; default parameters k = 10, num.pcs = 20) — github.com/prabhakarlab/DUBStepR
- **R** (Runtime environment for the DI/SI sweep; DUBStepR requires R >= 3.5.0)
- **Seurat** (Seurat object infrastructure for loading scRNA-seq data and supporting PC-space embedding used by DI and SI computation)
- **hdf5r** (Loads 10X HDF5 data (e.g., pbmc_1k_v2) into the Seurat object)

## Examples

```
DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)
```

## Evaluation signals

- The di_si_agreement_record states the DI-maximizing feature set coincides with the SI-maximizing feature set in 5 of the 7 benchmark datasets, consistent with Fig 5c,d
- DI and SI curves are positively correlated across feature set sizes within each dataset, reaching maxima at approximately the same feature set size
- The SI curve peaks at approximately 200–500 feature genes, matching Supp. Fig. S3
- PCA embedding uses DUBStepR defaults k = 10, num.pcs = 20; any deviation should be flagged
- Genes filtered before GGC construction exclude mitochondrial, ribosomal, spike-in, and pseudogenes

## Limitations

- DI-SI concordance holds for 5 of 7 benchmark datasets — disagreement in the remaining 2 is expected and should be reported rather than treated as failure
- The DI computation depends on DUBStepR's specific PC-space embedding (k = 10, num.pcs = 20); results may not transfer to other embeddings or feature selection methods
- Silhouette index computation requires independent cell type annotations, which are unavailable for truly unlabeled data — this is exactly the gap the label-free DI proxy addresses
- Key details (SI curve in Supp. Fig. S3, elbow point algorithm in Supp. Note 1B) are only in supplementary materials not included in the main text

## Evidence

- [discussion] Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3): "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
- [methods] getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error): "getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error)"
- [readme] After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [methods] DUBStepR requires your R version to be >= 3.5.0: "DUBStepR requires your R version to be >= 3.5.0"
- [intro] After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
