---
name: spatial-kernel-construction-from-coordinates
description: Use when spatial transcriptomics slides come with per-cell spatial coordinates
  and the goal is to remove local spatial-neighborhood effects to reveal global cell
  subpopulations hidden by spatial autocorrelation.
when_to_use_negative:
- When the goal is the opposite — enhancing or studying the spatial zonation/localization
  signal itself rather than removing it (e.g., the paper's circadian-zonation example
  filters spatial signal only to enhance the temporal circadian signal)
- When no per-cell spatial coordinates are available to define the similarity kernel,
  or when the signal of interest is batch rather than spatial (integration methods
  like bbknn, Harmony, or scVI are the paper's examples for that task)
- When downstream tasks require properly log-transformed expression — SiFT modifies
  gene expression so the filtered data is no longer properly log-transformed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: SiFT (sift-sc)
  role: Python package that computes the spatial-coordinate-based kernel (e.g. KnnKernel),
    projects expression onto it, and subtracts the projection to filter the local
    spatial signal
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: Installation channel for the sift-sc package
- name: Python
  role: Language in which the kernel construction and SiFT filtering are scripted
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:7e0e0db1843c0a1335780fd1b7d1a4649051c32fb548573c2dc6209e8e393008
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# spatial-kernel-construction-from-coordinates

## Summary

Constructs a cell-cell similarity kernel directly from spatial transcriptomics coordinates (e.g., a SiFT KnnKernel or distance kernel) so that the local spatial neighborhood signal can be projected out and filtered from expression data. In the APAP-induced liver regeneration dataset (GSE223561), this exposes global migratory subpopulations (ANXA2+ hepatocytes) otherwise obscured by local neighborhoods.

## When to use

Use when spatial transcriptomics slides come with per-cell spatial coordinates and the goal is to remove local spatial-neighborhood effects to reveal global cell subpopulations hidden by spatial autocorrelation.

## When NOT to use

- When the goal is the opposite — enhancing or studying the spatial zonation/localization signal itself rather than removing it (e.g., the paper's circadian-zonation example filters spatial signal only to enhance the temporal circadian signal)
- When no per-cell spatial coordinates are available to define the similarity kernel, or when the signal of interest is batch rather than spatial (integration methods like bbknn, Harmony, or scVI are the paper's examples for that task)
- When downstream tasks require properly log-transformed expression — SiFT modifies gene expression so the filtered data is no longer properly log-transformed

## Inputs

- Spatial transcriptomics count matrices with per-cell spatial coordinates (AnnData objects), e.g. APAP-injury liver slides from GEO accession GSE223561
- Choice of kernel type (distance/knn-type, e.g. sift.kernels.KnnKernel) built from the coordinate columns

## Outputs

- SiFT-filtered expression matrix per slide with the local spatial neighborhood signal removed
- Concatenated filtered dataset suitable for joint PCA, neighbor graph, and Leiden clustering across slides
- Cluster marker gene table (e.g. markers.csv) with Anxa2 ranking among top markers of the migratory cluster (cluster 1)

## How to apply

Load each ST slide (e.g., APAP-injury slides from GEO accession GSE223561) as an AnnData object holding the count matrix and the spatial coordinates. Build a cell-cell similarity kernel from the coordinates using a distance/knn-type kernel such as sift.kernels.KnnKernel, so that cells near each other in tissue are deemed similar with respect to the signal to filter. Run sift.SiFT with this kernel per slide: SiFT projects the expression count matrix onto the spatial kernel via matrix multiplication and deducts the projection, yielding a filtered expression matrix that removes the local neighborhood signal. Concatenate the SiFT-filtered slides across samples, embed them (PCA plus neighbor graph), and perform joint Leiden clustering; then compute marker genes with scanpy rank_genes_groups. The rationale, per the SiFT paper, is that a kernel capturing similarity 'with respect to the signal to be filtered' allows the projection-then-subtraction step to erase the encoded (spatial) signal while preserving global biological structure.

## Related tools

- **SiFT (sift-sc)** (Python package that computes the spatial-coordinate-based kernel (e.g. KnnKernel), projects expression onto it, and subtracts the projection to filter the local spatial signal) — https://github.com/nitzanlab/sift-sc
- **pip** (Installation channel for the sift-sc package)
- **Python** (Language in which the kernel construction and SiFT filtering are scripted)

## Examples

```
import sift; kernel = sift.kernels.KnnKernel(n_neighbors=k, adata=adata, keys=['spatial_x','spatial_y']); sft = sift.SiFT(kernel=kernel); sft.fit(adata)
```

## Evaluation signals

- After filtering and joint Leiden clustering, cluster 1 has Anxa2 uniquely among its top marker genes (scanpy rank_genes_groups), identifying the ANXA2+ migratory hepatocyte subpopulation
- Every cluster contains comparable proportions of cells from each APAP-injury slide (per-cluster sample compositions are homogeneous across slides)
- Cluster 1 shows a migratory GO-term phenotype, consistent with the reported Fig. 5 claims
- Cluster 1 cells are spatially dispersed rather than confined to local neighborhoods, reflecting removal of the local spatial signal

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel — a poorly chosen spatial kernel may filter the wrong signal
- No ground truth reference is available to directly evaluate computational filtering results; integration metrics, which may be application-specific, must be used instead
- SiFT modifies gene expression so the filtered data is no longer properly log-transformed, which matters for certain downstream tasks
- Statistical implications of applying commonly used analysis tools to corrected gene expression data remain unexplored

## Evidence

- [intro] Kernel construction from coordinates: "These attributes are used to compute a cell–cell similarity kernel with respect to the encoded signal"
- [intro] Projection and subtraction: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [discussion] Liver regeneration spatial filtering: "where we used the spatial coordinates to remove local signals and expose global subpopulations of interest involved in the regeneration process"
- [intro] Global subpopulations exposed: "global subpopulations involved in the regeneration process are exposed by filtering the local neighborhood of cells using the corresponding spatial coordinates"
- [methods] Kernel API availability: "sift.kernels.RBFKernel
sift.kernels.LaplacianKernel
sift.kernels.KnnKernel
sift.kernels.MappingKernel"
