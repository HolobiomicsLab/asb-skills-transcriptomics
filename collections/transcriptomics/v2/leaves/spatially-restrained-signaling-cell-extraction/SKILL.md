---
name: spatially-restrained-signaling-cell-extraction
description: Use when significant ligand-receptor interactions detected on spatially
  reconstructed SC data (spatialDM p ≤ 0.05) must be traced to specific cells and
  tissues, e.g. recovering the heart/liver cells carrying IGF2-IGF2R signals in the
  E9.5 mouse embryo scRNA-S1 sample.
when_to_use_negative:
- When truth single-cell spatial locations are already available — apply spatialDM
  directly instead of reconstructing coordinates first
- When only gene-expression-based global interaction inference is needed without cell-level
  spatial localization (CellChat alone suffices)
- When the SC query has a large gap from the ST reference — forcing SC cells to resemble
  ST can introduce bias and MDS-based de novo mapping loses the reference's cell distribution
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellContrast
  role: Reconstructs spatial relationships of SC cells via contrastive encoder representations;
    its inference module with --enable_denovo produces the reconstructed coordinates
    used for spatially restrained signaling extraction
  repo: https://github.com/HKU-BAL/CellContrast
- name: spatialDM
  role: Detects local cell-cell communications and significant ligand-receptor interactions
    on the reconstructed coordinates (n_neighbors=20, radial basis kernel l=0.1, p
    ≤ 0.05), identifying which cells carry the signal
- name: CellChat
  role: Expression-only baseline for ligand-receptor inference ('type' = 'truncatedMean')
    used to benchmark reference support and false-positive reduction against the spatially
    restrained results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:4946dfb54c59e1a6c5fbcfa90d96722a6cc40be8a26af57ebae0bbb411d89087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellChat
  - CellContrast
  - spatialDM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spatially-restrained-signaling-cell-extraction

## Summary

Localizes ligand-receptor signaling events to individual cells by running spatially aware CCC detection (spatialDM) on CellContrast-reconstructed single-cell coordinates, then extracting the significant cells (with normalized expression and reconstructed x/y coordinates) for a target interaction such as IGF2-IGF2R.

## When to use

Use when significant ligand-receptor interactions detected on spatially reconstructed SC data (spatialDM p ≤ 0.05) must be traced to specific cells and tissues, e.g. recovering the heart/liver cells carrying IGF2-IGF2R signals in the E9.5 mouse embryo scRNA-S1 sample.

## When NOT to use

- When truth single-cell spatial locations are already available — apply spatialDM directly instead of reconstructing coordinates first
- When only gene-expression-based global interaction inference is needed without cell-level spatial localization (CellChat alone suffices)
- When the SC query has a large gap from the ST reference — forcing SC cells to resemble ST can introduce bias and MDS-based de novo mapping loses the reference's cell distribution

## Inputs

- Query single-cell RNA-seq AnnData (e.g. scRNA-S1, GSE87038) with scran-normalized log-transformed expression
- Trained CellContrast encoder model checkpoint (e.g. epoch_3000.pt) from a Stereo-seq E9.5 reference ST sample
- Spatially reconstructed SC AnnData containing 2D pseudo-space coordinates ('de novo x', 'de novo y')
- Ligand-receptor pair(s) of interest (e.g. IGF2-IGF2R) for targeted cell extraction

## Outputs

- Set of significant cells for the target LRI (expected 45 for IGF2-IGF2R) with normalized expression values
- Reconstructed 2D coordinates of the significant cells for distance/tissue verification
- Local CCC calls from spatialDM (n_neighbors=20, l=0.1, p ≤ 0.05) on the reconstructed space

## How to apply

Train the CellContrast encoder-projector model on a Stereo-seq E9.5 (E2S1) reference using overlapping highly variable genes, then discard the projector and pass scran-normalized, log-transformed scRNA-S1 expression through the encoder to compute SC-SC cosine similarities; project dissimilarities to 2D pseudo space with MDS (--enable_denovo), yielding a reconstructed AnnData with de novo coordinates. Run spatialDM on the reconstructed locations (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05) to detect local cell-cell communications with spatial restraint. For the ligand-receptor pair of interest (e.g. IGF2-IGF2R), extract the cells flagged significant by spatialDM together with their normalized expressions and reconstructed coordinates. Judge correctness by checking that significant cells cluster at short reconstructed distances and fall in the expected tissues (45 cells, primarily heart and liver for IGF2-IGF2R), and that spatially informed calls show higher reference support than expression-only CellChat calls.

## Related tools

- **CellContrast** (Reconstructs spatial relationships of SC cells via contrastive encoder representations; its inference module with --enable_denovo produces the reconstructed coordinates used for spatially restrained signaling extraction) — https://github.com/HKU-BAL/CellContrast
- **spatialDM** (Detects local cell-cell communications and significant ligand-receptor interactions on the reconstructed coordinates (n_neighbors=20, radial basis kernel l=0.1, p ≤ 0.05), identifying which cells carry the signal)
- **CellChat** (Expression-only baseline for ligand-receptor inference ('type' = 'truncatedMean') used to benchmark reference support and false-positive reduction against the spatially restrained results)

## Examples

```
python cellContrast.py inference --ref_data_path train_ST.h5ad --query_data_path query_sc.h5ad --model_folder cellContrast_models/ --enable_denovo --save_path spatial_reconstructed_sc.h5ad
```

## Evaluation signals

- For IGF2-IGF2R, spatialDM on reconstructed coordinates flags exactly 45 significant cells, primarily in heart and liver
- Significant IGF2-IGF2R cells exhibit signals at short reconstructed pairwise distances in the 2D pseudo space
- Spatially informed LRIs show a greater portion with reference support >1 (0.28 vs 0.16 for CellChat at reference support 5)
- LRIs with no reference support are reduced relative to CellChat (0.12 vs 0.30)
- The reconstructed output AnnData contains expected new fields: 'cosine sim of rep', 'representation', 'referenced x', 'referenced y', 'de novo x', 'de novo y'

## Limitations

- SC-ST mapping can cause a 'cell clumping' issue where multiple neighbors are projected into the same reference location, losing predicted SC-SC distances
- CCC validation has circularity: reference CCCs were identified through the same approach with spatially reconstructed SC data, so results could be advanced toward it
- Large gaps between ST and SC samples degrade global reconstruction across all methods, and tissues with highly repeating elements challenge spatial reconstruction

## Evidence

- [discussion] reconstructed coordinates enable cell-level signaling extraction: "We employed CellContrast for CCC analysis on E9.5 mouse embryo SC data and compared CCCs inferred solely from gene expression (with CellChat)"
- [abstract] spatial information mitigates false positives: "applying it to cell-type co-localization and cell-cell communication analysis with real-world SC samples, proving the recovered cell locations empower more discoveries and mitigate potential false"
- [readme] MDS de novo projection of SC-SC distances: "--enable_denovo ## optional, run MDS to leverage the SC-SC pairwise distance to 2D pseudo space"
- [intro] cosine similarity encodes spatial proximity: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
