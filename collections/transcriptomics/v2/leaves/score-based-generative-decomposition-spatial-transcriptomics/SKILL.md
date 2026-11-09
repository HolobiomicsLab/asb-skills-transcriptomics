---
name: score-based-generative-decomposition-spatial-transcriptomics
description: Use when you have nucleus-segmented or singlet/doublet-classified spatial
  transcriptomics spots plus a labeled scRNA-seq reference and need transcriptome-wide
  per-cell expression beyond spot-level resolution.
when_to_use_negative:
- You only need spot-level cell-type proportions rather than per-cell expression —
  simpler deconvolution tools (RCTD, Cell2location, CARD) suffice, and the generative
  training/decomposition cost (hours or days) is unnecessary
- You lack a labeled scRNA-seq reference or cannot correct platform effects between
  reference and ST data — the article warns results 'would not be satisfactory if
  the platform effects are not appropriately corrected'
- No histological image or equivalent (e.g., DAPI) is available and cell counts per
  spot cannot be estimated, complicating the per-cell decomposition
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Implements the score-based conditional generative model, annealed Langevin
    dynamics posterior sampling, and γ̂g batch correction for gene expression decomposition
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist
  role: Provides per-spot nucleus segmentation (cell counts and cell_locations) on
    matched H&E images that feed the decomposition step
- name: RCTD
  role: Benchmarking baseline for the cell-type map produced alongside decomposition
    (e.g., mouse cortex StarDist+RCTD comparison)
  repo: https://github.com/dmcable/spacexr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:90c45016af57d19f280282917aa0bde6831fee8802e43e5d6e84519f63bea3b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - PASTE
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Score-based generative decomposition of spot-level ST expression to single-cell resolution

## Summary

SpatialScope's gene expression decomposition step: a score-based conditional generative model learned from a labeled scRNA-seq reference is combined with annealed Langevin dynamics posterior sampling to infer transcriptome-wide, single-cell-resolution expression for every nucleus/cell segmented from Visium or Slide-seq spots, after cross-platform batch correction.

## When to use

Use when you have nucleus-segmented or singlet/doublet-classified spatial transcriptomics spots plus a labeled scRNA-seq reference and need transcriptome-wide per-cell expression beyond spot-level resolution.

## When NOT to use

- You only need spot-level cell-type proportions rather than per-cell expression — simpler deconvolution tools (RCTD, Cell2location, CARD) suffice, and the generative training/decomposition cost (hours or days) is unnecessary
- You lack a labeled scRNA-seq reference or cannot correct platform effects between reference and ST data — the article warns results 'would not be satisfactory if the platform effects are not appropriately corrected'
- No histological image or equivalent (e.g., DAPI) is available and cell counts per spot cannot be estimated, complicating the per-cell decomposition

## Inputs

- Spot-level ST data (10x Visium or Slide-seq V2) with per-spot cell counts and cell_locations from nucleus segmentation or singlet/doublet classification
- Labeled scRNA-seq reference (e.g., Smart-seq2 mouse cortex reference with 14,249 cells across 23 cell types) normalized by total counts and log(1+x) transformed
- Pretrained score-based generative model checkpoint for the reference
- Cell type labels per segmented cell (e.g., CellTypeLabel_nu10.csv) from MAP/Potts inference

## Outputs

- Single-cell-resolution h5ad file with decomposed transcriptome-wide per-cell expression for every segmented cell
- Refined per-cell marker gene expression maps for downstream visualization (PNGs) and ligand-receptor / spatial DE analyses

## How to apply

After SpatialScope nucleus segmentation (e.g., StarDist on H&E for Visium) and cell type identification, run gene expression decomposition with a pretrained score-based conditional generative model checkpoint (annealed Langevin dynamics with L = 232, T = 5), processing at most ~1000 spots per batch on 4 GPUs. Before decomposition, correct platform effects between ST and scRNA-seq data with the gene-specific (γ̂g) cross-platform batch correction and sequencing-depth normalization, since uncorrected platform effects yield unsatisfactory results. For Slide-seq data, the model resolves spots by assuming at most two cell types per 10 μm spot (singlet/doublet classification replaces segmentation). The sampled posterior p(Xi|yi,ki) produces a single-cell-resolution h5ad containing decomposed per-cell expression for all segmented cells, which can then support ligand-receptor interaction detection, subtype localization, and spatially DE gene identification.

## Related tools

- **SpatialScope** (Implements the score-based conditional generative model, annealed Langevin dynamics posterior sampling, and γ̂g batch correction for gene expression decomposition) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist** (Provides per-spot nucleus segmentation (cell counts and cell_locations) on matched H&E images that feed the decomposition step)
- **RCTD** (Benchmarking baseline for the cell-type map produced alongside decomposition (e.g., mouse cortex StarDist+RCTD comparison)) — https://github.com/dmcable/spacexr

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && conda env create -f; docker pull xiaojs95/spatialscope
```

## Evaluation signals

- Output h5ad contains decomposed expression entries for every segmented cell (not per spot), confirming single-cell resolution
- Discrete cell type labels reproduce expected tissue architecture (e.g., mouse cortex L4 and L6 IT cells assigned to their corresponding cortical layers)
- Imputed transcriptome-wide expression validates against held-out image-based truth (e.g., MERFISH mouse1_slice180) where available
- Platform-effect-corrected reference: verify γ̂g batch correction and sequencing-depth normalization were applied before posterior sampling
- Batch-size/GPU constraint respected: at most ~1000 spots per decomposition batch

## Limitations

- Training the deep generative models is time-consuming, measured in hours or days
- Decomposition results are unsatisfactory if cross-platform effects between ST and scRNA-seq data are not appropriately corrected
- Training the generative model to approximate the single-cell reference distribution is a time-costly prerequisite before decomposition

## Evidence

- [methods] Three-step decomposition pipeline: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Generative model learning: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Posterior sampling: [evidence span withheld — anchor status: not_found]
- [methods] Batch correction before decomposition: "we adjust the batch effects between ST and single-cell reference data before we perform gene expression decomposition"
- [readme] Tool purpose: "A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models"
