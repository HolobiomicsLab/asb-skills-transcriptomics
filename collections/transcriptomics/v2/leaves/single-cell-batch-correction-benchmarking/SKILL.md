---
name: single-cell-batch-correction-benchmarking
description: Use when comparing how well batch-effect correction methods mix datasets
  with overlapping cell types while preserving cell-type resolution, e.g. reproducing
  Cell BLAST Fig. 2a-style integration benchmarks.
when_to_use_negative:
- Datasets lack overlapping cell types across batches, so neither alignment-score
  mixing nor MAP cell-type resolution can be meaningfully computed
- Cell type or batch labels are absent, making the supervised mixing and precision
  metrics undefined
- The goal is integration for downstream annotation of a single query set rather than
  systematic method benchmarking
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Cell-querying framework whose neural network-based generative model (DIRECTi)
    produces the batch-corrected embeddings being benchmarked
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on each dataset combination to obtain low-dimensional
    embeddings modeling multilevel batch effect
  repo: https://github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST
  role: Python package imported to fit models and compute posterior-distance-based
    cell-to-cell similarity for MAP
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Competing variational-autoencoder batch-correction method applied to the same
    dataset combinations for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:fbe044f406d3c089e5edc87697967fe7e2dc6893c0c311dee58116d2ad99f7ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - Cell_BLAST
  - DIRECTi
  - Harmony
  - PyTorch
  - Scanorama
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-batch-correction-benchmarking

## Summary

Benchmark batch-effect correction and dimension-reduction methods for scRNA-seq dataset integration by computing Seurat alignment scores (batch mixing among matched cell types) and mean average precision (MAP, cell-type resolution) on corrected embeddings from Cell BLAST's DIRECTi generative model versus ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, and scVI.

## When to use

Use when comparing how well batch-effect correction methods mix datasets with overlapping cell types while preserving cell-type resolution, e.g. reproducing Cell BLAST Fig. 2a-style integration benchmarks.

## When NOT to use

- Datasets lack overlapping cell types across batches, so neither alignment-score mixing nor MAP cell-type resolution can be meaningfully computed
- Cell type or batch labels are absent, making the supervised mixing and precision metrics undefined
- The goal is integration for downstream annotation of a single query set rather than systematic method benchmarking

## Inputs

- scRNA-seq dataset combinations with overlapping cell types, as h5ad (anndata) matrices
- Cell type labels for each cell
- Batch labels (including within-dataset multilevel batch labels)
- Corrected low-dimensional cell embeddings from each evaluated method (DIRECTi, ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI)

## Outputs

- Seurat alignment score per method per dataset combination and per batch condition
- Mean average precision (MAP) per method using posterior-distance-based cell-to-cell similarity
- batch_benchmark_results.csv results table covering all eight evaluated methods
- Bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5

## How to apply

Load scRNA-seq dataset combinations with overlapping cell types, plus cell type and batch labels, as h5ad (anndata) matrices. Fit the Cell BLAST DIRECTi deep generative model on each combination to obtain low-dimensional cell embeddings that model batch effect at multiple levels, and apply competing methods (ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI) to the same combinations to produce their corrected embeddings. Quantify batch mixing with the Seurat alignment score computed on each corrected embedding, and assess preservation of cell-type relationships with mean average precision (MAP) using Cell BLAST's posterior-distance-based cell-to-cell similarity. Repeat scoring in the multilevel within-dataset batch setting to evaluate handling of intra-dataset batch variation. Assemble scores into a results table (batch_benchmark_results.csv) and render bar plots reproducing Fig. 2a and Supplementary Figs. 4-5; the multilevel batch modeling and posterior-based similarity metric are what distinguish Cell BLAST from competitors.

## Related tools

- **Cell BLAST** (Cell-querying framework whose neural network-based generative model (DIRECTi) produces the batch-corrected embeddings being benchmarked) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on each dataset combination to obtain low-dimensional embeddings modeling multilevel batch effect) — https://github.com/gao-lab/Cell_BLAST
- **Cell_BLAST** (Python package imported to fit models and compute posterior-distance-based cell-to-cell similarity for MAP) — https://github.com/gao-lab/Cell_BLAST
- **scVI** (Competing variational-autoencoder batch-correction method applied to the same dataset combinations for comparison)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- batch_benchmark_results.csv contains Seurat alignment score and MAP entries for all eight evaluated methods across the dataset combinations and the within-dataset multilevel batch condition
- Cell BLAST's scores are comparable to the ranking reported in Fig. 2a, with significantly better dataset mixing at high cell-type resolution
- Seurat alignment scores computed on corrected embeddings show mixing of batches among matched cell types
- MAP values reflect preservation of cell-type relationships under the posterior-distance-based cell-to-cell similarity metric
- Bar plots visually reproduce Fig. 2a and Supplementary Figs. 4-5 comparisons

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- Cell BLAST is over-parameterized and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues

## Evidence

- [discussion] Cell BLAST models multilevel batch effect and uncertainty: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [methods] pip install Cell BLAST: "pip install Cell-BLAST"
- [methods] import Cell_BLAST: "import Cell_BLAST as cb"
- [discussion] scVI Gaussian variational posterior: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior."
- [intro] Existing metrics suffer from batch effect: "they may not reflect cell-to-cell similarity faithfully, and suffer from nonbiological confounding variation across datasets"
- [readme] Reproduce paper results: [evidence span withheld — anchor status: external_doc]
