---
name: weighted-nearest-neighbor-multimodal-clustering
description: Use when a droplet-based single-cell experiment (CITE-seq, ASAP-seq,
  TEA-seq) has paired mRNA and ADT matrices and you need joint clustering to test
  whether a protein normalization method (e.g., dsb vs CLR) resolves a specific cell
  population.
when_to_use_negative:
- Data is single-modality (mRNA only or ADT only) — WNN requires paired modalities
  to weight
- Protein counts are not yet normalized — clustering raw/ambient-contaminated ADT
  values can conflate clusters with ambient antibody noise
- Proteins with very low raw signal (maximum raw UMI count <5 across all cells) have
  not been removed prior to normalization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: Runs FindMultiModalNeighbors (WNN, dims.list = list(1:30, 1:18)) and FindClusters
    (algorithm=3, resolution=2) on the joint mRNA+ADT graph
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Provides the dsb-normalized protein values compared against CLR in the joint
    clustering; interfaces with Seurat
  repo: https://github.com/niaid/dsb
- name: R
  role: Implementation environment for the WNN clustering workflow (prcomp for mRNA
    PCs and the dsb/Seurat pipeline)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:fbfe54e86653c826c8b81019c59269137548128bde3dc18a0f2edf3125d7b297
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-data
  tools:
  - R
  - Seurat
  - dsb
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Weighted nearest neighbor multimodal clustering

## Summary

Jointly clusters single-cell mRNA and protein (ADT) data by combining RNA principal components with normalized protein values via Seurat's WNN (FindMultiModalNeighbors), then calling clusters on the joint graph. Used in the dsb paper to show dsb normalization unmasks biologically meaningful populations (e.g., MAIT cells in TEA-seq cluster 14) that CLR normalization obscures.

## When to use

Use when a droplet-based single-cell experiment (CITE-seq, ASAP-seq, TEA-seq) has paired mRNA and ADT matrices and you need joint clustering to test whether a protein normalization method (e.g., dsb vs CLR) resolves a specific cell population.

## When NOT to use

- Data is single-modality (mRNA only or ADT only) — WNN requires paired modalities to weight
- Protein counts are not yet normalized — clustering raw/ambient-contaminated ADT values can conflate clusters with ambient antibody noise
- Proteins with very low raw signal (maximum raw UMI count <5 across all cells) have not been removed prior to normalization

## Inputs

- Raw and filtered Cell Ranger mRNA and ADT (protein) UMI count matrices (e.g., from GEO, Swanson et al. TEA-seq)
- ADT matrix normalized by the method under comparison (dsb-normalized and CLR-normalized values)
- mRNA PCA embeddings (30 principal components)

## Outputs

- Joint WNN neighbor graph and cluster assignments per cell (Seurat FindClusters, algorithm=3, resolution=2)
- Cluster-level protein expression tables under dsb and CLR normalization
- Comparison of cluster identity (e.g., exclusive TCR-Va7.2 positivity of cluster 14 under dsb)

## How to apply

Define cells as barcodes in the filtered Cell Ranger output and background drops as the remaining raw barcodes with protein reads. Normalize the 45-protein ADT matrix with the method under test (dsb::DSBNormalizeProtein with denoise.counts=TRUE, use.isotype.control=TRUE, and an empty-droplet background matrix, versus CLR via Seurat NormalizeData). For each normalization, compute 30 mRNA principal components, run Seurat's FindMultiModalNeighbors with dims.list = list(1:30, 1:18), and cluster the joint graph with FindClusters (algorithm=3, resolution=2). The same 30 mRNA PCs are combined with either dsb- or CLR-normalized protein data so any cluster-level differences are attributable to the protein normalization. Inspect cluster-level protein values (e.g., TCR-Va7.2 exclusivity) to link clusters to cell identities.

## Related tools

- **Seurat** (Runs FindMultiModalNeighbors (WNN, dims.list = list(1:30, 1:18)) and FindClusters (algorithm=3, resolution=2) on the joint mRNA+ADT graph) — https://github.com/satijalab/seurat-data
- **dsb** (Provides the dsb-normalized protein values compared against CLR in the joint clustering; interfaces with Seurat) — https://github.com/niaid/dsb
- **R** (Implementation environment for the WNN clustering workflow (prcomp for mRNA PCs and the dsb/Seurat pipeline))

## Examples

```
seurat_obj <- FindMultiModalNeighbors(seurat_obj, reduction.list = list('pca', 'adt_pca'), dims.list = list(1:30, 1:18)); seurat_obj <- FindClusters(seurat_obj, graph.name = 'wsnn', algorithm = 3, resolution = 2)
```

## Evaluation signals

- Cluster of interest (cluster 14 in TEA-seq) is exclusively TCR-Va7.2-positive under dsb normalization, not under CLR
- The same 30 mRNA PCs and identical clustering parameters (algorithm=3, resolution=2) are used for both dsb and CLR arms, isolating the protein-normalization effect
- dims.list for WNN is list(1:30, 1:18) covering 30 mRNA PCs and 18 protein dimensions of the 45-protein ADT panel
- Cluster assignments are stable when recomputed from the same joint graph

## Limitations

- Results depend on how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Empty-droplet definition and dsb sensitivity caveats are detailed in the Supplementary Note, not the main text
- CLR (median) values can correlate with the 98th percentile empty-droplet expression (r=0.8, p=2.4e-7 vs r=0.13, p=0.39 for dsb), so joint clusters built on CLR ADT may reflect ambient noise rather than biology

## Evidence

- [discussion] Multimodal WNN clustering combining 30 mRNA principal components with normalized protein values: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] Cluster definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [discussion] Low-signal protein removal: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [discussion] Background droplet caveat: "defining a reasonable subset of barcodes as background droplets still requires care"
- [readme] dsb interfaces with Seurat: "The functions in this package return standard R matrix objects that can be added to any data container like a `SingleCellExperiment`, `Seurat`,
or `AnnData` related python objects."
