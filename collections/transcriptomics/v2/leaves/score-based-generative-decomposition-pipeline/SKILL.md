---
name: score-based-generative-decomposition-pipeline
description: Use when seq-based ST data (e.g. 10x Visium, Slide-seq V2) must be decomposed
  to single-cell resolution using a paired scRNA-seq reference, or when image-based
  ST data need transcriptome-wide expression inference.
when_to_use_negative:
- Only spot-level cell-type proportions are needed without single-cell resolution
  — deconvolution methods like RCTD, Cell2location, or CARD suffice and are cheaper
- No paired histological/nuclear image is available and the platform is not Slide-seq,
  so nucleus segmentation and cell-count estimation cannot be performed reliably
- A gene panel of only a few hundred measured genes is the target output rather than
  transcriptome-wide inference — image-based ST (MERFISH/seqFISH) may already suffice
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_2229
tools:
- name: SpatialScope
  role: Primary three-step pipeline (nucleus segmentation, cell type identification,
    gene expression decomposition via score-based generative model); installed from
    GitHub or via Docker image xiaojs95/spatialscope
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist+RCTD
  role: 'Combined benchmarking baseline: StarDist for H&E nucleus segmentation, RCTD
    with doublet_mode=''full'' for cell type identification'
  repo: https://github.com/dmcable/spacexr
- name: Tangram
  role: Alignment baseline that assigns scRNA-seq cells to spatial spots for comparison
    of decomposition accuracy
- name: CytoSPACE
  role: Alignment baseline whose mapped single-cell ids are used directly as decomposed
    expressions for cosine-similarity comparison
- name: Cell2location
  role: Deconvolution baseline run with regression max_epochs=250, batch_size=2500,
    lr=0.002; model max_epochs=30000
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: Spatially informed deconvolution baseline run with defaults
  repo: https://github.com/YingMa0107/CARD
- name: SpaOTsc
  role: Optimal-transport mapping deconvolution baseline
  repo: https://github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: Optimal-transport alignment deconvolution baseline
  repo: https://github.com/rajewsky-lab/novosparc
- name: DestVI
  role: scvi-tools-based deconvolution baseline run with max_epochs=250, lr=0.0001
  repo: https://github.com/scverse/scvi-tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:c50b3ec29dca255f586391c77b843e1bd8035a2fd87bd53898ae8b070f310f05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CARD
  - Cell2location
  - CytoSPACE
  - DSTG
  - DestVI
  - RCTD
  - SPOTlight
  - STRIDE
  - SpaOTsc
  - SpatialDWLS
  - SpatialScope
  - StarDist
  - StarDist+RCTD
  - Tangram
  - novoSpaRc
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# score-based-generative-decomposition-pipeline

## Summary

Three-step SpatialScope workflow — nucleus segmentation, cell type identification with Potts smoothness, and gene expression decomposition via a conditional score-based generative model with annealed Langevin dynamics sampling — that resolves spot-level spatial transcriptomics data to single-cell, transcriptome-wide resolution.

## When to use

Use when seq-based ST data (e.g. 10x Visium, Slide-seq V2) must be decomposed to single-cell resolution using a paired scRNA-seq reference, or when image-based ST data need transcriptome-wide expression inference.

## When NOT to use

- Only spot-level cell-type proportions are needed without single-cell resolution — deconvolution methods like RCTD, Cell2location, or CARD suffice and are cheaper
- No paired histological/nuclear image is available and the platform is not Slide-seq, so nucleus segmentation and cell-count estimation cannot be performed reliably
- A gene panel of only a few hundred measured genes is the target output rather than transcriptome-wide inference — image-based ST (MERFISH/seqFISH) may already suffice

## Inputs

- Spot-level spatial transcriptomics counts (10x Visium, Slide-seq V2) with spatial coordinates
- Paired scRNA-seq/snRNA-seq reference with cell-type labels (e.g. human heart snRNA-seq atlas with 10 major cell types; mouse MOp droplet-based snRNA-seq, 26,431 cells across 20 cell types)
- Paired H&E histology image (or DAPI/nuclear image) for nucleus segmentation
- Preprocessed count matrix filtered: genes with zero counts across all spots removed, spots with total UMI < 100 removed, genes retained if non-zero on ≥10 spots, spots retained if non-zero for ≥50 genes

## Outputs

- Per-nucleus cell type labels for each spatial spot
- Transcriptome-wide single-cell resolution gene expression matrix decomposed per cell within each spot
- Benchmark metrics: single-cell misclassification error rate, spot-level PCC/RMSE, per-cell cosine similarity vs ground truth

## How to apply

Run the SpatialScope three-step pipeline: (1) segment nuclei from the paired H&E image (or, for Slide-seq, replace segmentation with singlet/doublet classification assuming at most two cell types per spot) to quantify cell counts per spot; (2) identify cell types per nucleus using the scRNA-seq reference with a Potts smoothness prior at ν=10 that encourages neighboring cells to share cell type; (3) decompose gene expression by learning a conditional score-based generative model from the single-cell reference and sampling posterior expression per cell with annealed Langevin dynamics (trained 7500 epochs, L=232, T=5), after batch-effect correction between ST and reference data. Before modeling, filter genes with zero counts across all spots and spots with total UMI < 100; retain genes expressed on at least 10 spots and spots expressing at least 50 genes. Compare against alignment baselines (Tangram, CytoSPACE) and StarDist+RCTD to benchmark output quality.

## Related tools

- **SpatialScope** (Primary three-step pipeline (nucleus segmentation, cell type identification, gene expression decomposition via score-based generative model); installed from GitHub or via Docker image xiaojs95/spatialscope) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist+RCTD** (Combined benchmarking baseline: StarDist for H&E nucleus segmentation, RCTD with doublet_mode='full' for cell type identification) — https://github.com/dmcable/spacexr
- **Tangram** (Alignment baseline that assigns scRNA-seq cells to spatial spots for comparison of decomposition accuracy)
- **CytoSPACE** (Alignment baseline whose mapped single-cell ids are used directly as decomposed expressions for cosine-similarity comparison)
- **Cell2location** (Deconvolution baseline run with regression max_epochs=250, batch_size=2500, lr=0.002; model max_epochs=30000) — https://github.com/BayraktarLab/cell2location
- **CARD** (Spatially informed deconvolution baseline run with defaults) — https://github.com/YingMa0107/CARD
- **SpaOTsc** (Optimal-transport mapping deconvolution baseline) — https://github.com/zcang/SpaOTsc
- **novoSpaRc** (Optimal-transport alignment deconvolution baseline) — https://github.com/rajewsky-lab/novosparc
- **DestVI** (scvi-tools-based deconvolution baseline run with max_epochs=250, lr=0.0001) — https://github.com/scverse/scvi-tools

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git; cd SpatialScope; conda env create -f environment.yml  # or: docker pull xiaojs95/spatialscope
```

## Evaluation signals

- On simulated Dataset 1, per-cell cosine similarity should be ≈0.90 for SpatialScope vs 0.44 (Tangram) and 0.57 (CytoSPACE)
- Single-cell misclassification error rate should drop by 50.3% vs Tangram, 20.6% vs CytoSPACE, and 6.3% vs StarDist+RCTD
- Spot-level PCC improvements of 9.4%–157.5% (up to 51.4%) and RMSE improvements of 25.3%–89.7% relative to competing methods
- Cosine similarity should remain highest across all tested UMI subsample rates and numbers of cells per simulated spot
- Output should be single-cell resolution: each simulated/segmented cell within a spot receives a cell type label and a transcriptome-wide expression vector

## Limitations

- Training the deep generative model is time-consuming, measured in hours or days
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Platform effects between ST and scRNA-seq data must be corrected or cell type identification and gene expression decomposition results will be unsatisfactory
- Paired histological images may be unavailable; alternative image types (e.g. DAPI) or absent images complicate cell count estimation
- Adaptive learning of neighboring cell-type similarity (via DGI or GCN) is left as future work

## Evidence

- [methods] Three-step pipeline: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Potts smoothness prior: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Filter thresholds: "We filtered out genes that have zero counts across all spots and filtered out spots with total UMI counts less than 100"
- [methods] Slide-seq adaptation: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [abstract] Claimed capability: "SpatialScope not only enhances seq-based ST data to achieve single-cell resolution, but also accurately infers transcriptome-wide expression levels for image-based ST data"
- [discussion] Training time caveat: "the training time of the deep generative models is measured in hours or days"
- [discussion] Potts assumption caveat: "the assumption that neighboring cells belong to the same cell type may not always be valid"
- [readme] Installation path: "$ git clone https://github.com/YangLabHKUST/SpatialScope.git"
