---
name: cross-dataset-cell-embedding-evaluation
description: Use when scRNA-seq datasets with overlapping cell types must be integrated
  and you need to quantify dataset mixing vs. cell-type resolution for competing correction
  methods (e.g., ComBat, MNN, CCA, Harmony, scVI, DIRECTi).
when_to_use_negative:
- Datasets with fewer than ~1000 cells, since the over-parameterized neural-network
  model may suffer from under-training issues (a cell number of 3000 is generally
  sufficient)
- Mega-scale dataset integration where highly skewed cell-type composition may lead
  to over-correcting of batches
- The goal is cell querying/annotation against a curated reference rather than comparing
  batch-correction methods' embeddings
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Cell BLAST / DIRECTi
  role: Fits the deep generative model producing unbiased low-dimensional cell embeddings
    with multilevel batch-effect modeling; its corrected embeddings are benchmarked
    against competitors
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Competing canonical variational autoencoder method whose corrected embeddings
    are evaluated with Seurat alignment score and MAP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:2cd1b24364d806872190bd1d603ca718b590299ecb5627adf8875e39dd26f250
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell BLAST / DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# cross-dataset-cell-embedding-evaluation

## Summary

Benchmarks batch-effect correction / integration methods on scRNA-seq dataset combinations by scoring corrected cell embeddings with the Seurat alignment score (batch mixing among matched cell types) and mean average precision (MAP, cell-type relationship preservation), reproducing Cell BLAST's Fig. 2a comparison.

## When to use

Use when scRNA-seq datasets with overlapping cell types must be integrated and you need to quantify dataset mixing vs. cell-type resolution for competing correction methods (e.g., ComBat, MNN, CCA, Harmony, scVI, DIRECTi).

## When NOT to use

- Datasets with fewer than ~1000 cells, since the over-parameterized neural-network model may suffer from under-training issues (a cell number of 3000 is generally sufficient)
- Mega-scale dataset integration where highly skewed cell-type composition may lead to over-correcting of batches
- The goal is cell querying/annotation against a curated reference rather than comparing batch-correction methods' embeddings

## Inputs

- scRNA-seq dataset combinations with overlapping cell types (h5ad/anndata matrices) with cell type and batch labels
- Corrected low-dimensional cell embeddings from each evaluated method (Cell BLAST DIRECTi plus ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI)

## Outputs

- batch_benchmark_results.csv containing Seurat alignment score and MAP entries for all eight evaluated methods across dataset combinations and the within-dataset multilevel batch condition
- Bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5

## How to apply

Load the scRNA-seq dataset combinations with overlapping cell types, together with cell type and batch labels, as h5ad (anndata) matrices. Fit the Cell BLAST DIRECTi deep generative model on each combination to obtain low-dimensional cell embeddings that model multilevel batch effect, then apply competing methods (ComBat, MNN, CCA, CCA anchor, Harmony, scPhere, scVI) to the same data to produce their corrected embeddings. Compute the Seurat alignment score on each corrected embedding to quantify mixing of batches among matched cell types, and compute mean average precision (MAP) using posterior-distance-based cell-to-cell similarity to assess preservation of cell type relationships. Repeat the scoring for the multilevel within-dataset batch setting to assess handling of intra-dataset batch variation, then assemble all scores into a results table (batch_benchmark_results.csv) and render bar-plot comparisons reproducing Fig. 2a and Supplementary Figs. 4-5.

## Related tools

- **Cell BLAST / DIRECTi** (Fits the deep generative model producing unbiased low-dimensional cell embeddings with multilevel batch-effect modeling; its corrected embeddings are benchmarked against competitors) — github.com/gao-lab/Cell_BLAST
- **scVI** (Competing canonical variational autoencoder method whose corrected embeddings are evaluated with Seurat alignment score and MAP)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- batch_benchmark_results.csv contains Seurat alignment score and MAP entries for all eight evaluated methods across the dataset combinations and the within-dataset multilevel batch condition
- Cell BLAST's scores are comparable to the ranking reported in Fig. 2a: significantly better dataset mixing while maintaining high cell-type resolution
- Each method's corrected embedding yields a finite Seurat alignment score quantifying batch mixing among matched cell types
- MAP values computed from posterior-distance-based cell-to-cell similarity preserve the expected ordering of methods
- Bar plots match the layout of Fig. 2a and Supplementary Figs. 4-5

## Limitations

- Integrating datasets at mega-scale may incur over-correcting of batches with highly skewed cell-type composition
- The Cell BLAST model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues
- Existing cell-querying metrics may not faithfully reflect cell-to-cell similarity and can suffer from nonbiological confounding variation such as batch effect, which is why multilevel batch effect and uncertainty are explicitly modeled

## Evidence

- [methods] DIRECTi multilevel batch modeling: "a deep generative model (currently named **DIRECTi**)"
- [discussion] scVI posterior constraint: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution"
- [discussion] Cell BLAST robustness rationale: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [discussion] Under-training limitation: [evidence span withheld — anchor status: not_found]
- [readme] Reproduction pointer: [evidence span withheld — anchor status: external_doc]
