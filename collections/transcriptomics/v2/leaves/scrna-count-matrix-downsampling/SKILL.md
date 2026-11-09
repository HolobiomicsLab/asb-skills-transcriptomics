---
name: scrna-count-matrix-downsampling
description: Use when benchmarking dimensionality reduction or PC-selection methods
  on scRNA-seq data by simulating lower sequencing depth (e.g., downsampling deeply
  sequenced datasets, TGC > 25,000 per cell, to ~5,000 TGC/cell) and comparing reduced
  embeddings of original vs. downsampled data.
when_to_use_negative:
- Input datasets are already shallowly sequenced (average TGC near or below ~5,000
  per cell) — downsampling would push counts below a meaningful depth
- The goal is comparing clustering/biological signal recovery rather than local-structure
  preservation at reduced depth; the paper's clustering benchmark uses a different
  protocol
- Data have not passed QC — cells with <200 expressed genes, genes expressed in <15
  cells, or mitochondrial fraction >5% must be filtered first or downsampled comparisons
  are confounded
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3170
tools:
- name: scLENS
  role: Reference dimensionality reduction method whose embedding of the original
    data defines the kNN reference graph and whose RMT-based PC selection is benchmarked
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python implementation used to run scLENS on original and downsampled matrices
  repo: https://github.com/Mathbiomed/scLENSpy
- name: Seurat
  role: 'Comparison DR method: log normalization + PCA with 50 PCs, run with default
    settings on original and downsampled data'
- name: Scanpy
  role: Comparison DR method (log normalization + 50 PCs) and source of 'highly_variable_genes'
    for selecting 10,000 HVGs for scVI
- name: Monocle3
  role: 'Comparison DR method: log normalization + PCA with 50 PCs'
- name: ParallelPCA
  role: Comparison method with automatic PC selection via parallel analysis
- name: Randomly
  role: 'Comparison method: RMT-based automatic signal selection with log normalization'
- name: ACTIONet
  role: Comparison method based on matrix factorization
- name: SHARP
  role: Comparison method based on random projection
- name: scDHA
  role: 'Comparison method: autoencoder-based dimensionality reduction'
- name: scVI
  role: 'Comparison method: autoencoder run on 10,000 highly variable genes'
- name: ZINB-WaVE
  role: 'Comparison method: model-based dimensionality reduction'
- name: scTransform
  role: 'Comparison method: model-based normalization'
- name: UMAP
  role: Applied to scLENS embeddings to visualize downsampled vs. original structure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:3060dd9f70bb7ffc229cfb3303c884e8099e5f9a7c0c2df5cf4a99a5d7c2d521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - ACTIONet
  - Monocle3
  - ParallelPCA
  - Randomly
  - SHARP
  - Scanpy
  - Seurat
  - UMAP
  - ZINB-WaVE
  - scDHA
  - scLENS
  - scLENSpy
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# scrna-count-matrix-downsampling

## Summary

Downsample deeply sequenced scRNA-seq count matrices using weighted random sampling of cells, targeting a lower average total gene count (TGC) per cell, to benchmark how well dimensionality reduction methods preserve local structure (kNN-overlap) at reduced sequencing depth.

## When to use

Use when benchmarking dimensionality reduction or PC-selection methods on scRNA-seq data by simulating lower sequencing depth (e.g., downsampling deeply sequenced datasets, TGC > 25,000 per cell, to ~5,000 TGC/cell) and comparing reduced embeddings of original vs. downsampled data.

## When NOT to use

- Input datasets are already shallowly sequenced (average TGC near or below ~5,000 per cell) — downsampling would push counts below a meaningful depth
- The goal is comparing clustering/biological signal recovery rather than local-structure preservation at reduced depth; the paper's clustering benchmark uses a different protocol
- Data have not passed QC — cells with <200 expressed genes, genes expressed in <15 cells, or mitochondrial fraction >5% must be filtered first or downsampled comparisons are confounded

## Inputs

- Deeply sequenced scRNA-seq UMI or read count datasets (average TGC > 25,000 per cell), e.g., 10x matrix.mtx.gz / features.tsv.gz / barcodes.tsv.gz or CSV/JLD2 count matrices
- Gene expression profiles used to define per-cell sampling probability weights
- Per-cell QC metrics (number of expressed genes, mitochondrial fraction)
- List of dimensionality reduction methods to benchmark (12 packages)

## Outputs

- Downsampled count matrices at target average TGC of 5,000 per cell and at downsampling ratios 0.1–0.5
- Average kNN-overlap scores per method (k = 5 to 50, normalized by (n x k)/2) for original vs. downsampled data
- Normalized kNN-overlap scores for PC-selection strategies (scLENS vs. elbow vs. 95% variance) across downsampling ratios
- Results tables and plots (e.g., Fig. 6a and Fig. 6c equivalents)

## How to apply

Start with deeply sequenced UMI count datasets (average TGC > 25,000 per cell; the paper used 15 UMI and 4 read count datasets from Supplementary Table 2) and apply scLENS QC filtering: discard cells with fewer than 200 expressed genes, genes expressed in fewer than 15 cells, and cells with mitochondrial fraction above 5%. Downsample each dataset by weighted random sampling where each cell's sampling probability is proportional to its gene expression levels, targeting an average TGC of 5,000 per cell (matching typical 10x depth); for PC-selection comparisons use downsampling ratios from 0.1 to 0.5. Run the dimensionality reduction methods with default settings on both original and downsampled data (Seurat/Scanpy/Monocle3 with log normalization + 50 PCs, ParallelPCA, Randomly, ACTIONet, SHARP, scDHA, scVI with 10,000 HVGs, ZINB-WaVE, scTransform, and scLENS with scaling factor 1 + L2 normalization + RMT noise filtering + signal robustness test). Quantify structure preservation by building kNN graphs from reduced original and downsampled data, counting shared edges normalized by (n x k)/2, and averaging kNN-overlap over k = 5 to 50. For comparing PC-selection criteria, reduce log-normalized + L2-normalized downsampled data to the top 100 PCs, generate 98 kNN graphs varying PC count from 3 to 100, use the scLENS embedding of the original data as reference, and normalize scores per dataset by the maximum average kNN-overlap.

## Related tools

- **scLENS** (Reference dimensionality reduction method whose embedding of the original data defines the kNN reference graph and whose RMT-based PC selection is benchmarked) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python implementation used to run scLENS on original and downsampled matrices) — https://github.com/Mathbiomed/scLENSpy
- **Seurat** (Comparison DR method: log normalization + PCA with 50 PCs, run with default settings on original and downsampled data)
- **Scanpy** (Comparison DR method (log normalization + 50 PCs) and source of 'highly_variable_genes' for selecting 10,000 HVGs for scVI)
- **Monocle3** (Comparison DR method: log normalization + PCA with 50 PCs)
- **ParallelPCA** (Comparison method with automatic PC selection via parallel analysis)
- **Randomly** (Comparison method: RMT-based automatic signal selection with log normalization)
- **ACTIONet** (Comparison method based on matrix factorization)
- **SHARP** (Comparison method based on random projection)
- **scDHA** (Comparison method: autoencoder-based dimensionality reduction)
- **scVI** (Comparison method: autoencoder run on 10,000 highly variable genes)
- **ZINB-WaVE** (Comparison method: model-based dimensionality reduction)
- **scTransform** (Comparison method: model-based normalization)
- **UMAP** (Applied to scLENS embeddings to visualize downsampled vs. original structure)

## Examples

```
sclens_embedding = scLENS.sclens(scLENS.preprocess(scLENS.read_file('data/Z8eq.csv.gz')), device_='gpu'); CSV.write('out/pca.csv', sclens_embedding[:pca_n1])
```

## Evaluation signals

- Average kNN-overlap scores rank scLENS above the other 11 DR methods overall, matching the reported Fig. 6a result
- Downsampled datasets hit the target average TGC of ~5,000 per cell (typical 10x depth)
- kNN-overlap is computed over k = 5 to 50 (46 values) and normalized by (n x k)/2 for each kNN graph
- For the PC-selection comparison, scLENS-selected PC sets yield normalized kNN-overlap scores close to the maximum, while the elbow method under-selects and the 95% variance criterion over-selects PCs across downsampling ratios 0.1–0.5
- Per-dataset normalization by the maximum average kNN-overlap confirms scLENS remains top-ranked on both UMI and read count data

## Limitations

- scLENS's RMT-based filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- Operation on large datasets with whole genes still requires a computer with high specifications
- Selecting genes based on high variance (e.g., the 10,000 HVGs used for scVI) potentially compromises local structure contained in low-variance genes
- Benchmark validity depends on QC and weighted sampling choices; results were established for deeply sequenced data downsampled to typical 10x depth

## Evidence

- [methods] Weighted downsampling with scLENS QC: [evidence span withheld — anchor status: not_found]
- [methods] Sampling weights and TGC target: [evidence span withheld — anchor status: not_found]
- [methods] kNN-overlap normalization: [evidence span withheld — anchor status: not_found]
- [methods] PC-selection comparison protocol: [evidence span withheld — anchor status: not_found]
- [abstract] L2 normalization rationale: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [readme] QC thresholds in README: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [discussion] Memory limitation: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
