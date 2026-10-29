---
name: single-cell-spatial-location-reconstruction
description: Use when a query SC dataset (e.g., .h5ad) lacks coordinates but a matched
  ST reference (SeqFISH/Visium-like) with truth x/y locations and overlapping genes
  is available, and spatially-aware downstream analyses such as co-localization or
  CCC are needed.
when_to_use_negative:
- 'The query SC data and ST reference have a large gap (different tissues/conditions):
  forcing SC assignment to resemble ST can introduce bias, and all methods show a
  performance drop in global reconstruction.'
- Tissue has a high level of repeating elements, which challenges spatial reconstruction
  of SC cells.
- The input already has truth cell locations or is itself an ST dataset — evaluation
  modules (neighbor hit, JSD, Spearman) should be used instead of inference.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellContrast
  role: 'Primary method: contrastive encoder-projector trained on ST reference; encoder
    representations and cosine similarity yield SC-SC distances and projected SC coordinates'
  repo: https://github.com/HKU-BAL/CellContrast
- name: CeLEry
  role: Supervised deep-learning baseline/complement for recovering spatial information
    of SC data, run with default parameters
- name: SageNet
  role: Graph-neural-network baseline producing a predicted cell-cell distance matrix
    projected into 2D pseudo space via MDS
- name: SpatialDM
  role: Moran's R co-localization detection on reconstructed coordinates (n_neighbors=20,
    radial basis kernel l=0.1 or 1e-4, p<=0.05)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:e20bba13c3dae92b5cc2c810ff71a3c6aba37e08938386ad5791428d0aedd5a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - SageNet
  - SpatialDM
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-spatial-location-reconstruction

## Summary

Reconstruct spatial locations of single-cell RNA-seq (SC) cells from a spatial transcriptomics (ST) reference using CellContrast's contrastive encoder, then benchmark or complement the reconstruction with CeLEry and SageNet for downstream cell-type co-localization analysis.

## When to use

Use when a query SC dataset (e.g., .h5ad) lacks coordinates but a matched ST reference (SeqFISH/Visium-like) with truth x/y locations and overlapping genes is available, and spatially-aware downstream analyses such as co-localization or CCC are needed.

## When NOT to use

- The query SC data and ST reference have a large gap (different tissues/conditions): forcing SC assignment to resemble ST can introduce bias, and all methods show a performance drop in global reconstruction.
- Tissue has a high level of repeating elements, which challenges spatial reconstruction of SC cells.
- The input already has truth cell locations or is itself an ST dataset — evaluation modules (neighbor hit, JSD, Spearman) should be used instead of inference.

## Inputs

- ST reference in AnnData .h5ad format with truth locations in .obs[['x','y']] (e.g., SeqFISH embryo 1 L1, 10,150 cells)
- Query SC data in AnnData .h5ad format without coordinates (e.g., mouse SC atlas sample S27, E8.5, 6,460 cells)
- Pretrained CellContrast encoder (Zenodo record 11395995) or trained model checkpoint (e.g., cellContrast_models/epoch_3000.pt)
- Parameter JSON for imaging-based or spot-based ST (parameters/parameters_singleCell. or parameters_spot.)

## Outputs

- spatial_reconstructed_sc.h5ad with .uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]
- Cell-type co-localization counts and Fisher's exact test p-values (e.g., colocalization_results.csv)
- MDS-derived 2D pseudo-space from SC-SC pairwise distances when --enable_denovo is used

## How to apply

Load the ST reference (e.g., SeqFISH embryo L1, 10,150 cells) and the query SC sample (e.g., mouse SC atlas S27, 6,460 cells) and align overlapping genes (350 here). Train CellContrast's encoder-projector model on the ST data with infoNCE loss, discard the projector at inference, and map SC cells onto reference coordinates via a weighted similarity metric combining SC-SC and SC-ST similarities (use --single_cell and parameters_singleCell. for imaging-based ST), saving coordinates as spatial_reconstructed_sc.h5ad. Generate comparative reconstructions with CeLEry (default parameters) and SageNet (MDS projection of predicted cell-cell distances into pseudo space). For co-localization, encode binary cell-type occupancy per location and run SpatialDM Moran's R with n_neighbors=20 and radial basis kernel l=0.1 for CellContrast/CeLEry coordinates (l=1e-4 for the SageNet MDS pseudo-space), calling significant pairs at p<=0.05, then filter to pairs with reference support >1 and test enrichment with Fisher's exact test.

## Related tools

- **CellContrast** (Primary method: contrastive encoder-projector trained on ST reference; encoder representations and cosine similarity yield SC-SC distances and projected SC coordinates) — https://github.com/HKU-BAL/CellContrast
- **CeLEry** (Supervised deep-learning baseline/complement for recovering spatial information of SC data, run with default parameters)
- **SageNet** (Graph-neural-network baseline producing a predicted cell-cell distance matrix projected into 2D pseudo space via MDS)
- **SpatialDM** (Moran's R co-localization detection on reconstructed coordinates (n_neighbors=20, radial basis kernel l=0.1 or 1e-4, p<=0.05))

## Examples

```
python cellContrast.py reconstruct --train_data_path train_ST.h5ad --query_data_path query_sc.h5ad --single_cell --parameter_file parameters/parameters_singleCell. --save_folder cellContrast_models/ --save_path spatial_reconstructed_sc.h5ad
```

## Evaluation signals

- Benchmarking metrics: nearest neighbor hit, Jensen-Shannon distance, and Spearman's rank correlation via `python cellContrast.py eval`
- Reproduced co-localization counts should equal 42 (CellContrast), 42 (CeLEry), and 38 (SageNet) pairs for sample S27
- Fisher's exact test p-values should match 2.793e-9, 1.121e-6, and 7.409e-6 within reasonable numerical tolerance, with 27 co-localizations consistently detected across all three methods
- Output h5ad must contain the new .uns fields: 'cosine sim of rep', 'representation', 'referenced x', 'referenced y', 'de novo x', 'de novo y'
- Detected co-localization pairs should show reference support >1 across the six SeqFISH reference datasets

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, causing overlap and loss of predicted SC-SC distances.
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods.
- MDS-based (de novo) mapping only relies on SC-SC estimated distances and loses the pre-knowledge of the reference's cell distribution; using predicted SC-SC neighbors may reduce similarity to the reference spatial distributions.
- Tangram can generate low probabilities for matched SC-ST pairs and CytoSpace may filter out some SC cells, potentially leading to incomplete reconstructions; SageNet's preprocessing is dataset-specific and memory consumption explodes with gene number.

## Evidence

- [abstract] we introduce CellContrast, a computational method for reconstructing spatial relationships among SC cells from spatial transcriptomics (ST) reference: "we introduce CellContrast, a computational method for reconstructing spatial relationships among SC cells from spatial transcriptomics (ST) reference"
- [methods] During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [discussion] we project the SC samples onto reference ST locations based on the similarities of the learnt representations. This projection employs a weighted similarity metric based on both SC neighbors and ST: "we project the SC samples onto reference ST locations based on the similarities of the learnt representations. This projection employs a weighted similarity metric based on both SC neighbors and ST"
- [discussion] the approach of SC-ST mapping also introduces a potential "cell clumping" issue, where multiple neighbors are projected into the same reference location: "the approach of SC-ST mapping also introduces a potential "cell clumping" issue, where multiple neighbors are projected into the same reference location"
- [readme] what will be newly added in `sptial_reconstructed_sc.h5ad`:`.uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]`: "what will be newly added in `sptial_reconstructed_sc.h5ad`:`.uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]`"
