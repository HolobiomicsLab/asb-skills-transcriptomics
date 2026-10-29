---
name: st-reference-contrastive-model-inference
description: Use when SC cells lack coordinates but an ST reference (e.g. SeqFISH
  embryo ST h5ad with .obs[['x','y']]) exists, and reconstructed per-cell spatial
  locations are needed for downstream co-localization or cell-cell communication analyses.
when_to_use_negative:
- 'There is a large gap between the query SC data and the ST reference: forcing SC
  assignment to resemble the ST could introduce more bias, and MDS de novo mode loses
  the pre-knowledge of the reference''s cell distribution'
- The SC data already contains truth spatial coordinates (use them directly as reference
  patterns instead)
- Spot-based ST parameters are mistakenly used for imaging-based single-cell ST references
  — the predefined --single_cell parameter set must be adopted instead
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_0203
tools:
- name: CellContrast
  role: 'Core method: contrastive encoder-projector model trained on the ST reference
    and applied at inference to map SC cells onto reference ST locations (weighted
    SC-SC/SC-ST similarity metric) or an MDS pseudo-space'
  repo: https://github.com/HKU-BAL/CellContrast
- name: CeLEry
  role: Comparative supervised deep neural network method used to generate alternative
    reconstructions (default parameters) of the same SC sample for validation
- name: SageNet
  role: Comparative graph neural network method producing an estimated cell-cell distance
    matrix that is MDS-projected into pseudo space for cross-method validation
- name: SpatialDM
  role: Downstream co-localization detection (Moran's R, n_neighbors=20, radial kernel
    l=0.1, p<=0.05) on the reconstructed coordinates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:781c7e7c8ffe51a13a464386935fabd93e41363d8d0c9b7b09a7681ae4b8b74c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - SageNet
  - SpatialDM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ST-reference contrastive model inference

## Summary

Apply a contrastively trained CellContrast encoder to spatially reconstruct single-cell RNA-seq (SC) cells by projecting their latent representations onto a spatial transcriptomics (ST) reference with truth locations, or onto an MDS pseudo-space from SC-SC cosine-similarity distances.

## When to use

Use when SC cells lack coordinates but an ST reference (e.g. SeqFISH embryo ST h5ad with .obs[['x','y']]) exists, and reconstructed per-cell spatial locations are needed for downstream co-localization or cell-cell communication analyses.

## When NOT to use

- There is a large gap between the query SC data and the ST reference: forcing SC assignment to resemble the ST could introduce more bias, and MDS de novo mode loses the pre-knowledge of the reference's cell distribution
- The SC data already contains truth spatial coordinates (use them directly as reference patterns instead)
- Spot-based ST parameters are mistakenly used for imaging-based single-cell ST references — the predefined --single_cell parameter set must be adopted instead

## Inputs

- ST reference h5ad (AnnData) with truth locations in .obs[['x','y']] (e.g. SeqFISH embryo 1 L1 ST reference, 10,150 cells)
- Query SC h5ad (e.g. mouse SC atlas sample S27, 6,460 cells) with genes aligned to the ST reference (e.g. 350 overlapping genes)
- Trained CellContrast model folder (e.g. cellContrast_models/epoch_3000.pt) or pre-trained encoder from Zenodo
- Parameter JSON file (parameters/parameters_singleCell. for imaging-based ST, parameters/parameters_spot. for spot-based ST)

## Outputs

- spatial_reconstructed_sc.h5ad (AnnData) with .uns keys: 'cosine sim of rep', 'representation', 'referenced x', 'referenced y', 'de novo x', 'de novo y'
- Reconstructed per-cell spatial coordinates for downstream SpatialDM Moran's R co-localization analysis
- Optional benchmark metrics (nearest neighbor hit, Jensen-Shannon distance, Spearman's rank correlation) via the eval module

## How to apply

Train (or load from Zenodo, doi 10.5281/zenodo.11395995) the encoder-projector model on the ST reference with infoNCE loss, then discard the projector and use encoder representations for spatial reconstruction via cosine similarity. For imaging-based ST references (e.g. single-cell-resolution SeqFISH), set the --single_cell flag and use parameters/parameters_singleCell.; for spot-based data use parameters_spot.. Run the inference/reconstruct module with the SC query h5ad (e.g. mouse SC atlas sample S27, 6,460 cells) to obtain reconstructed coordinates saved as spatial_reconstructed_sc.h5ad. Optionally add --enable_denovo to run MDS on SC-SC pairwise distances into a 2D pseudo-space, which is more robust when there is a large gap between SC and ST but loses knowledge of the reference cell distribution. Downstream, encode binary cell-type occupancy and run SpatialDM Moran's R (n_neighbors=20, radial kernel l=0.1, significance at p<=0.05) to validate that reference co-localization patterns are captured.

## Related tools

- **CellContrast** (Core method: contrastive encoder-projector model trained on the ST reference and applied at inference to map SC cells onto reference ST locations (weighted SC-SC/SC-ST similarity metric) or an MDS pseudo-space) — https://github.com/HKU-BAL/CellContrast
- **CeLEry** (Comparative supervised deep neural network method used to generate alternative reconstructions (default parameters) of the same SC sample for validation)
- **SageNet** (Comparative graph neural network method producing an estimated cell-cell distance matrix that is MDS-projected into pseudo space for cross-method validation)
- **SpatialDM** (Downstream co-localization detection (Moran's R, n_neighbors=20, radial kernel l=0.1, p<=0.05) on the reconstructed coordinates)

## Examples

```
python cellContrast.py inference --ref_data_path train_ST.h5ad --query_data_path query_sc.h5ad --model_folder cellContrast_models/ --parameter_file parameters/parameters_singleCell. --save_path spatial_reconstructed_sc.h5ad
```

## Evaluation signals

- Output h5ad contains all expected .uns fields: 'cosine sim of rep', 'representation', 'referenced x', 'referenced y', 'de novo x', 'de novo y'
- Number of reconstructed cells equals the query SC cell count (e.g. 6,460 for mouse SC atlas sample S27)
- Downstream SpatialDM co-localization counts on the reconstruction match the expected values (CellContrast 42 pairs, Fisher's exact p = 2.793e-9), with ~27 co-localizations consistently detected across CellContrast, CeLEry (42, p = 1.121e-6), and SageNet (38, p = 7.409e-6)
- Benchmark metrics from the eval module (nearest neighbor hit, Jensen-Shannon distance, Spearman's rank correlation) show reasonable spatial reconstruction quality
- Genes aligned to the reference overlap correctly (e.g. 350 overlapping genes between SeqFISH ST and SC atlas)

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, causing overlap and loss of predicted SC-SC distances
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods
- MDS-based de novo mapping only relies on SC-SC estimated distances and loses the pre-knowledge of the reference's cell distribution
- Tissues with a high level of repeating elements could pose a challenge for spatial reconstruction of SC cells

## Evidence

- [discussion] we project the SC samples onto reference ST locations based on the similarities of the learnt representations: "we project the SC samples onto reference ST locations based on the similarities of the learnt representations"
- [discussion] mapped to location coordinates using ST reference or MDS algorithm: "The spatially related representations of SC cells can be further mapped to location coordinates using ST reference or MDS algorithm."
- [intro] During inference, we discarded the projector: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction"
- [readme] inference command output fields: "what will be newly added in `sptial_reconstructed_sc.h5ad`:`.uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]`"
- [readme] single_cell flag for imaging-based ST: "Adopt the predefined parameters for imaging-based ST data by setting `--single_cell`."
