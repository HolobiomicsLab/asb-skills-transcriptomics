---
name: nuclei-segmentation-from-he-histology-images
description: Use when processing seq-based ST data (e.g. 10x Visium) that has a paired
  H&E image (~10000x10000 pixels) and you need per-spot cell counts and cell centroid
  locations before single-cell resolution reconstruction.
when_to_use_negative:
- No histological (or single-channel nuclear, e.g. DAPI) image is available — alternative
  image types or no images complicate cell count estimation and a different strategy
  is needed
- Data are spot-level but image-free such as Slide-seq — SpatialScope replaces nucleus
  segmentation with singlet/doublet classification instead
- Goal is pure downstream deconvolution of proportions per spot (e.g. RCTD/Cell2location/CARD
  alone) without single-cell resolution reconstruction — segmentation-based cell counts
  are not required
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SpatialScope
  role: Hosts the nucleus-segmentation pipeline (Step 1) that runs StarDist on the
    H&E image and outputs sp_adata_ns.h5ad with cell_locations per Visium spot
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist
  role: Deep-learning nucleus segmentation model invoked by SpatialScope to detect
    nuclei in the ~10000x10000 H&E heart Visium image
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:915895bbe4e6cd9f7b64e7fcbb89381dbe9c1efa9e07cb878ab57ae00295e8ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - Giotto
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# nuclei-segmentation-from-he-histology-images

## Summary

Segment nuclei in H&E-stained histology images (e.g. StarDist as used inside SpatialScope) to count cells per spatial transcriptomics spot, producing per-spot cell counts and single-cell spatial locations that feed downstream cell-type identification and expression decomposition.

## When to use

Use when processing seq-based ST data (e.g. 10x Visium) that has a paired H&E image (~10000x10000 pixels) and you need per-spot cell counts and cell centroid locations before single-cell resolution reconstruction.

## When NOT to use

- No histological (or single-channel nuclear, e.g. DAPI) image is available — alternative image types or no images complicate cell count estimation and a different strategy is needed
- Data are spot-level but image-free such as Slide-seq — SpatialScope replaces nucleus segmentation with singlet/doublet classification instead
- Goal is pure downstream deconvolution of proportions per spot (e.g. RCTD/Cell2location/CARD alone) without single-cell resolution reconstruction — segmentation-based cell counts are not required

## Inputs

- Raw H&E-stained histology image paired with the ST capture area (e.g. ~10000x10000 pixel Visium tissue image)
- Spot-level Visium ST data with spot coordinates (e.g. filtered tissue spots such as the 3813-spot human heart sample after removing spots not mapping to tissue or with total UMI counts < 100)

## Outputs

- Segmented AnnData object sp_adata_ns.h5ad containing per-spot cell counts and cell_locations (nucleus centroids per spot)
- Per-spot cell-count table used as input to SpatialScope cell type identification

## How to apply

Run the SpatialScope nucleus-segmentation step (StarDist model) on the raw H&E image matched to the spot-level ST data (e.g. the human heart Visium sample of ~10000x10000 pixels). The step detects and localizes each nucleus, quantifies how many cells fall within each 55 μm Visium spot, and writes a segmented AnnData object (sp_adata_ns.h5ad) carrying cell_locations per spot. These counts and locations are the building block for the subsequent SpatialScope cell type identification and gene expression decomposition steps, which need to know how many single cells to resolve per spot. Note that segmentation replaces or supplements singlet/doublet classification used for image-less platforms like Slide-seq.

## Related tools

- **SpatialScope** (Hosts the nucleus-segmentation pipeline (Step 1) that runs StarDist on the H&E image and outputs sp_adata_ns.h5ad with cell_locations per Visium spot) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist** (Deep-learning nucleus segmentation model invoked by SpatialScope to detect nuclei in the ~10000x10000 H&E heart Visium image)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git; conda env create -f environment.yml  # then run SpatialScope's StarDist-based Nuclei_Segmentation on the heart Visium H&E image to output sp_adata_ns.h5ad with cell_locations
```

## Evaluation signals

- sp_adata_ns.h5ad is produced and contains a cell_locations field with per-spot nucleus centroids
- Segmented cell counts are plausible relative to the 55 μm spot size (Visium spots often contain multiple cells), not zero for tissue-mapped spots
- Downstream SpatialScope steps run successfully: cell type identification yields CellTypeLabel_nu10.csv and the decomposition step regenerates single-cell expression (generated_cells.h5ad)
- Regenerated cell-type composition is consistent with the snRNA-seq reference and cell counts per spot do not exceed reasonable nuclei numbers visible in the H&E image

## Limitations

- Paired histological images may be unavailable; alternative single-channel nuclear images (e.g. DAPI) or missing images complicate cell count estimation
- Segmentation only counts nuclei — it does not assign cell types; erroneous segmentation propagates into cell type identification and decomposition
- The downstream Potts smoothness prior assumes neighboring cells share cell type, which may not hold regardless of segmentation quality

## Evidence

- [discussion] Nucleus segmentation serves as a building block to quantify the cell count at each spot.: "Noted that Step 1, Nucleus segmentation serves as a building block to quantify the cell count at each spot."
- [methods] the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [discussion] circumstances may arise where histological images are unavailable, and alternative types of image data, such as single-channel nuclear image data (e.g., DAPI image), are provided instead: "circumstances may arise where histological images are unavailable, and alternative types of image data, such as single-channel nuclear image data (e.g., DAPI image), are provided instead"
- [readme] $ git clone https://github.com/YangLabHKUST/SpatialScope.git: "$ git clone https://github.com/YangLabHKUST/SpatialScope.git"
