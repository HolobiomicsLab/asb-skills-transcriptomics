---
name: score-based-gene-expression-decomposition-via-langevin-sampling
description: Use when spot-level ST data (e.g., Visium, ~55 µm spots with multiple
  cells) must be decomposed into per-cell whole-transcriptome expression using an
  unpaired or paired scRNA-seq reference.
when_to_use_negative:
- The ST platform already provides single-cell, whole-transcriptome resolution, or
  the goal is only spot-level cell-type proportion estimation — deconvolution methods
  like RCTD, Cell2location, or CARD suffice
- Only a few hundred predefined genes are measured with no scRNA-seq reference to
  train the score-based generative model
- Platform effects between the ST data and the single-cell reference cannot be appropriately
  corrected — the article warns decomposition results 'would not be satisfactory'
  in that case
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SpatialScope
  role: Unified deep generative framework whose third step (gene expression decomposition)
    trains a conditional score-based model on the scRNA-seq reference and samples
    per-cell expression via annealed Langevin dynamics
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist
  role: Nucleus segmentation producing the per-spot cell counts and cell_locations
    that define which cells receive decomposed expression
- name: RCTD
  role: Benchmarked cell-type deconvolution baseline; estimates per-spot cell-type
    proportions without single-cell resolution
  repo: https://github.com/dmcable/spacexr
- name: Giotto
  role: Downstream ligand-receptor interaction detection (spatCellCellcom) run on
    the decomposed single-cell output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:e883d21be0ec6d44f4d11aa46c048449ca78a482555ed54f04a2692227ee73cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - Giotto
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Score-based gene expression decomposition via Langevin sampling

## Summary

SpatialScope's third step: a conditional score-based generative model trained on the snRNA-seq reference infers posterior-mean single-cell, transcriptome-wide expression at each segmented nucleus by annealed Langevin dynamics posterior sampling, upgrading spot-level Visium data to single-cell resolution.

## When to use

Use when spot-level ST data (e.g., Visium, ~55 µm spots with multiple cells) must be decomposed into per-cell whole-transcriptome expression using an unpaired or paired scRNA-seq reference.

## When NOT to use

- The ST platform already provides single-cell, whole-transcriptome resolution, or the goal is only spot-level cell-type proportion estimation — deconvolution methods like RCTD, Cell2location, or CARD suffice
- Only a few hundred predefined genes are measured with no scRNA-seq reference to train the score-based generative model
- Platform effects between the ST data and the single-cell reference cannot be appropriately corrected — the article warns decomposition results 'would not be satisfactory' in that case

## Inputs

- Spot-level ST count matrix (e.g., Visium h5ad, heart dataset with 3813 spots after removing spots with total UMI < 100)
- Segmented sp_adata_ns.h5ad with cell_locations from StarDist nucleus segmentation
- Cell type assignments (CellTypeLabel_nu10.csv / sp_adata.h5ad) from the cell type identification step
- Pretrained score-based generative model checkpoint (e.g., model_5000.pt) trained on the snRNA-seq reference
- Single-cell reference data (e.g., human heart snRNA-seq atlas with 10 major cell types) for batch-effect correction (gamma_g) and count normalization

## Outputs

- generated_cells.h5ad containing posterior-mean single-cell, transcriptome-wide expression for each segmented cell
- Decomposed single-cell expression supporting downstream analyses: ligand-receptor interaction detection, cellular subtype localization, and spatially differentially expressed gene identification

## How to apply

After SpatialScope nucleus segmentation (StarDist) and cell type identification (Poisson model with Potts prior nu=10) produce sp_adata.h5ad with CellTypeLabel_nu10.csv, first correct batch effects between ST and single-cell data: divide ST counts by exp(gamma_g) for each gene's platform effect and normalize total counts to the single-cell reference median. Then load the pretrained score-based checkpoint (e.g., model_5000.pt) trained on the reference and run gene expression decomposition with annealed Langevin dynamics (L=232 steps, T=5 noise levels) to sample the posterior p(Xi|yi,ki) at each cell, yielding per-cell posterior-mean expression. For Slide-seq, replace nucleus segmentation with singlet/doublet classification assuming at most two cell types per spot. Validate that the regenerated single-cell expression reproduces the reference cell-type profiles and supports downstream ligand-receptor analysis (e.g., Giotto spatCellCellcom recovering JAG1-NOTCH3, DLL4-NOTCH3, SERPINE2-LRP1, RTN4-LINGO1 in the heart vascular ROI).

## Related tools

- **SpatialScope** (Unified deep generative framework whose third step (gene expression decomposition) trains a conditional score-based model on the scRNA-seq reference and samples per-cell expression via annealed Langevin dynamics) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist** (Nucleus segmentation producing the per-spot cell counts and cell_locations that define which cells receive decomposed expression)
- **RCTD** (Benchmarked cell-type deconvolution baseline; estimates per-spot cell-type proportions without single-cell resolution) — https://github.com/dmcable/spacexr
- **Giotto** (Downstream ligand-receptor interaction detection (spatCellCellcom) run on the decomposed single-cell output)

## Examples

```
python SpatialScope/Decomposition.py --cell_type_result CellTypeLabel_nu10.csv --ST_point sp_adata_ns.h5ad --checkpoint model_5000.pt --output generated_cells.h5ad --L 232 --T 5
```

## Evaluation signals

- Regenerated cell-type composition from the decomposed single cells is consistent with the snRNA-seq reference (e.g., 10 major heart cell types)
- Arterial SMCs (TAGLN-high/LGR6-low) localize to the central vascular structure (~18 spots within the 331-spot vascular ROI), matching the atlas-defined arterial signature
- Giotto spatCellCellcom on the decomposed output (p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5, rec_expr > 0.5) recovers the expected EC-SMC pairs JAG1-NOTCH3, DLL4-NOTCH3, SERPINE2-LRP1, RTN4-LINGO1
- Imputed transcriptome-wide expression matches measured values when validated on image-based ST data (MERFISH MOp mouse1_slice180)
- Batch-effect correction is verified: ST counts divided by exp(gamma_g) and total-count normalized to the single-cell reference median before decomposition

## Limitations

- Training the deep generative models is time-consuming — measured in hours or days — although inference is fast
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Cell type identification and gene expression decomposition results would not be satisfactory if the platform (batch) effects are not appropriately corrected
- If histological images are unavailable, alternative image data (e.g., DAPI) complicate nucleus segmentation and thus per-cell assignment

## Evidence

- [methods] the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] we use Langevin dynamics to get samples from the posterior p(Xi|yi,ki): [evidence span withheld — anchor status: not_found]
- [methods] we adjust the batch effects between ST and single-cell reference data before we perform gene expression decomposition: "we adjust the batch effects between ST and single-cell reference data before we perform gene expression decomposition"
- [methods] We first learn a score-based generative model to approximate the expression distribution of different cell types: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [intro] considering the larger spot size of 55 μm, a Visium spot often contains multiple cells, which limits its usage in resolving detailed tissue structure: "considering the larger spot size of 55 μm, a Visium spot often contains multiple cells, which limits its usage in resolving detailed tissue structure"
- [methods] we selected top and reliable ligand-receptor signaling with the following threshold: p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 & rec_expr > 0.5: "we selected top and reliable ligand-receptor signaling with the following threshold: p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 & rec_expr > 0.5"
