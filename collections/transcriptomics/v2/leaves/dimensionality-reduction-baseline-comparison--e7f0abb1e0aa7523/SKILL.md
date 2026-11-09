---
name: dimensionality-reduction-baseline-comparison
description: Use when benchmarking cell-type resolution of scRNA-seq cell embeddings
  — comparing Cell BLAST/DIRECTi against DCA, PCA, and tSNE — via mean average precision
  (MAP) on query-reference pairs.
when_to_use_negative:
- Reference datasets with fewer than 1000 cells — the paper notes such small-sized
  datasets may suffer from under-training issues of the neural network-based model
- Goal is mega-scale dataset integration with highly skewed cell-type composition
  — the article warns of over-correcting batches in that regime
- Input is not a scRNA-seq query/reference pair (e.g. bulk RNA-seq or feature tables)
  — the posterior-embedding similarity metric is defined on single-cell data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cell BLAST
  role: Cell-querying framework providing the DIRECTi generative embeddings, BLAST
    retrieval, and posterior-distribution similarity metric under benchmark
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on the reference to learn low-dimensional cell
    embeddings (Cell BLAST's posterior embeddings)
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.blast
  role: Retrieves nearest reference cells for each query cell using the posterior-distribution
    distance metric
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.data
  role: Loads benchmark datasets in h5ad (AnnData) format and converts legacy h5 files
  repo: github.com/gao-lab/Cell_BLAST
- name: DCA
  role: Baseline dimension-reduction method compared in the MAP benchmark
- name: PCA
  role: Baseline dimension-reduction method compared in the MAP benchmark
- name: tSNE
  role: Baseline dimension-reduction method compared in the MAP benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:5bd41a0dbc3494693559908d54f905222716ae093ace61d8551613ef283764d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST.blast
  - Cell_BLAST.data
  - DCA
  - DIRECTi
  - PCA
  - tSNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# dimensionality-reduction-baseline-comparison

## Summary

Benchmark the cell-type resolution of Cell BLAST's DIRECTi generative embeddings against DCA, PCA, and tSNE dimension-reduction methods using mean average precision (MAP) on scRNA-seq query/reference pairs, as done in the Cell BLAST paper.

## When to use

Use when benchmarking cell-type resolution of scRNA-seq cell embeddings — comparing Cell BLAST/DIRECTi against DCA, PCA, and tSNE — via mean average precision (MAP) on query-reference pairs.

## When NOT to use

- Reference datasets with fewer than 1000 cells — the paper notes such small-sized datasets may suffer from under-training issues of the neural network-based model
- Goal is mega-scale dataset integration with highly skewed cell-type composition — the article warns of over-correcting batches in that regime
- Input is not a scRNA-seq query/reference pair (e.g. bulk RNA-seq or feature tables) — the posterior-embedding similarity metric is defined on single-cell data

## Inputs

- Reference scRNA-seq dataset in h5ad (AnnData) format
- Query scRNA-seq dataset in h5ad (AnnData) format
- Cell-type annotations on reference (and query) cells for MAP computation

## Outputs

- MAP (mean average precision) scores for Cell BLAST/DIRECTi, DCA, PCA, and tSNE
- Results table formatted like Supplementary Table 1/2
- Comparison figure summarizing cell-type resolution as in Supplementary Fig. 3

## How to apply

Load the real-world reference and query scRNA-seq datasets in h5ad (AnnData) format using Cell_BLAST.data. Fit the DIRECTi deep generative model on the reference dataset with Cell_BLAST.directi.fit_DIRECTi to learn low-dimensional cell embeddings. Embed both reference and query cells with each method: DIRECTi (Cell BLAST's posterior embeddings), DCA, PCA, and tSNE. For every query cell, retrieve nearest reference cells via Cell_BLAST.blast.BLAST using the posterior-distribution distance similarity metric. Compute mean average precision for each method with K set to 1% of the total number of cells, compile the MAP scores into a results table formatted like Supplementary Table 1/2, and summarize as in Supplementary Fig. 3.

## Related tools

- **Cell BLAST** (Cell-querying framework providing the DIRECTi generative embeddings, BLAST retrieval, and posterior-distribution similarity metric under benchmark) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on the reference to learn low-dimensional cell embeddings (Cell BLAST's posterior embeddings)) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.blast** (Retrieves nearest reference cells for each query cell using the posterior-distribution distance metric) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.data** (Loads benchmark datasets in h5ad (AnnData) format and converts legacy h5 files) — github.com/gao-lab/Cell_BLAST
- **DCA** (Baseline dimension-reduction method compared in the MAP benchmark)
- **PCA** (Baseline dimension-reduction method compared in the MAP benchmark)
- **tSNE** (Baseline dimension-reduction method compared in the MAP benchmark)

## Examples

```
import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref); bl = cb.blast.BLAST(model, ref); hits = bl.query(query, K=int(0.01 * len(ref)))
```

## Evaluation signals

- The MAP results table contains scores for all four methods (Cell BLAST/DIRECTi, DCA, PCA, tSNE)
- Cell BLAST's MAP reproduces the relative ranking over DCA, PCA, and tSNE reported in Supplementary Table 1/2 and Supplementary Fig. 3
- K for mean average precision is set to 1% of the total number of cells
- Input datasets validate as h5ad (AnnData) format and the package imports cleanly (cb.__version__ prints without error)
- Reference cells are retrieved via the posterior-distribution distance metric, not cosine or Hamming distance

## Limitations

- Cell BLAST's model is over-parameterized like all neural network-based models and requires a reasonable number of samples to train properly; ~3000 cells is generally sufficient, while datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- The scVI-style canonical variational autoencoder posterior (diagonal-covariance Gaussian) differs from Cell BLAST's posterior design, which the paper motivates via its customized cell-to-cell similarity metric

## Evidence

- [other] Cell BLAST's generative model, together with DCA, PCA, and tSNE, is among the best performing methods in cell-type resolution as measured by mean average precision (MAP).: [evidence span withheld — anchor status: not_found]
- [abstract] Cell BLAST abstract description: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [discussion] Batch-effect modeling rationale: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [discussion] Sample-size caveat: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
- [readme] Reproduction pointer: [evidence span withheld — anchor status: external_doc]
