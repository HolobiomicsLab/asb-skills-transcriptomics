---
name: pc-count-selection-comparison
description: Use when evaluating how well an automatic dimension-selection criterion
  (RMT-based, elbow, or 95%-variance) preserves the local neighborhood structure of
  a scRNA-seq embedding, or when comparing PC counts across downsampled data.
when_to_use_negative:
- The input is already a low-dimensional embedding or feature table — no PCA/PC selection
  is needed
- The dataset is shallowly sequenced such that no deep-sequenced reference exists
  to build the ground-truth kNN graph
- The goal is clustering or cell-type annotation performance rather than local-structure
  (kNN-overlap) preservation
tools:
- name: scLENS
  role: Automatic PC selection via RMT-based noise filtering and signal robustness
    test; also builds the reference kNN graph on the original data
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python implementation of scLENS used to run the pipeline and save results
    (e.g. AnnData h5ad, PCA CSV)
  repo: https://github.com/Mathbiomed/scLENSpy
- name: ParallelPCA
  role: Comparator PC-selection method (parallel-analysis-based automatic PC selection)
    run with default settings
- name: Scanpy
  role: Used for log normalization, L2 normalization, 'highly_variable_genes' selection
    (for scVI comparator), and PCA with 50 PCs as baseline DR method
- name: Seurat
  role: Baseline DR method (log normalization + PCA with 50 PCs) whose implicit PC
    heuristics anchor the elbow/95%-variance comparison
- name: UMAP
  role: Optional 2D embedding applied to the selected PCs for visualization of the
    embedding quality
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:458fe832e6b070c33e5faff15b69a6aaea768f2efc083bad85a8ee6b9ab92e27
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# pc-count-selection-comparison

## Summary

Benchmark strategy from the scLENS paper for comparing how different PC-selection strategies (scLENS's RMT-based automatic selection, elbow method, 95% variance criterion) preserve local structure of scRNA-seq embeddings under downsampling, using normalized kNN-overlap scores against a reference graph.

## When to use

Use when evaluating how well an automatic dimension-selection criterion (RMT-based, elbow, or 95%-variance) preserves the local neighborhood structure of a scRNA-seq embedding, or when comparing PC counts across downsampled data.

## When NOT to use

- The input is already a low-dimensional embedding or feature table — no PCA/PC selection is needed
- The dataset is shallowly sequenced such that no deep-sequenced reference exists to build the ground-truth kNN graph
- The goal is clustering or cell-type annotation performance rather than local-structure (kNN-overlap) preservation

## Inputs

- Deeply sequenced scRNA-seq count matrix (original data, e.g. average TGC > 25,000 per cell)
- Downsampled UMI/read count matrices at ratios 0.1-0.5 (target ~5,000 TGC per cell via weighted random sampling)
- PC-count grids (3 to 100 PCs) from log-normalized, L2-normalized data

## Outputs

- Normalized kNN-overlap scores for each PC-selection strategy (scLENS, elbow, 95% variance) per dataset and downsampling ratio
- 98 kNN graphs per dataset spanning PC counts 3-100
- Reference kNN graph from scLENS applied to the original dataset
- Results table and plots for Fig. 6b-c style comparison

## How to apply

Normalize the downsampled scRNA-seq data with log normalization plus L2 normalization, reduce to the top 100 PCs, and generate 98 kNN graphs while varying the PC count from 3 to 100. Build a reference kNN graph by applying scLENS (log normalization with scaling factor 1, L2 normalization, RMT-based noise filtering, signal robustness test) to the original, deeply sequenced dataset. For downsampling ratios from 0.1 to 0.5, collect kNN-overlap scores at the PC sets chosen by scLENS, the elbow method, and the 95% variance criterion, normalizing each by the maximum average kNN-overlap score per dataset. Compute kNN-overlap as common edges between reduced and reference graphs, normalized by (n x k)/2 and averaged over k = 5 to 50 (46 values). The rationale is that the RMT-based signal robustness test avoids the subjectivity of manual elbow reading while selecting fewer, higher-quality dimensions than the 95%-variance heuristic.

## Related tools

- **scLENS** (Automatic PC selection via RMT-based noise filtering and signal robustness test; also builds the reference kNN graph on the original data) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python implementation of scLENS used to run the pipeline and save results (e.g. AnnData h5ad, PCA CSV)) — https://github.com/Mathbiomed/scLENSpy
- **ParallelPCA** (Comparator PC-selection method (parallel-analysis-based automatic PC selection) run with default settings)
- **Scanpy** (Used for log normalization, L2 normalization, 'highly_variable_genes' selection (for scVI comparator), and PCA with 50 PCs as baseline DR method)
- **Seurat** (Baseline DR method (log normalization + PCA with 50 PCs) whose implicit PC heuristics anchor the elbow/95%-variance comparison)
- **UMAP** (Optional 2D embedding applied to the selected PCs for visualization of the embedding quality)

## Examples

```
pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.save_anndata('out/test_data.h5ad', sclens_embedding)
```

## Evaluation signals

- The normalized kNN-overlap score at the scLENS-selected PC count is close to the maximum average kNN-overlap score per dataset
- The elbow method systematically under-selects PCs while the 95% variance criterion over-selects PCs across downsampling ratios 0.1-0.5 (matching reported Fig. 6c)
- kNN-overlap values are normalized by (n x k)/2 and averaged over 46 values (k = 5 to 50)
- scLENS ranks above the other 11 DR methods in average kNN-overlap on both original and downsampled data (Fig. 6a)
- The reference kNN graph is built exclusively from scLENS applied to the original, deeply sequenced data, not from downsampled data

## Limitations

- scLENS's RMT-based approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- Selection of genes based on high variance potentially compromises the local structure contained in low variance genes
- Feature selection of highly variable genes (e.g. Scanpy's 'highly_variable_genes' used for scVI) can compromise low-variance local structure
- Most DR methods require subjective user decisions to set the signal-noise threshold, reducing reliability and reproducibility

## Evidence

- [abstract] Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets"
- [discussion] scLENS provides data-driven signal-noise thresholding: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [discussion] scLENS removes low-quality signals from stochastic dropout via a signal robustness test: "scLENS removes low-quality signals arising from stochastic dropout events using a signal robustness test"
- [readme] scLENS-py preprocess function QC thresholds: "During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out"
- [readme] scLENS automatic signal detection without parameter tuning: "scLENS automatically detects biologically meaningful signals without requiring time-consuming parameter tuning."
