---
name: frobenius-norm-distance-metrics
description: Use when you must choose the optimal gene feature set size for single-cell
  clustering from a label-free curve, sweeping DUBStepR feature set sizes and comparing
  a label-free metric like the Density Index against the label-dependent silhouette
  index.
when_to_use_negative:
- Cell type labels are available and only the supervised silhouette index is needed
  — the DI sweep is specifically for label-free selection
- You have not yet produced PC embeddings from DUBStepR-ordered features; the metric
  requires a PC-space representation per feature set size
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DUBStepR
  role: Provides the correlation-based feature ordering and getOptimalFeatureSet machinery
    whose feature set size sweep the DI/SI metrics are computed over
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Environment for loading the scRNA-seq data and producing the PC-space cell
    embeddings used to compute the metrics
- name: R
  role: Implementation language; DUBStepR requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:9ee7391d95f7c6a42aaca47f966ee059f57edba5fb60c95549c4fa36b01ae82f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Frobenius-norm distance metrics

## Summary

Computes label-free global distance metrics (e.g., the Density Index) in principal-component space using Frobenius-norm shortcuts for root-mean-squared pairwise distances, as used by DUBStepR to select the optimal feature set size without cell type labels.

## When to use

Use when you must choose the optimal gene feature set size for single-cell clustering from a label-free curve, sweeping DUBStepR feature set sizes and comparing a label-free metric like the Density Index against the label-dependent silhouette index.

## When NOT to use

- Cell type labels are available and only the supervised silhouette index is needed — the DI sweep is specifically for label-free selection
- You have not yet produced PC embeddings from DUBStepR-ordered features; the metric requires a PC-space representation per feature set size

## Inputs

- Benchmark scRNA-seq datasets (e.g., 10X Genomics pbmc_1k_v2) with independent cell type annotations, loaded into Seurat objects via hdf5r
- DUBStepR-ordered feature lists (gene-gene correlation range z-score ordering) at varying feature set sizes
- Cell embeddings in PC space (num.pcs = 20, k = 10) per feature set size

## Outputs

- Per-dataset table of Density Index versus feature set size
- Per-dataset table of Silhouette Index versus feature set size
- di_si_agreement_record stating agreement between DI-argmax and SI-argmax feature set sizes (expected: 5 of 7 datasets)

## How to apply

Run DUBStepR's correlation-based feature ordering (DUBStepR defaults: k = 10, num.pcs = 20) on each scRNA-seq dataset, then embed cells in PC space for a sweep of feature set sizes (the reported maximum silhouette index occurs at 200-500 feature genes). At each feature set size, compute the Density Index as the root-mean-squared pairwise distance over the mean k-NN distance in PC space, evaluating the RMS pairwise term efficiently via the Frobenius norm of the PC coordinate matrix. In parallel, compute the silhouette index of the clustering using known cell type labels in the same PC space. Tabulate DI and SI versus feature set size, take the argmax of each curve, and record agreement per dataset; in the DUBStepR benchmarking, the feature set with the highest DI also maximized SI in 5 of 7 benchmark datasets (consistent with Fig 5c,d). The agreement record (di_si_agreement_record) is the validation artifact.

## Related tools

- **DUBStepR** (Provides the correlation-based feature ordering and getOptimalFeatureSet machinery whose feature set size sweep the DI/SI metrics are computed over) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Environment for loading the scRNA-seq data and producing the PC-space cell embeddings used to compute the metrics)
- **R** (Implementation language; DUBStepR requires R >= 3.5.0)

## Examples

```
opt.res <- getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k=10, num.pcs=20, error=0)
```

## Evaluation signals

- The di_si_agreement_record states the DI-maximizing feature set coincides with the SI-maximizing feature set in 5 of the 7 benchmark datasets, consistent with Fig 5c,d
- Both DI and SI curves are positively correlated and reach maxima at approximately the same feature set size
- Maximum silhouette index falls in the reported 200-500 feature genes range (Supp. Fig. S3)
- The DI's RMS pairwise-distance term equals the Frobenius norm of the PC coordinate matrix for each feature set size
- DUBStepR was run with default parameters k = 10, num.pcs = 20

## Limitations

- Key results, including the 200-500 feature genes silhouette behavior (Supp. Fig. S3), are deferred to supplementary figures and notes not included in the main text
- The benchmark dataset collection is only identified by reference number 28 in the discussion, not named or accessioned in the main text, complicating exact reproduction
- The elbow point computation algorithm is described in Supp. Note 1B, not in the main text

## Evidence

- [discussion] silhouette peak range: "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
- [readme] DUBStepR method intuition: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [methods] R version requirement: "DUBStepR requires your R version to be >= 3.5.0"
- [intro] GGC feature ordering: "DUBStepR constructs a GGC matrix and bins genes by expression to compute their correlation range z-scores, which are used to select well-correlated genes."
