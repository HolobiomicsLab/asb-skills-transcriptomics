---
name: cell-type-deconvolution-method-evaluation
description: Use when comparing cell type identification or gene expression decomposition
  methods on simulated single-slice spatial transcriptomics benchmarks with paired
  scRNA-seq references and ground-truth per-cell assignments.
when_to_use_negative:
- Input is real (non-simulated) tissue ST data without ground-truth single-cell labels
  — error-rate and cosine-similarity evaluation is not possible
- The goal is deconvolving a single new dataset rather than benchmarking methods head-to-head
  — use a single chosen method instead
- Data are unpaired, i.e. no scRNA-seq reference exists for the tissue being profiled,
  since all compared methods are reference-based
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Method under evaluation; three-step pipeline (nucleus segmentation, cell type
    identification, gene expression decomposition) benchmarked on Datasets 1-4
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Alignment-based baseline; assigns scRNA-seq cells to spatial spots for cell
    type identification and expression decomposition
- name: CytoSPACE
  role: Alignment-based baseline; mapped single-cell ids used directly as decomposed
    expressions
- name: StarDist
  role: H&E nucleus segmentation combined with RCTD as the segmentation-based baseline
- name: RCTD
  role: Deconvolution baseline run in full mode and, with StarDist, doublet_mode='full';
    learns cell type profiles from scRNA-seq to label spatial pixels
  repo: https://github.com/dmcable/spacexr
- name: Cell2location
  role: Deconvolution baseline; regression max_epochs=250, batch_size=2500, lr=0.002,
    model max_epochs=30000
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: Spatially informed deconvolution baseline run with defaults
  repo: https://github.com/YingMa0107/CARD
- name: DestVI
  role: Deconvolution baseline via scvi-tools; max_epochs=250, lr=0.0001
  repo: https://github.com/scverse/scvi-tools
- name: SpaOTsc
  role: Optimal-transport mapping baseline
  repo: https://github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: Optimal-transport alignment baseline
  repo: https://github.com/rajewsky-lab/novosparc
- name: STRIDE
  role: Topic-modeling deconvolution baseline; gene-use=All, st/sc-scale-factor=300
  repo: https://github.com/DongqingSun96/STRIDE
- name: SPOTlight
  role: Deconvolution baseline
  repo: https://github.com/MarcElosua/SPOTlight
- name: DSTG
  role: Graph-convolution deconvolution baseline
  repo: https://github.com/Su-informatics-lab/DSTG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:16de6388d5687d41b7e4cee59550694ea5e42e6793942f6fb9552a983fe40d2d
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
  - Tangram
  - novoSpaRc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# cell-type-deconvolution-method-evaluation

## Summary

Benchmarking protocol for evaluating cell type identification and single-cell gene expression decomposition methods on simulated single-slice ST datasets with paired scRNA-seq references, as used to evaluate SpatialScope against Tangram, CytoSPACE, StarDist+RCTD and ten deconvolution baselines.

## When to use

Use when comparing cell type identification or gene expression decomposition methods on simulated single-slice spatial transcriptomics benchmarks with paired scRNA-seq references and ground-truth per-cell assignments.

## When NOT to use

- Input is real (non-simulated) tissue ST data without ground-truth single-cell labels — error-rate and cosine-similarity evaluation is not possible
- The goal is deconvolving a single new dataset rather than benchmarking methods head-to-head — use a single chosen method instead
- Data are unpaired, i.e. no scRNA-seq reference exists for the tissue being profiled, since all compared methods are reference-based

## Inputs

- Simulated single-slice ST benchmark datasets (Dataset 1-4) with ground-truth cell type assignments per cell
- Paired scRNA-seq reference atlases (e.g. Smart-seq2 mouse cortical reference, 14,249 cells, 23 cell types; mouse cerebellum snRNA-seq, 15,609 cells, 19 cell types; droplet MOp snRNA-seq, 26,431 cells, 20 cell types; human heart snRNA-seq atlas, 10 major cell types)
- H&E histology images for segmentation-based methods (StarDist)
- Method-specific parameter settings per baseline (e.g. Cell2location regression max_epochs=250, lr=0.002)

## Outputs

- Per-method single-cell misclassification error rates on each benchmark dataset
- Spot-level cell-type proportion accuracy (PCC and RMSE) per method
- Per-cell cosine similarity of inferred transcriptome-wide expression vs ground truth (per method)
- Robustness curves across UMI subsample rates and varying cells-per-spot

## How to apply

Load the simulated benchmark datasets (Datasets 1-6, e.g. Dataset 1-4) with their paired scRNA-seq references and run each method under its documented settings: SpatialScope via its three-step pipeline (nucleus segmentation or singlet/doublet classification for Slide-seq, cell type identification with Potts smoothness at nu=10, gene expression decomposition via the score-based generative model trained 7500 epochs, L=232, T=5); StarDist+RCTD with RCTD in doublet_mode='full'; Tangram and CytoSPACE alignment baselines; and deconvolution baselines RCTD (full mode), SpatialDWLS (n_cell=20), Cell2location (regression max_epochs=250, batch_size=2500, lr=0.002; model max_epochs=30000), CARD, SpaOTsc, novoSpaRc, DestVI (max_epochs=250, lr=0.0001), STRIDE, SPOTlight, DSTG. Evaluate cell type identification by single-cell misclassification error rate and spot-level PCC/RMSE, and gene expression decomposition by per-cell cosine similarity, repeating under varying UMI subsample rates and numbers of cells per simulated spot. Data preprocessing follows the paper's filtering: drop genes with zero counts across all spots, spots with total UMI < 100, retain genes expressed on >=10 spots and spots expressing >=50 genes. The rationale is that simulated datasets carry ground-truth single-cell labels, enabling exact error-rate and cosine-similarity comparisons that real tissues cannot provide.

## Related tools

- **SpatialScope** (Method under evaluation; three-step pipeline (nucleus segmentation, cell type identification, gene expression decomposition) benchmarked on Datasets 1-4) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Alignment-based baseline; assigns scRNA-seq cells to spatial spots for cell type identification and expression decomposition)
- **CytoSPACE** (Alignment-based baseline; mapped single-cell ids used directly as decomposed expressions)
- **StarDist** (H&E nucleus segmentation combined with RCTD as the segmentation-based baseline)
- **RCTD** (Deconvolution baseline run in full mode and, with StarDist, doublet_mode='full'; learns cell type profiles from scRNA-seq to label spatial pixels) — https://github.com/dmcable/spacexr
- **Cell2location** (Deconvolution baseline; regression max_epochs=250, batch_size=2500, lr=0.002, model max_epochs=30000) — https://github.com/BayraktarLab/cell2location
- **CARD** (Spatially informed deconvolution baseline run with defaults) — https://github.com/YingMa0107/CARD
- **DestVI** (Deconvolution baseline via scvi-tools; max_epochs=250, lr=0.0001) — https://github.com/scverse/scvi-tools
- **SpaOTsc** (Optimal-transport mapping baseline) — https://github.com/zcang/SpaOTsc
- **novoSpaRc** (Optimal-transport alignment baseline) — https://github.com/rajewsky-lab/novosparc
- **STRIDE** (Topic-modeling deconvolution baseline; gene-use=All, st/sc-scale-factor=300) — https://github.com/DongqingSun96/STRIDE
- **SPOTlight** (Deconvolution baseline) — https://github.com/MarcElosua/SPOTlight
- **DSTG** (Graph-convolution deconvolution baseline) — https://github.com/Su-informatics-lab/DSTG

## Examples

```
devtools::install_github("dmcable/spacexr", build_vignettes = FALSE)
```

## Evaluation signals

- SpatialScope per-cell cosine similarity should reach ~0.90 vs ~0.44 (Tangram) and ~0.57 (CytoSPACE) on Dataset 1
- Single-cell misclassification error rate reductions of 50.3% (vs Tangram), 20.6% (vs CytoSPACE), and 6.3% (vs StarDist+RCTD) should be reproduced
- Spot-level PCC improvements of 9.4%-157.5% (up to 51.4%) and RMSE improvements of 25.3%-89.7% vs compared methods
- Highest cosine similarity must hold across all varied cells-per-spot and UMI subsample scenarios
- Preprocessing invariants: spots with total UMI < 100 removed; genes expressed on >=10 spots and spots expressing >=50 genes retained

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Training the deep generative model is time-consuming (hours or days), making full benchmark sweeps expensive
- Platform effects must be appropriately corrected between ST and scRNA-seq data or cell type identification and decomposition results are unsatisfactory
- The evaluation relies on simulated benchmark datasets; conclusions depend on the simulation assumptions (cells per spot, UMI subsampling)

## Evidence

- [methods] the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] cell type identification on six benchmarking datasets (Supplementary Fig. 4, Dataset 1-6): "cell type identification on six benchmarking datasets (Supplementary Fig. 4, Dataset 1-6)"
- [methods] we retained genes with non-zero expression level on at least 10 spots: "we retained genes with non-zero expression level on at least 10 spots"
- [readme] RCTD learns cell type profiles from the scRNA-seq dataset, and uses these to label the spatial transcriptomics pixels as cell types.: [evidence span withheld — anchor status: external_doc]
- [intro] unlike alignment-based methods such as Tangram18 and CytoSPACE21 that assign existing cells from scRNA-seq data to spatial spots: "unlike alignment-based methods such as Tangram18 and CytoSPACE21 that assign existing cells from scRNA-seq data to spatial spots"
- [methods] we use Langevin dynamics to get samples from the posterior p(Xi|yi,ki): [evidence span withheld — anchor status: not_found]
