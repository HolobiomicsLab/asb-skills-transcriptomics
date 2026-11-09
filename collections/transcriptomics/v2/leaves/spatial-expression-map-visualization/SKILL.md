---
name: spatial-expression-map-visualization
description: Use when you need to render gene expression as 2D spatial maps keyed
  to spot/cell coordinates — e.g. visually validating CeLEry-predicted 2D coordinates
  against known spatial gradients (ERBB2, ESR1, GATA3) or inspecting VAE-generated
  ST replicates against originals.
when_to_use_negative:
- The query data has no spatial anchoring — plain scRNA-seq cells without recovered
  2D coordinates cannot be placed into a spatial map
- The goal is quantitative benchmarking (pairwise-distance correlations, median Euclidean
  distances) rather than visual inspection of recovered gradients
- The training ST data is a single-cell-resolution platform with a small gene panel,
  where CeLEry itself is suboptimal and maps may not reflect reliable gradients
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CeLEry
  role: Produces the predicted 2D coordinates and VAE-reconstructed spatial gene expression
    matrices that are rendered as spatial maps
  repo: https://github.com/QihuangZhang/CeLEry
- name: Scanpy
  role: Loads and filters the Xenium/Visium data and normalizes expression before
    maps are plotted
- name: PyTorch
  role: Implements the convolutional VAE whose decoder output provides the reconstructed
    spatial gene expression matrices to visualize
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:a1dc0e9a0a0ec28acd9d4f3faac7d925f9bc30b03c6c6d7689e8ab58f680bba4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - CeLEry
  - PyTorch
  - Scanpy
  - Tangram
  - novoSpaRc
  - scikit-learn
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spatial-expression-map-visualization

## Summary

Renders gene expression as 2D spatial maps arranged by spot/cell coordinates — used with CeLEry to plot recovered marker-gene gradients (ERBB2, ESR1, GATA3) in predicted-coordinate space and to compare VAE-generated ST replicates against original expression maps as PNG figures.

## When to use

Use when you need to render gene expression as 2D spatial maps keyed to spot/cell coordinates — e.g. visually validating CeLEry-predicted 2D coordinates against known spatial gradients (ERBB2, ESR1, GATA3) or inspecting VAE-generated ST replicates against originals.

## When NOT to use

- The query data has no spatial anchoring — plain scRNA-seq cells without recovered 2D coordinates cannot be placed into a spatial map
- The goal is quantitative benchmarking (pairwise-distance correlations, median Euclidean distances) rather than visual inspection of recovered gradients
- The training ST data is a single-cell-resolution platform with a small gene panel, where CeLEry itself is suboptimal and maps may not reflect reliable gradients

## Inputs

- Normalized spatial gene expression matrix with spot/cell coordinates (e.g. 10x Xenium breast cancer Replicate 2 cells after low-UMI filtering)
- CeLEry-predicted 2D coordinates for query scRNA-seq cells
- VAE-decoded reconstructed spatial gene expression matrices (augmented ST replicates)
- Original 2D expression matrices of exemplar genes for side-by-side comparison

## Outputs

- PNG spatial expression maps of marker genes (ERBB2, ESR1, GATA3) in predicted-coordinate space
- Comparison figure panels of generated vs. original 2D expression maps for exemplar genes (RPS20, RPS29, CALB1)

## How to apply

First normalize expression with CeLEry's two-step scheme (spot/cell-level log transformation of UMI counts, then gene-level z-scoring across spots). Reshape each gene's 1D expression vector into a 2D matrix arranged by spot spatial coordinates, coding spots without a measured value as 0. For location-recovery validation, map each filtered query cell's expression (e.g. ERBB2, ESR1, GATA3 in the 10x Xenium breast cancer data) into its CeLEry-predicted 2D coordinate space and render the recovered spatial pattern as a PNG map. For data-augmentation inspection, decode VAE-sampled embeddings into reconstructed spatial gene expression matrices and save a comparison panel of generated versus original 2D maps for exemplar genes such as RPS20, RPS29, and CALB1. Judge correctness by whether predicted-coordinate maps reproduce the known spatial gradients of the reference figure (Fig. 10) — expected when pairwise-distance correlations reach ~0.74 (single-cell), 0.46 (artificial Visium), or 0.51 (TESLA) — and by generated replicates resembling the original patterns while varying among replicates.

## Related tools

- **CeLEry** (Produces the predicted 2D coordinates and VAE-reconstructed spatial gene expression matrices that are rendered as spatial maps) — https://github.com/QihuangZhang/CeLEry
- **Scanpy** (Loads and filters the Xenium/Visium data and normalizes expression before maps are plotted)
- **PyTorch** (Implements the convolutional VAE whose decoder output provides the reconstructed spatial gene expression matrices to visualize)

## Evaluation signals

- Predicted-coordinate maps of ERBB2, ESR1, and GATA3 reproduce the spatial gradients shown in Fig. 10 of the paper
- Underlying quantitative check: CeLEry pairwise-distance correlations approximate 0.74 (single-cell), 0.46 (artificial Visium), and 0.51 (TESLA) on the Xenium breast cancer data
- VAE-generated replicates show 2D gene expression patterns similar to the original ST data while varying among the three replicates
- Spots/cells without a measured expression value are coded as 0 in the reshaped 2D matrix before plotting
- Figures are saved as PNG files (marker-gene maps; generated-vs-original comparison panels)

## Limitations

- Visual map inspection is qualitative; the paper's validation is quantitative (pairwise-distance correlations), so maps should be paired with correlation metrics
- It is not yet clear how many spatially variable gradient genes exist in tissues, limiting which genes can serve as visual validation markers
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels, so maps from such data may not show trustworthy gradients

## Evidence

- [methods] we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training: "we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training"
- [methods] Reshape each gene's 1D expression vector into a 2D matrix arranged by spot spatial coordinates, coding spots without a measured value as 0.: [evidence span withheld — anchor status: not_found]
- [readme] A deep neural network is trained to learn the relationship between the spotwise gene expression and location information, minimizing the loss functions: "A deep neural network is trained to learn the relationship between the spotwise gene expression and location information, minimizing the loss functions"
- [readme] CeLEry can infer the spatial origins of cells in scRNA-seq at multiple levels, including 2D location as well as the spatial domain or tissue layer of a cell.: "CeLEry can infer the spatial origins of cells in scRNA-seq at multiple levels, including 2D location as well as the spatial domain or tissue layer of a cell."
- [results] Visualize the recovered spatial expression patterns of ERBB2, ESR1, and GATA3 in the predicted-coordinate space as PNG maps.: [evidence span withheld — anchor status: not_found]
