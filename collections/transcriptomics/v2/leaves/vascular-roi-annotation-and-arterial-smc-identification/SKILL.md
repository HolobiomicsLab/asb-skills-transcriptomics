---
name: vascular-roi-annotation-and-arterial-smc-identification
description: Use when a Visium ST section contains a vascular region of interest and
  you must assign SMC subtype identity (arterial vs venous) using an unpaired snRNA-seq
  atlas reference and single-cell-resolution decomposition output.
when_to_use_negative:
- The reference atlas does not define separate arterial and venous SMC subtypes, so
  the TAGLN/LGR6 signature cannot discriminate them
- The goal is spot-level deconvolution proportions only, not single-cell-resolution
  subtype localization — RCTD, Cell2location or CARD suffice
- The tissue lacks a histological image and only DAPI or no image is available, complicating
  nucleus segmentation and per-spot cell-count estimation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Deep generative pipeline (StarDist segmentation, cell type identification
    with Potts nu=10, Langevin-dynamics decomposition) producing single-cell expression
    in the vascular ROI
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Giotto
  role: spatCellCellcom on a Delaunay-triangulation spatial network over the SpatialScope
    output to detect significant EC-SMC ligand-receptor interactions supporting the
    arterial annotation
- name: StarDist
  role: Nucleus segmentation backend of SpatialScope Step 1, counting cells per Visium
    spot on the heart H&E image
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:781a14f3922ffc62b7728fbeae6868413e1517ce10373c1813401114be039d96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - Giotto
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Vascular ROI annotation and arterial SMC identification in heart Visium data

## Summary

Identifies the arterial vs venous identity of smooth muscle cells within a vascular region of interest on human heart Visium data by decomposing spots to single cells with SpatialScope and checking the TAGLN-high/LGR6-low signature against the snRNA-seq atlas-defined arterial SMC subtype.

## When to use

Use when a Visium ST section contains a vascular region of interest and you must assign SMC subtype identity (arterial vs venous) using an unpaired snRNA-seq atlas reference and single-cell-resolution decomposition output.

## When NOT to use

- The reference atlas does not define separate arterial and venous SMC subtypes, so the TAGLN/LGR6 signature cannot discriminate them
- The goal is spot-level deconvolution proportions only, not single-cell-resolution subtype localization — RCTD, Cell2location or CARD suffice
- The tissue lacks a histological image and only DAPI or no image is available, complicating nucleus segmentation and per-spot cell-count estimation

## Inputs

- Visium Human Heart Dataset H&E image (~10000x10000 raw) and spot-level count matrix (3813 spots after filtering UMI < 100)
- Human heart snRNA-seq atlas reference (10 major cell types)
- SpatialScope pretrained score-based checkpoint (model_5000.pt)
- SpatialScope outputs: sp_adata_ns.h5ad with cell_locations, CellTypeLabel_nu10.csv, sp_adata.h5ad, generated_cells.h5ad

## Outputs

- Per-cell cell-type labels for the 331-spot vascular ROI with arterial SMCs flagged by TAGLN-high/LGR6-low expression
- Spatial map of arterial SMC localization (central vascular structure, ~18 spots)
- Filtered EC–SMC ligand-receptor pair list (e.g., JAG1-NOTCH3, DLL4-NOTCH3, SERPINE2-LRP1, RTN4-LINGO1) from Giotto spatCellCellcom

## How to apply

Run SpatialScope's three steps — StarDist nucleus segmentation on the H&E image, cell type identification with the human heart snRNA-seq atlas (10 major cell types, top 1000 HVGs + 50 markers/cell type, Poisson model with gene- and spot-specific batch effects and Potts prior nu=10), and gene expression decomposition via the pretrained score-based model with annealed Langevin dynamics — to obtain posterior-mean single-cell expression in generated_cells.h5ad. Restrict to the vascular ROI (331 spots in the heart sample) and classify decomposed SMCs as arterial when they express TAGLN at high levels while LGR6 is low, matching the atlas-defined arterial SMC signature. Optionally validate cell–cell communication context by running Giotto's spatCellCellcom on a Delaunay-triangulation spatial network with 876 ligands/receptors, filtering pairs at p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 and rec_expr > 0.5. Correct gene-specific batch effects (divide counts by exp(gamma_g)) and normalize to the reference median total counts before decomposition, since uncorrected platform effects yield unsatisfactory results.

## Related tools

- **SpatialScope** (Deep generative pipeline (StarDist segmentation, cell type identification with Potts nu=10, Langevin-dynamics decomposition) producing single-cell expression in the vascular ROI) — https://github.com/YangLabHKUST/SpatialScope
- **Giotto** (spatCellCellcom on a Delaunay-triangulation spatial network over the SpatialScope output to detect significant EC-SMC ligand-receptor interactions supporting the arterial annotation)
- **StarDist** (Nucleus segmentation backend of SpatialScope Step 1, counting cells per Visium spot on the heart H&E image)

## Examples

```
import scanpy as sc; cells = sc.read_h5ad('generated_cells.h5ad'); arterial = cells[(cells[:, 'TAGLN'].X > 0) & (cells[:, 'LGR6'].X == 0)]
```

## Evaluation signals

- Decomposed SMCs in the vascular ROI show TAGLN-high/LGR6-low expression, matching the atlas-defined arterial SMC signature (not the venous signature)
- Arterial SMCs localize to the central vascular structure (~18 spots) within the 331-spot ROI, consistent with the visible vasculature on H&E
- Regenerated cell-type composition is consistent with the 10-major-cell-type snRNA-seq atlas reference proportions
- Giotto results recover known EC-SMC pairs JAG1-NOTCH3, DLL4-NOTCH3, SERPINE2-LRP1 and RTN4-LINGO1 after filtering (p.adj < 0.25, abs(log2fc) > 0.1, lig_nr/rec_nr > 10, lig_expr/rec_expr > 0.5)
- Cell type identification log (CellTypeLabel_nu10.csv) shows converged Poisson fit with gene- and spot-specific batch effects applied before decomposition

## Limitations

- The Potts smoothness prior assumes neighboring cells belong to the same cell type, which may not hold at vessel boundaries where endothelial and SMC cells interleave
- Training the score-based generative model is time-consuming (measured in hours or days) and no software version numbers or computational environment details are reported
- Platform effects must be appropriately corrected (divide by exp(gamma_g), normalize to reference median) or cell type identification and decomposition results will not be satisfactory

## Evidence

- [methods] Heart spots retained after filtering: "After removing spots that did not map to the tissue region or with total UMI counts less than 100, 3813 spots were left"
- [methods] Paired heart snRNA-seq atlas: "we used a paired human heart snRNA-seq atlas that consists of 10 major cell types"
- [methods] Giotto ligand-receptor thresholds: "we selected top and reliable ligand-receptor signaling with the following threshold: p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 & rec_expr > 0.5"
- [methods] SpatialScope three-step pipeline: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [discussion] Platform effects required for satisfactory results: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
- [readme] SpatialScope repo description: "A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models"
