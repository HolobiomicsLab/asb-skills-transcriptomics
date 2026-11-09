---
name: seurat-alignment-score-computation
description: Use when comparing batch-effect correction or dimension-reduction methods
  (ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI, Cell BLAST/DIRECTi) on scRNA-seq
  dataset combinations with overlapping cell types and you need a quantitative batch-mixing
  metric.
when_to_use_negative:
- Datasets have no overlapping cell types across batches, so a batch-mixing score
  among matched cell types is undefined
- The goal is cell querying/annotation with Cell BLAST itself rather than benchmarking
  embedding quality against competing batch-correction methods
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: Provides the customized neural network-based generative model and posterior-distance-based
    cell-to-cell similarity metric used to embed cells and compute MAP during benchmarking
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on each dataset combination to obtain low-dimensional
    cell embeddings modeling batch effect at multiple levels
- name: Cell_BLAST
  role: Python package imported to access Cell BLAST/DIRECTi functionality for fitting
    embeddings
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Competing variational autoencoder-based method whose corrected embeddings
    are scored with the Seurat alignment score and MAP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:96f61e029a31ae972f4b4ccebac66fb7f25418357b093d074fec46437cef78c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# seurat-alignment-score-computation

## Summary

Computes the Seurat alignment score on batch-corrected low-dimensional embeddings of scRNA-seq datasets to quantify mixing of batches among matched cell types, and pairs it with mean average precision (MAP) based on Cell BLAST's posterior-distance cell-to-cell similarity to benchmark batch-effect correction methods as in Cell BLAST's Fig. 2a comparison.

## When to use

Use when comparing batch-effect correction or dimension-reduction methods (ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI, Cell BLAST/DIRECTi) on scRNA-seq dataset combinations with overlapping cell types and you need a quantitative batch-mixing metric.

## When NOT to use

- Datasets have no overlapping cell types across batches, so a batch-mixing score among matched cell types is undefined
- The goal is cell querying/annotation with Cell BLAST itself rather than benchmarking embedding quality against competing batch-correction methods

## Inputs

- scRNA-seq dataset combinations with overlapping cell types as h5ad (anndata) matrices
- Cell type labels and batch labels per cell
- Low-dimensional cell embeddings from each batch-correction method (DIRECTi, ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI)

## Outputs

- Seurat alignment score per method per dataset combination and per batch level
- Mean average precision (MAP) per method based on posterior-distance cell-to-cell similarity
- Results table batch_benchmark_results.csv aggregating all scores
- Bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5

## How to apply

Load the scRNA-seq dataset combinations with cell type and batch labels as h5ad (anndata) matrices. For each combination, fit Cell BLAST's DIRECTi deep generative model to obtain low-dimensional embeddings that model batch effect at multiple levels, and run competing correction methods (ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI) on the same combinations to produce their corrected embeddings. Compute the Seurat alignment score on each method's corrected embedding to quantify batch mixing among matched cell types, and compute mean average precision (MAP) using Cell BLAST's posterior-distance-based cell-to-cell similarity to assess preservation of cell-type relationships. Repeat the scoring for the multilevel within-dataset batch condition to capture intra-dataset batch variation. Assemble scores into a results table (batch_benchmark_results.csv) and render bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5.

## Related tools

- **Cell BLAST** (Provides the customized neural network-based generative model and posterior-distance-based cell-to-cell similarity metric used to embed cells and compute MAP during benchmarking) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on each dataset combination to obtain low-dimensional cell embeddings modeling batch effect at multiple levels)
- **Cell_BLAST** (Python package imported to access Cell BLAST/DIRECTi functionality for fitting embeddings) — github.com/gao-lab/Cell_BLAST
- **scVI** (Competing variational autoencoder-based method whose corrected embeddings are scored with the Seurat alignment score and MAP)

## Examples

```
pip install Cell-BLAST; import Cell_BLAST as cb; print(cb.__version__)
```

## Evaluation signals

- Results table batch_benchmark_results.csv contains Seurat alignment score and MAP entries for all eight evaluated methods across the dataset combinations and the within-dataset multilevel batch condition
- Cell BLAST scores are comparable to the ranking reported in Fig. 2a, with significantly better dataset mixing while maintaining high cell-type resolution
- Each scoring run covers both the cross-dataset combination setting and the multilevel within-dataset batch setting
- MAP is computed with posterior-distance-based cell-to-cell similarity rather than a generic distance, matching the Cell BLAST metric

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- The Cell BLAST model is over-parameterized and requires a reasonable number of samples; around 3000 cells is generally sufficient, while datasets with fewer than 1000 cells may suffer from under-training issues
- Existing metrics such as approximate cosine distance or LSH Hamming distance may not faithfully reflect cell-to-cell similarity and suffer from nonbiological confounding variation such as batch effect, so score choice matters

## Evidence

- [abstract] Cell BLAST effectively models multilevel batch effect: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [discussion] Cell BLAST explicitly models batch effects for robust querying: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [methods] DIRECTi is the deep generative model behind the embeddings: "a deep generative model (currently named **DIRECTi**)"
- [readme] Reproducing the paper's benchmark results requires the rep branch: [evidence span withheld — anchor status: external_doc]
