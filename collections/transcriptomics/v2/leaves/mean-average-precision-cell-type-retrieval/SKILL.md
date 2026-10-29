---
name: mean-average-precision-cell-type-retrieval
description: Use when benchmarking batch-effect correction or dimension-reduction
  methods on scRNA-seq dataset combinations and you need a retrieval-style metric
  of cell-type resolution (preservation of cell type relationships) alongside a dataset-mixing
  metric.
when_to_use_negative:
- You only need to quantify dataset mixing among batches without assessing cell-type
  relationship preservation (the Seurat alignment score alone addresses mixing)
- Datasets have fewer than about 1000 cells, since the neural network-based model
  may be under-trained and the similarity metric unreliable
- Input is raw counts without cell type labels, since MAP requires known cell type
  annotations to define relevant retrievals
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: Provides the customized neural network-based generative model and cell-to-cell
    similarity metric used to compute MAP for cell-type retrieval benchmarking
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model producing low-dimensional cell embeddings with multilevel
    batch-effect modeling on which MAP is computed
- name: Cell_BLAST
  role: Python package imported (as cb) to fit models and access data utilities for
    embedding and similarity computation
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Competing variational autoencoder method whose corrected embeddings are also
    scored with MAP for benchmark comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:352eb25a7eea10fac94634d092b97766b7508741ebda2cc7a959ae426c2d593d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mean-average-precision-cell-type-retrieval

## Summary

Computes mean average precision (MAP) using Cell BLAST's posterior-distance-based cell-to-cell similarity metric to quantify how well cell type relationships are preserved in low-dimensional cell embeddings after batch-effect correction or dimension reduction.

## When to use

Use when benchmarking batch-effect correction or dimension-reduction methods on scRNA-seq dataset combinations and you need a retrieval-style metric of cell-type resolution (preservation of cell type relationships) alongside a dataset-mixing metric.

## When NOT to use

- You only need to quantify dataset mixing among batches without assessing cell-type relationship preservation (the Seurat alignment score alone addresses mixing)
- Datasets have fewer than about 1000 cells, since the neural network-based model may be under-trained and the similarity metric unreliable
- Input is raw counts without cell type labels, since MAP requires known cell type annotations to define relevant retrievals

## Inputs

- scRNA-seq dataset combinations with overlapping cell types as h5ad (anndata) matrices with cell type and batch labels
- Low-dimensional cell embeddings from Cell BLAST DIRECTi or from competing methods (ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI)

## Outputs

- Per-method mean average precision (MAP) scores quantifying cell-type resolution
- batch_benchmark_results.csv containing Seurat alignment score and MAP entries for all eight evaluated methods
- Bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5

## How to apply

Fit Cell BLAST's DIRECTi deep generative model (or load corrected embeddings from competing methods such as ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, and scVI) on scRNA-seq dataset combinations with overlapping cell types, loaded as h5ad (anndata) matrices with cell type and batch labels. For each cell, use the model's posterior-distance-based cell-to-cell similarity to rank reference cells, treating shared cell type labels as relevant retrievals, and compute the mean of average precision values across query cells. Because DIRECTi explicitly models multilevel batch effect and uncertainty in similarity estimation, MAP computed on its embeddings is comparable to the paper's Fig. 2a benchmark. Report MAP together with the Seurat alignment score so that mixing of batches and preservation of cell-type resolution are assessed jointly, and repeat the scoring for the multilevel within-dataset batch setting. Assemble scores into a results table (batch_benchmark_results.csv) and render bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5.

## Related tools

- **Cell BLAST** (Provides the customized neural network-based generative model and cell-to-cell similarity metric used to compute MAP for cell-type retrieval benchmarking) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model producing low-dimensional cell embeddings with multilevel batch-effect modeling on which MAP is computed)
- **Cell_BLAST** (Python package imported (as cb) to fit models and access data utilities for embedding and similarity computation) — github.com/gao-lab/Cell_BLAST
- **scVI** (Competing variational autoencoder method whose corrected embeddings are also scored with MAP for benchmark comparison)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- The results table batch_benchmark_results.csv contains MAP entries for all eight evaluated methods (Cell BLAST, ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI) across dataset combinations and the within-dataset multilevel batch condition
- Cell BLAST MAP scores are comparable to the cell-type resolution ranking reported in Fig. 2a of the paper
- MAP is reported jointly with the Seurat alignment score so high mixing does not come at the cost of low cell-type resolution
- Reproduced bar plots match the layout of Fig. 2a and Supplementary Figs. 4-5

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition, distorting retrieval precision
- The Cell BLAST model is over-parameterized and requires a reasonable number of samples; datasets with fewer than 1000 cells may suffer from under-training issues, while a cell number of 3000 is generally sufficient
- The posterior-distance similarity metric is model-specific to Cell BLAST/DIRECTi, so MAP values are not directly comparable to metrics computed on other similarity definitions

## Evidence

- [abstract] posterior-distance-based similarity and MAP definition: "Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric"
- [discussion] multilevel batch effect modeling: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [intro] existing metrics may suffer from confounding variation: "While these metrics per se are intuitive and computationally efficient, they may not reflect cell-to-cell similarity faithfully, and suffer from nonbiological confounding variation across datasets"
- [methods] installation and import of the package: "pip install Cell-BLAST"
- [methods] input data format conversion: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [readme] how to reproduce paper results: [evidence span withheld — anchor status: external_doc]
