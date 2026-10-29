---
name: single-cell-spatial-cell-type-benchmarking
description: Use when evaluating how well a spatial transcriptomics integration method
  identifies cell types or decomposes spot-level gene expression on simulated benchmarks
  with ground-truth single-cell assignments.
when_to_use_negative:
- Real tissue data without single-cell ground truth — misclassification error rate
  and per-cell cosine similarity cannot be computed, so this benchmarking design does
  not apply
- Methods that only output spot-level cell-type proportions without single-cell assignments
  (e.g., pure deconvolution) when the metric of interest is per-cell cosine similarity
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: 'candidate method: nucleus segmentation, Potts-based cell type identification
    (ν=10), and score-based generative gene expression decomposition'
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: alignment baseline assigning scRNA-seq cells to spatial spots
- name: CytoSPACE
  role: alignment baseline; mapped single-cell ids used directly as decomposed expressions
- name: StarDist
  role: H&E nucleus segmentation paired with RCTD as combined segmentation+deconvolution
    baseline
- name: RCTD
  role: deconvolution baseline (full mode) and combined with StarDist
  repo: https://github.com/dmcable/spacexr
- name: Cell2location
  role: deconvolution baseline; regression max_epochs=250, batch_size=2500, lr=0.002;
    model max_epochs=30000
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: spatially informed deconvolution baseline run with defaults
  repo: https://github.com/YingMa0107/CARD
- name: SpaOTsc
  role: optimal-transport mapping baseline
  repo: https://github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: optimal-transport alignment baseline
  repo: https://github.com/rajewsky-lab/novosparc
- name: DestVI
  role: deconvolution baseline via scvi-tools; max_epochs=250, lr=0.0001
  repo: https://github.com/scverse/scvi-tools
- name: STRIDE
  role: topic-modeling deconvolution baseline; gene-use=All, st/sc-scale-factor=300
  repo: https://github.com/DongqingSun96/STRIDE
- name: SPOTlight
  role: NMF-based deconvolution baseline
  repo: https://github.com/MarcElosua/SPOTlight
- name: DSTG
  role: graph-convolution deconvolution baseline
  repo: https://github.com/Su-informatics-lab/DSTG
- name: SpatialDWLS
  role: deconvolution baseline with n_cell=20
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:3341bfd81752d2f4305636995c31434e073d8d3244ff5102b65183672f69094d
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# single-cell-spatial-cell-type-benchmarking

## Summary

Benchmarking protocol for comparing SpatialScope's three-step pipeline (nucleus segmentation, cell type identification, gene expression decomposition) against alignment, segmentation+deconvolution, and deconvolution baselines on simulated single-slice ST datasets with paired scRNA-seq references.

## When to use

Use when evaluating how well a spatial transcriptomics integration method identifies cell types or decomposes spot-level gene expression on simulated benchmarks with ground-truth single-cell assignments.

## When NOT to use

- Real tissue data without single-cell ground truth — misclassification error rate and per-cell cosine similarity cannot be computed, so this benchmarking design does not apply
- Methods that only output spot-level cell-type proportions without single-cell assignments (e.g., pure deconvolution) when the metric of interest is per-cell cosine similarity

## Inputs

- Four simulated single-slice benchmark ST datasets (Datasets 1-4) with known per-cell cell-type ground truth
- Paired scRNA-seq reference profiles (e.g., Smart-seq2 mouse cortical reference with 14,249 cells / 23 cell types, snRNA-seq references with 19-20 cell types)
- Documented baseline configurations (RCTD full mode, SpatialDWLS n_cell=20, Cell2location epochs/lr, DestVI max_epochs=250, STRIDE scale-factor=300, etc.)

## Outputs

- Per-method single-cell misclassification error rates
- Spot-level PCC and RMSE for cell type identification
- Per-cell cosine similarity of decomposed gene expression (SpatialScope vs Tangram/CytoSPACE/StarDist+RCTD)
- Sensitivity curves across varying UMI subsample rates and numbers of cells per simulated spot

## How to apply

Load the simulated single-slice benchmark datasets (Datasets 1-4) with their paired scRNA-seq references and run the candidate method's full pipeline: for SpatialScope, nucleus segmentation followed by cell type identification using Potts smoothness (ν=10), then gene expression decomposition via a score-based generative model trained 7500 epochs (L=232, T=5) with annealed Langevin posterior sampling. Run baselines per documented settings: StarDist+RCTD (RCTD doublet_mode='full'), Tangram and CytoSPACE alignment (CytoSPACE mapped single-cell ids used directly as decomposed expressions), and ten deconvolution methods including RCTD (full mode), SpatialDWLS (n_cell=20), Cell2location (regression max_epochs=250, batch_size=2500, lr=0.002; model max_epochs=30000), CARD, SpaOTsc, novoSpaRc, DestVI (max_epochs=250, lr=0.0001), STRIDE (gene-use=All, scale-factor=300), SPOTlight, and DSTG. Evaluate cell type identification by single-cell misclassification error rate and spot-level PCC/RMSE, and decomposition by per-cell cosine similarity. Rationale: simulating spots from single-cell ground truth enables exact error quantification that is impossible on real tissue.

## Related tools

- **SpatialScope** (candidate method: nucleus segmentation, Potts-based cell type identification (ν=10), and score-based generative gene expression decomposition) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (alignment baseline assigning scRNA-seq cells to spatial spots)
- **CytoSPACE** (alignment baseline; mapped single-cell ids used directly as decomposed expressions)
- **StarDist** (H&E nucleus segmentation paired with RCTD as combined segmentation+deconvolution baseline)
- **RCTD** (deconvolution baseline (full mode) and combined with StarDist) — https://github.com/dmcable/spacexr
- **Cell2location** (deconvolution baseline; regression max_epochs=250, batch_size=2500, lr=0.002; model max_epochs=30000) — https://github.com/BayraktarLab/cell2location
- **CARD** (spatially informed deconvolution baseline run with defaults) — https://github.com/YingMa0107/CARD
- **SpaOTsc** (optimal-transport mapping baseline) — https://github.com/zcang/SpaOTsc
- **novoSpaRc** (optimal-transport alignment baseline) — https://github.com/rajewsky-lab/novosparc
- **DestVI** (deconvolution baseline via scvi-tools; max_epochs=250, lr=0.0001) — https://github.com/scverse/scvi-tools
- **STRIDE** (topic-modeling deconvolution baseline; gene-use=All, st/sc-scale-factor=300) — https://github.com/DongqingSun96/STRIDE
- **SPOTlight** (NMF-based deconvolution baseline) — https://github.com/MarcElosua/SPOTlight
- **DSTG** (graph-convolution deconvolution baseline) — https://github.com/Su-informatics-lab/DSTG
- **SpatialDWLS** (deconvolution baseline with n_cell=20)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml && jupyter notebook demos/Benchmarking-Dataset_1.ipynb
```

## Evaluation signals

- SpatialScope per-cell cosine similarity ≈0.90 on Dataset 1 vs 0.44 (Tangram) and 0.57 (CytoSPACE)
- Single-cell misclassification error rate reduced by 50.3% vs Tangram, 20.6% vs CytoSPACE, 6.3% vs StarDist+RCTD
- Spot-level PCC gains of 9.4%-157.5% (up to 51.4%) and RMSE gains of 25.3%-89.7% over baselines
- Highest cosine similarity maintained across all UMI-subsample rates and cells-per-spot scenarios
- SpatialScope pipeline must run end-to-end with the documented settings (ν=10, 7500 epochs, L=232, T=5) without failing on all four benchmark datasets

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Training the deep generative model is time-consuming (hours or days), making the benchmark expensive to rerun
- Simulation-based ground truth may not reflect real-tissue platform effects; results would not be satisfactory if platform effects are not appropriately corrected
- Paired histological images may be unavailable, complicating the nucleus segmentation step; DAPI-only or no-image cases require adaptation

## Evidence

- [results] SpatialScope outperformed all compared cell type identification methods on the four single-slice simulated benchmark datasets: [evidence span withheld — anchor status: not_found]
- [methods] three-step pipeline definition: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [intro] baseline deconvolution methods listed: "There are now several cell-type deconvolution methods for ST data, including RCTD14, Cell2location15, CARD16 and spatialDWLS17."
- [intro] alignment baselines: "unlike alignment-based methods such as Tangram18 and CytoSPACE21 that assign existing cells from scRNA-seq data to spatial spots"
- [discussion] platform effects caveat: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
- [readme] RCTD platform effect normalization (README): [evidence span withheld — anchor status: not_found]
