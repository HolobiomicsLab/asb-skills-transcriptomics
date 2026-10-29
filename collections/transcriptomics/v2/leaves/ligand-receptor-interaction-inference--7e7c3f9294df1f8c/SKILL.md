---
name: ligand-receptor-interaction-inference
description: Use when analyzing cell-cell communication in SC data lacking native
  spatial coordinates but with a matched ST reference (e.g., Stereo-seq E9.5), to
  compare gene-expression-only vs spatially-informed LRI detection.
when_to_use_negative:
- Query SC data and ST reference have a large gap/mismatch — forcing SC assignment
  to resemble the ST can introduce bias, and MDS-based mapping loses the reference's
  cell-distribution pre-knowledge
- Target tissues have a high level of repeating elements, which poses a challenge
  for spatial reconstruction of SC cells
- You need a strict gold-standard CCC validation — the reference CCCs here were identified
  through the same spatially-reconstructed approach, introducing circularity
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: CellContrast
  role: Reconstructs spatial relationships (2D pseudo coordinates) of SC cells from
    an ST reference via contrastive learning; supplies the coordinates spatialDM needs
  repo: https://github.com/HKU-BAL/CellContrast
- name: CellChat
  role: Baseline gene-expression-only global ligand-receptor interaction inference
    using the recommended pipeline with 'type'='truncatedMean'
- name: spatialDM
  role: Detects significant LRIs and local cell-cell communications on the reconstructed
    spatial coordinates (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:9b5decf55c712e1cf8b0453872a37dac0817a1b3dbebd413c5a9e7cacaf15541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellChat
  - CellContrast
  - CellPhoneDB
  - Connectome
  - Cytotalk
  - LIANA
  - NATMI
  - SingleCellSignalR
  - iTALK
  - spatialDM
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ligand-receptor-interaction-inference

## Summary

Infers ligand-receptor interactions (LRIs) and local cell-cell communications from single-cell RNA-seq data by incorporating CellContrast-reconstructed spatial locations, using CellChat for gene-expression-only global inference and spatialDM for spatially-informed detection, thereby reducing false positives.

## When to use

Use when analyzing cell-cell communication in SC data lacking native spatial coordinates but with a matched ST reference (e.g., Stereo-seq E9.5), to compare gene-expression-only vs spatially-informed LRI detection.

## When NOT to use

- Query SC data and ST reference have a large gap/mismatch — forcing SC assignment to resemble the ST can introduce bias, and MDS-based mapping loses the reference's cell-distribution pre-knowledge
- Target tissues have a high level of repeating elements, which poses a challenge for spatial reconstruction of SC cells
- You need a strict gold-standard CCC validation — the reference CCCs here were identified through the same spatially-reconstructed approach, introducing circularity

## Inputs

- Query single-cell RNA-seq dataset (e.g., scRNA-S1, GSE87038) in AnnData .h5ad format, scran-normalized and log-transformed
- Spatial transcriptomics reference dataset with truth cell locations (e.g., Stereo-seq E9.5 E2S1) in .h5ad format with .obs[['x','y']]
- Overlapping highly variable genes between SC and ST samples (e.g., 444 HVGs)
- Trained CellContrast encoder model checkpoint (e.g., epoch_3000.pt)
- Ligand-receptor interaction database used by CellChat

## Outputs

- Spatially reconstructed SC AnnData (.h5ad) with cosine similarities, latent representations, and de novo MDS coordinates (.uns['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y'])
- Global LRI list from CellChat (gene-expression-only inference)
- Significant LRIs and local CCCs from spatialDM on reconstructed coordinates with p values ≤ 0.05
- Reference support values per LRI (number of reference datasets with the same signal) for comparison
- Per-cell significant interaction table, e.g., 45 significant IGF2-IGF2R cells with normalized expressions and reconstructed coordinates

## How to apply

Train the CellContrast encoder-projector contrastive model on the ST reference (e.g., Stereo-seq E9.5 E2S1) using overlapping highly variable genes with infoNCE loss (temperature 0.05, batch size 64, spot-based positive pairs from k=20 nearest spatial neighbors, SGD momentum 0.9, weight decay 5e-4, initial LR 0.1 with cosine decay, 3,000 epochs). At inference, discard the projector, feed scran log-normalized SC expressions through the encoder, compute pairwise SC-SC cosine similarities, and project dissimilarities to 2D pseudo space via MDS. Run CellChat with 'type'='truncatedMean' for gene-expression-only global LRI detection, and spatialDM (n_neighbors=20, radial basis kernel l=0.1, p value ≤ 0.05) on the reconstructed coordinates for spatially-informed LRIs and local CCCs. Define reference CCCs by applying spatialDM to the ST reference datasets and compute reference support (number of reference datasets with the same signal) for both results, restricted to overlapping tissues such as forebrain, hindbrain, heart, and liver. The rationale is that reconstructed cell locations enable spatial validation of interactions, mitigating false positives from expression-only inference.

## Related tools

- **CellContrast** (Reconstructs spatial relationships (2D pseudo coordinates) of SC cells from an ST reference via contrastive learning; supplies the coordinates spatialDM needs) — https://github.com/HKU-BAL/CellContrast
- **CellChat** (Baseline gene-expression-only global ligand-receptor interaction inference using the recommended pipeline with 'type'='truncatedMean')
- **spatialDM** (Detects significant LRIs and local cell-cell communications on the reconstructed spatial coordinates (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05))

## Examples

```
python cellContrast.py reconstruct --train_data_path train_ST.h5ad --query_data_path query_sc.h5ad --parameter_file parameters/parameters_spot. --save_folder cellContrast_models/ --enable_denovo --save_path spatial_reconstructed_sc.h5ad
```

## Evaluation signals

- spatialDM on reconstructed locations yields a greater portion of LRIs with reference support >1 than CellChat (e.g., 0.28 vs 0.16 at reference support 5)
- Reduction in LRIs with no reference support compared to CellChat (e.g., 0.12 vs 0.30)
- Significant IGF2-IGF2R interactions detected in ~45 cells, primarily in heart and liver, at short reconstructed distances
- Reconstructed SC-SC cosine similarities correlate inversely with spatial distance (higher cosine similarity indicates shorter spatial distance)
- Reference CCCs restricted to overlapping tissues (forebrain, hindbrain, heart, liver) show consistent support across the five Stereo-seq E9.5 reference datasets

## Limitations

- CCC validation is circular: reference CCCs were identified through the same approach with spatially reconstructed SC data
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods
- MDS-based de novo mapping relies only on SC-SC estimated distances and loses the pre-knowledge of the reference's cell distribution
- Tissues with a high level of repeating elements pose a challenge for spatial reconstruction of SC cells

## Evidence

- [results] Using CellContrast-reconstructed cell positions, spatialDM consistently identified a greater portion of LRIs with reference support >1 (e.g., 0.28 vs 0.16 for reference support 5): [evidence span withheld — anchor status: not_found]
- [intro] During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [discussion] We employed CellContrast for CCC analysis on E9.5 mouse embryo SC data and compared CCCs inferred solely from gene expression: "We employed CellContrast for CCC analysis on E9.5 mouse embryo SC data and compared CCCs inferred solely from gene expression (with CellChat) to those incorporating spatial information (with"
- [discussion] because the reference CCCs were identified through the same approach with the spatial reconstructed SC data, the results could be advanced to it to some extent: "because the reference CCCs were identified through the same approach with the spatial reconstructed SC data, the results could be advanced to it to some extent"
- [readme] what will be newly added in `sptial_reconstructed_sc.h5ad`:`.uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]`: "what will be newly added in `sptial_reconstructed_sc.h5ad`:`.uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]`"
