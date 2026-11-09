---
name: spatially-smoothed-cell-type-identification-potts-model
description: Use when integrating spot-level ST data (Visium or Slide-seq) with an
  scRNA-seq reference to resolve per-cell cell type labels with a spatial smoothness
  prior, e.g. producing CellTypeLabel_nu10.csv for downstream decomposition.
when_to_use_negative:
- When only spot-level cell-type proportions are needed and no single-cell resolution
  is required — deconvolution methods like RCTD, Cell2location, CARD or spatialDWLS
  estimate proportions per spot without resolving individual cells
- When a histological image is unavailable for Visium data and no alternative image
  (e.g. DAPI) can support nucleus segmentation, making per-spot cell counts unreliable
- When the biological context violates the Potts assumption — e.g. tissues where neighboring
  cells are expected to be of different types
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: 'Main implementation: cell type identification module with Poisson model,
    gene-/spot-specific batch effects and Potts smoothness prior nu=10'
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist
  role: Nucleus segmentation on the Visium H&E image to quantify cell count at each
    spot before label assignment
- name: RCTD
  role: Inspiration for singlet/doublet classification replacing nucleus segmentation
    on Slide-seq data; benchmarked against for cell-type deconvolution
  repo: https://github.com/dmcable/spacexr
- name: Cell2location
  role: Existing deconvolution baseline for spot-level cell-type composition, not
    per-cell labeling
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: Spatially informed deconvolution baseline for comparison
  repo: https://github.com/YingMa0107/CARD
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:e9a8126c73cd8cac2e27a95d2da4121dcabea51af4392092949c0f4e094819a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CARD
  - Cell2location
  - Giotto
  - RCTD
  - SpatialScope
  - StarDist
  - Tangram
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Spatially smoothed cell type identification with Potts model prior

## Summary

The SpatialScope cell type identification step assigns cell types to cells within each ST spot by combining a Poisson model with gene- and spot-specific batch effects and a Potts-model spatial smoothness prior (nu=10), so that spatially adjacent cells are encouraged to share cell type labels.

## When to use

Use when integrating spot-level ST data (Visium or Slide-seq) with an scRNA-seq reference to resolve per-cell cell type labels with a spatial smoothness prior, e.g. producing CellTypeLabel_nu10.csv for downstream decomposition.

## When NOT to use

- When only spot-level cell-type proportions are needed and no single-cell resolution is required — deconvolution methods like RCTD, Cell2location, CARD or spatialDWLS estimate proportions per spot without resolving individual cells
- When a histological image is unavailable for Visium data and no alternative image (e.g. DAPI) can support nucleus segmentation, making per-spot cell counts unreliable
- When the biological context violates the Potts assumption — e.g. tissues where neighboring cells are expected to be of different types

## Inputs

- Spot-level ST count matrix (e.g. Visium h5ad, 3813 spots for human heart; Slide-seq V2 cerebellum with 20,141 genes x 8,952 spots after filtering)
- Paired or unpaired scRNA-seq reference (e.g. human heart snRNA-seq atlas with 10 major cell types; mouse cerebellum snRNA-seq, 24,387 genes, 15,609 cells, 19 cell types)
- Per-spot cell counts from nucleus segmentation (sp_adata_ns.h5ad with cell_locations) or singlet/doublet classification for Slide-seq
- H&E or DAPI histological image (for Visium nucleus segmentation)

## Outputs

- CellTypeLabel_nu10.csv (per-cell cell type labels from the Potts-smoothed Poisson fit, nu=10)
- sp_adata.h5ad (annotated ST object with per-cell cell type assignments)
- Inferred cell-type composition per spot and across tissue, usable for subtype localization (e.g. arterial TAGLN-high/LGR6-low SMCs in the heart vascular ROI)

## How to apply

First obtain per-spot cell counts: run SpatialScope nucleus segmentation (StarDist) on the H&E image for Visium, or replace segmentation with RCTD-inspired singlet/doublet classification for Slide-seq (assuming at most two cells per 10 μm spot). Prepare the single-cell reference by total-counts normalization and log(1+x) transformation, then select the top 1000 highly variable genes plus 50 top marker genes per cell type as training genes. Fit a Poisson model with gene- and spot-specific batch effects (exp(gamma_g)) and a Potts spatial smoothness prior at nu=10 to infer each cell's cell type within its spot. For Slide-seq V2 cerebellum this used a paired snRNA-seq reference of 24,387 genes, 15,609 cells and 19 cell types across 8,952 spots. The output is a per-cell label file (CellTypeLabel_nu10.csv) plus an updated sp_adata.h5ad carrying cell_locations, which feeds the gene expression decomposition step.

## Related tools

- **SpatialScope** (Main implementation: cell type identification module with Poisson model, gene-/spot-specific batch effects and Potts smoothness prior nu=10) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist** (Nucleus segmentation on the Visium H&E image to quantify cell count at each spot before label assignment)
- **RCTD** (Inspiration for singlet/doublet classification replacing nucleus segmentation on Slide-seq data; benchmarked against for cell-type deconvolution) — https://github.com/dmcable/spacexr
- **Cell2location** (Existing deconvolution baseline for spot-level cell-type composition, not per-cell labeling) — https://github.com/BayraktarLab/cell2location
- **CARD** (Spatially informed deconvolution baseline for comparison) — https://github.com/YingMa0107/CARD

## Evaluation signals

- Regenerated cell-type composition matches the proportions observed in the snRNA-seq reference (e.g. 10 major heart cell types)
- For Slide-seq reproduction, the labeling covers 10,975 cells across 8,952 spots (6,929 singlet spots and 2,023 doublet spots)
- Spatial coherence: Potts nu=10 labels form spatially contiguous regions consistent with known anatomy, e.g. arterial SMCs (TAGLN-high/LGR6-low) localize to the central ~18-spot vascular structure in the heart data
- Platform effects were corrected (gene-specific exp(gamma_g) division plus depth normalization to the single-cell reference median); uncorrected data gives unsatisfactory identification
- Output files CellTypeLabel_nu10.csv and sp_adata.h5ad exist with per-cell labels matching the cell_locations from segmentation

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Cell type identification results would not be satisfactory if the platform effects are not appropriately corrected
- Paired histological images may be unavailable; alternative image types (e.g. single-channel DAPI) or no images complicate per-spot cell count estimation
- Training the underlying deep generative models is time-consuming, measured in hours or days

## Evidence

- [methods] the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Potts prior and training gene selection: [evidence span withheld — anchor status: not_found]
- [methods] singlet/doublet replacement for Slide-seq: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [discussion] Potts assumption caveat: "the assumption that neighboring cells belong to the same cell type may not always be valid"
- [discussion] platform effect correction requirement: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
