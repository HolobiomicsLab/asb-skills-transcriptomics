---
name: non-binary-information-quantification-delta-sil-shuffle
description: Use when benchmarking scLENS on real or simulated scRNA-seq datasets
  and you need to quantify the amount of non-binary (graded, non-zero-count) signal
  the embedding retains, e.g. to relate performance gains to dataset sparsity.
when_to_use_negative:
- Input is already a low-dimensional embedding or cluster assignment rather than a
  raw count matrix, since the test requires re-running scLENS on both the original
  and the non-zero-shuffled matrix.
- The dataset contains essentially no non-zero structure worth testing (near-binary
  data), making SILp indistinguishable from SIL and Delta-SIL uninformative.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Dimensionality reduction tool run on both the original and the non-zero-shuffled
    matrix to produce embeddings whose silhouette scores yield Delta-SIL
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS used for environment setup and embedding generation
    on CPU or CUDA-enabled GPU
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:fe5ffefef19729cd2c2be02c16977b35d054f69c061b0eb473dc045eaf289c65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - Monocle3
  - ParallelPCA
  - Scanpy
  - Seurat
  - scLENS
  - scLENSpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Non-binary information quantification via Delta-SIL shuffle test

## Summary

A permutation-based diagnostic that quantifies how much non-binary (graded expression) information scLENS captures from a scRNA-seq dataset: non-zero values are shuffled, scLENS is re-run on the permuted matrix, and Delta-SIL = SIL - SILp measures the silhouette loss caused by destroying non-binary structure.

## When to use

Use when benchmarking scLENS on real or simulated scRNA-seq datasets and you need to quantify the amount of non-binary (graded, non-zero-count) signal the embedding retains, e.g. to relate performance gains to dataset sparsity.

## When NOT to use

- Input is already a low-dimensional embedding or cluster assignment rather than a raw count matrix, since the test requires re-running scLENS on both the original and the non-zero-shuffled matrix.
- The dataset contains essentially no non-zero structure worth testing (near-binary data), making SILp indistinguishable from SIL and Delta-SIL uninformative.

## Inputs

- QC-filtered scRNA-seq count matrix (genes x cells) from real or simulated datasets (e.g. Tabula Muris-derived scDesign2 simulations, Zheng-mixture datasets)
- Ground-truth cell-type labels for silhouette computation
- scLENS embedding / clustering result from the unpermuted matrix

## Outputs

- Delta-SIL value per dataset (SIL - SILp) quantifying non-binary information
- SILp (silhouette of scLENS embedding of the non-zero-shuffled matrix)
- Per-dataset table of Delta-SIL or relative scLENS performance vs. sparsity for trend analysis

## How to apply

Start from the same QC-filtered count matrix used for the main benchmark (cells with <200 expressed genes, genes expressed in <15 cells, and cells with >5% mitochondrial reads removed). Run scLENS normally to obtain the reference embedding, then compute its silhouette (SIL) score against ground-truth labels. Create a permuted matrix by shuffling the non-zero values of the dataset while leaving the zero/non-zero pattern intact, re-run scLENS on this permuted matrix (SILp), and compute Delta-SIL = SIL - SILp. Larger Delta-SIL indicates that scLENS' embedding exploits non-binary information; trend Delta-SIL or relative ECS performance against dataset sparsity to interpret where scLENS' advantage (L2 normalization + RMT noise filtering + signal robustness test) originates.

## Related tools

- **scLENS** (Dimensionality reduction tool run on both the original and the non-zero-shuffled matrix to produce embeddings whose silhouette scores yield Delta-SIL) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS used for environment setup and embedding generation on CPU or CUDA-enabled GPU) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.plot_stability(sclens_embedding)
```

## Evaluation signals

- Delta-SIL is computed exactly as SIL - SILp, where SILp comes from re-running scLENS on a matrix whose non-zero values were shuffled while zeros are preserved.
- QC invariants hold before embedding: cells with <200 expressed genes and genes expressed in <15 cells filtered (preprocess function).
- SILp should generally be lower than SIL on datasets with graded expression; Delta-SIL near zero suggests little non-binary information.
- Delta-SIL values should vary systematically with dataset sparsity when plotted against the fraction of zeros (potentially exceeding 90% in scRNA-seq data).
- The scLENS run on the permuted matrix uses the identical pipeline/device settings as the original run so SIL differences are attributable to the shuffle alone.

## Limitations

- Requires re-running scLENS on every permuted matrix, multiplying compute cost; RMT-based filtering necessitates computing all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources.
- Operation on large data with whole genes still requires a computer with high specifications, even though median scaling reduced memory demand.
- Interpretation depends on quality ground-truth labels; on real datasets where labels were not determined independently of the assay, SIL comparisons are less reliable (Koh, Kumar, Trapnell labels were used because they were determined independently of the scRNA-seq assay).

## Evidence

- [other] Quantify non-binary information per dataset by computing Delta-SIL = SIL - SILp, shuffling non-zero values and re-running scLENS.: [evidence span withheld — anchor status: not_found]
- [abstract] Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
- [readme] During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality.: "During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
