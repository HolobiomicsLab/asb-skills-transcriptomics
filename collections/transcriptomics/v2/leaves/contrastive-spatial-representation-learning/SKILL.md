---
name: contrastive-spatial-representation-learning
description: Use when you have unspatial single-cell RNA-seq data (e.g., scRNA-S1,
  GSE87038) plus a matching ST reference with truth coordinates (e.g., Stereo-seq
  E9.5 E2S1 .h5ad with obs x/y) and need pseudo-spatial coordinates or SC-SC distance
  estimates for downstream spatial analyses.
when_to_use_negative:
- Tissues with a high level of repeating elements, where spatial reconstruction of
  SC cells is challenged
- Large gaps between ST reference and SC query with forced assignment to ST coordinates
  — MDS-based de novo mapping may be more appropriate, or mapping may introduce bias
- When a mapping that filters cells (e.g., CytoSpace) is acceptable; this skill targets
  complete reconstructions without cell clumping or filtered-out cells
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellContrast
  role: 'Core tool: trains the contrastive encoder-projector model and performs spatial
    inference (train/eval/inference/reconstruct modules) to obtain SC spatial representations
    and coordinates'
  repo: https://github.com/HKU-BAL/CellContrast
- name: spatialDM
  role: 'Downstream tool: detects ligand-receptor interactions and local CCCs on the
    reconstructed SC coordinates (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05)'
- name: CellChat
  role: 'Baseline tool: expression-only CCC inference (type=''truncatedMean'') used
    to demonstrate false-positive reduction when spatial information is added'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:ea27198f793f97596297067ef09899f35ba4874826dc167625061ea4cecd85de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellChat
  - CellContrast
  - spatialDM
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# contrastive-spatial-representation-learning

## Summary

Reconstruct spatial relationships among single-cell RNA-seq cells by training a CellContrast encoder-projector contrastive model on a spatial transcriptomics reference, then applying the encoder to SC data so that cosine similarity of latent representations reflects physical proximity. Recovered cell locations empower downstream discoveries (e.g., cell-cell communication with spatialDM) while mitigating false positives from gene-expression-only inference.

## When to use

Use when you have unspatial single-cell RNA-seq data (e.g., scRNA-S1, GSE87038) plus a matching ST reference with truth coordinates (e.g., Stereo-seq E9.5 E2S1 .h5ad with obs x/y) and need pseudo-spatial coordinates or SC-SC distance estimates for downstream spatial analyses.

## When NOT to use

- Tissues with a high level of repeating elements, where spatial reconstruction of SC cells is challenged
- Large gaps between ST reference and SC query with forced assignment to ST coordinates — MDS-based de novo mapping may be more appropriate, or mapping may introduce bias
- When a mapping that filters cells (e.g., CytoSpace) is acceptable; this skill targets complete reconstructions without cell clumping or filtered-out cells

## Inputs

- ST reference AnnData (.h5ad) with truth coordinates in .obs[['x','y']] (e.g., Stereo-seq E9.5 E2S1)
- Query SC AnnData (.h5ad) with scran-normalized, log-transformed gene expressions (e.g., scRNA-S1, GSE87038)
- Parameter file (e.g., parameters/parameters_spot. for sequencing-based ST, parameters_singleCell. for imaging-based ST)
- Trained model checkpoint (epoch_3000.pt)

## Outputs

- Spatially reconstructed SC AnnData (.h5ad) with .uns fields: cosine sim of rep, representation, referenced x/y, de novo x/y
- Pairwise SC-SC cosine similarity (cell-cell distance) matrix
- 2D pseudo-space coordinates (MDS or reference-projection) per SC cell
- Downstream CCC results (e.g., significant IGF2-IGF2R interactions with 45 cells, reference-support distributions of LRIs)

## How to apply

Train the CellContrast encoder-projector model on the ST reference using infoNCE loss with spot-based positive pairs drawn from the k=20 nearest spatial neighbors, using the 444 highly variable genes overlapping the query SC data, with SGD (momentum 0.9, weight_decay 5e-4), initial learning rate 0.1 with cosine decay, batch size 64, for 3,000 epochs. At inference, discard the projector and feed scran log-normalized SC expressions through the encoder, computing pairwise SC-SC cosine similarities as cell-cell distance estimates; project these dissimilarities to 2D pseudo space with MDS (or map to reference ST coordinates via the weighted similarity metric). Feed the reconstructed coordinates into downstream analyses such as spatialDM ligand-receptor interaction detection (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05) and compare against expression-only tools like CellChat to verify reduced false positives. Benchmarks are judged with nearest neighbor hit, Jensen-Shannon distance, and Spearman's rank correlation against truth locations.

## Related tools

- **CellContrast** (Core tool: trains the contrastive encoder-projector model and performs spatial inference (train/eval/inference/reconstruct modules) to obtain SC spatial representations and coordinates) — https://github.com/HKU-BAL/CellContrast
- **spatialDM** (Downstream tool: detects ligand-receptor interactions and local CCCs on the reconstructed SC coordinates (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05))
- **CellChat** (Baseline tool: expression-only CCC inference (type='truncatedMean') used to demonstrate false-positive reduction when spatial information is added)

## Examples

```
python cellContrast.py reconstruct --train_data_path train_ST.h5ad --query_data_path query_sc.h5ad --parameter_file parameters/parameters_spot. --save_folder cellContrast_models/ --enable_denovo --save_path spatial_reconstructed_sc.h5ad
```

## Evaluation signals

- Benchmark metrics: nearest neighbor hit, Jensen-Shannon distance, and Spearman's rank correlation of reconstructed vs. truth locations (result.csv from eval module)
- CCC validation: spatialDM on reconstructed locations yields a greater portion of LRIs with reference support >1 (0.28 vs 0.16 at reference support 5) than CellChat
- Reduction of unsupported local CCCs (no-reference-support LRIs drop from 0.30 with CellChat to 0.12 with spatialDM on reconstructed coordinates)
- Case check: 45 cells exhibit significant IGF2-IGF2R signals at short reconstructed distances, primarily in heart and liver

## Limitations

- SC-ST mapping can cause 'cell clumping' where multiple neighbors project into the same reference location, losing predicted SC-SC distances
- Large gaps between ST and SC samples degrade global reconstruction across all methods, and forcing SC to resemble ST can introduce bias; MDS is more robust but loses the reference's cell distribution prior
- A certain level of mismatch may exist between query SC data and ST reference due to limited ST resources; CCC validation carries circularity since reference CCCs were identified via the same reconstructed-SC approach
- Tissues with a high level of repeating elements pose a challenge for spatial reconstruction

## Evidence

- [intro] During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [discussion] Contrastive pairs from ST truth locations: "constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss"
- [discussion] MDS mapping robustness trade-off: "MDS only relies on SC-SC estimated distances, thus it is more robust when there is a large gap between SC and ST reference, but it loses the pre-knowledge of reference"
- [readme] README: reconstruct module usage: "CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']])"
- [readme] README: inference outputs: "what will be newly added in `sptial_reconstructed_sc.h5ad`:`.uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]`"
