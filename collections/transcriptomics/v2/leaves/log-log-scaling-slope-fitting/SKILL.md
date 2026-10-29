---
name: log-log-scaling-slope-fitting
description: 'Use when benchmarking computational scalability of a method: you have
  one large input dataset, need runtime as a function of size (e.g. number of cells),
  and want a fitted scaling exponent comparable across methods.'
when_to_use_negative:
- You need quality or accuracy benchmarking (denoising performance), not runtime scalability
  — use task-level quality metrics instead
- The tool cannot run on the downsampled sizes at all (e.g. SAVER/scImpute on 100,000
  cells took hours) — partial timing curves would not support a fair slope comparison
- The input is not a size-scalable single dataset; timing across heterogeneous datasets
  conflates size with dataset difficulty
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DCA
  role: Method under test; denoises each downsampled subset with its negative binomial
    noise model while runtime is recorded
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Provides filter_genes_dispersion to select the top 1,000 highly variable genes
    in each downsampled subset
- name: anndata
  role: Holds the count matrices in h5ad-formatted HDF5 files during downsampling
    and timing
  repo: https://github.com/theislab/anndata
- name: MAGIC
  role: Competing denoising method timed on subsets where tractable for runtime comparison
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Competing denoising method timed on subsets where tractable for runtime comparison
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Competing denoising method timed on subsets where tractable for runtime comparison
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:677d831e75af79712c7cca1a2221ba0b76aa4b36284f2306d6a751b49baa5a82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - MAGIC
  - SAVER
  - Scanpy
  - TensorFlow
  - anndata
  - scImpute
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# log-log-scaling-slope-fitting

## Summary

Empirically characterizes how a tool's runtime scales with input size by timing the tool on progressively downsampled datasets and fitting a linear regression in log10-log10 space to extract the scaling slope. In the DCA paper this established that DCA's runtime scales near-linearly with cell count (slope 0.66 on the 1.3M cell dataset), unlike MAGIC, SAVER and scImpute.

## When to use

Use when benchmarking computational scalability of a method: you have one large input dataset, need runtime as a function of size (e.g. number of cells), and want a fitted scaling exponent comparable across methods.

## When NOT to use

- You need quality or accuracy benchmarking (denoising performance), not runtime scalability — use task-level quality metrics instead
- The tool cannot run on the downsampled sizes at all (e.g. SAVER/scImpute on 100,000 cells took hours) — partial timing curves would not support a fair slope comparison
- The input is not a size-scalable single dataset; timing across heterogeneous datasets conflates size with dataset difficulty

## Inputs

- 1.3 million cell 10X Genomics scRNA-seq count matrix (Zheng et al., single-cell-3prime-paper)
- Downsampled count matrices at 100, 1,000, 2,000, 5,000, 10,000 and 100,000 cells (top 1,000 highly variable genes, zero-expression cells/genes removed)
- Wall-clock runtime measurements per method per subset size

## Outputs

- Fitted log-log scaling slope (expected ~0.66 for DCA, close to 1 for linear scaling)
- runtime_scaling_results.csv containing the runtimes table and fitted slope

## How to apply

Obtain the largest available input (here the 1.3 million cell 10X Genomics count matrix from the single-cell-3prime-paper repository) and downsample it to a geometric-ish spread of sizes: 100, 1,000, 2,000, 5,000, 10,000 and 100,000 cells. Restrict each subset to the top 1,000 highly variable genes selected with Scanpy's filter_genes_dispersion, then remove cells and genes with zero expression so all subsets are comparable. Denoise each subset with the tool under test (DCA with its negative binomial noise model, default 64-32-64 three-hidden-layer architecture, RMSProp optimizer, learning rate 0.001) on fixed hardware, recording wall-clock runtime; run competitors (MAGIC, SAVER, scImpute) only where tractable. Take log10 of both runtime and cell count, fit an ordinary least-squares line, and report the slope: a slope near 1 indicates linear scaling, and a slope well above 1 indicates worse-than-linear growth.

## Related tools

- **DCA** (Method under test; denoises each downsampled subset with its negative binomial noise model while runtime is recorded) — https://github.com/theislab/dca
- **Scanpy** (Provides filter_genes_dispersion to select the top 1,000 highly variable genes in each downsampled subset)
- **anndata** (Holds the count matrices in h5ad-formatted HDF5 files during downsampling and timing) — https://github.com/theislab/anndata
- **MAGIC** (Competing denoising method timed on subsets where tractable for runtime comparison) — https://github.com/pkathail/magic
- **SAVER** (Competing denoising method timed on subsets where tractable for runtime comparison) — https://github.com/mohuangx/SAVER
- **scImpute** (Competing denoising method timed on subsets where tractable for runtime comparison) — https://github.com/Vivianstats/scImpute

## Examples

```
import numpy as np; slope = np.polyfit(np.log10(n_cells), np.log10(dca_runtime_s), 1)[0]
```

## Evaluation signals

- The fitted log-log slope for DCA runtime versus cell count should be close to 1 (paper reports 0.66), consistent with linear scaling
- The downsampled sizes cover the expected spread: 100, 1,000, 2,000, 5,000, 10,000 and 100,000 cells
- Each subset is restricted to the top 1,000 highly variable genes and zero-expression cells/genes are removed before timing
- DCA denoises 100,000 cells in minutes while MAGIC, SAVER and scImpute take hours — a runtime gap of roughly an order of magnitude or more at large n
- runtime_scaling_results.csv exists and contains both the per-method/per-size runtime table and the fitted slope

## Limitations

- Scalability analysis was performed on a server with two Intel Xeon E5-2620 2.40 GHz CPUs; slope estimates are hardware-dependent and not directly comparable across machines
- Competing methods may not complete at the largest sizes, so the comparison curve can be truncated rather than full-range
- Hyperparameters were not fine-tuned for any analysis in the manuscript, so timings reflect default DCA settings (64-32-64 architecture, RMSProp, lr 0.001)

## Evidence

- [discussion] The 1.3 million cell data matrix was downsampled to 100, 1000, 2000, 5000, 10,000 and 100,000 cells: "The 1.3 million cell data matrix was downsampled to 100, 1000, 2000, 5000, 10,000 and 100,000 cells"
- [abstract] linear scaling finding: "Our method scales linearly with the number of cells and can, therefore, be applied to datasets of millions of cells."
- [discussion] HVG filtering: "the top 1000 highly variable genes are selected using “filter_genes_dispersion” function of Scanpy"
- [discussion] zero-expression removal: "First, cells and genes with zero expression are removed from the count matrix."
- [discussion] hardware: "Scalability analysis was performed on a server with two Intel Xeon E5-2620 2.40 GHz CPUs."
