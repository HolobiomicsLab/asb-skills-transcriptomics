---
name: single-cell-rna-seq-count-loading
description: Use when preparing scRNA-seq count matrices (e.g. .h5ad, PBMC interferon-beta
  or TICA subsets) for GeneVector, which requires raw (unnormalized) counts in AnnData
  .X before discretize_genes() and GeneVectorDataset construction.
when_to_use_negative:
- The matrix in .X is already normalized, log-transformed, or scaled — GeneVector's
  discretization and MI joint histograms expect raw counts, so loading such data here
  is the wrong choice
- The input is a bulk RNA-seq feature table or a precomputed co-expression/gene-similarity
  matrix rather than cell-by-gene single-cell counts
- The goal is the Pearson/Spearman/Jaccard/cosine target on already-processed expression
  vectors without the raw-count discretization step
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Reads the .h5ad count file into an AnnData object and performs seurat_v3 highly-variable-gene
    subsetting before GeneVectorDataset construction
- name: AnnData
  role: Container format holding raw counts in .X that GeneVectorDataset consumes
    for discretization
- name: GeneVector
  role: Provides GeneVectorDataset.quality_control (entropy_threshold=1.0 gene filtering)
    and GeneVectorDataset, which discretizes raw counts and computes pairwise co-expression
    targets
  repo: https://github.com/nceglia/genevector
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:12f72fa3169ecd563d7dd5eb4e5e65d0dd57649ce3639f1c0a80e5e83a3a15de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-rna-seq-count-loading

## Summary

Loads raw single-cell RNA-seq counts into a Scanpy AnnData object (.X) and optionally subsets genes, as the mandatory entry format for GeneVector's GeneVectorDataset before discretization and MI computation.

## When to use

Use when preparing scRNA-seq count matrices (e.g. .h5ad, PBMC interferon-beta or TICA subsets) for GeneVector, which requires raw (unnormalized) counts in AnnData .X before discretize_genes() and GeneVectorDataset construction.

## When NOT to use

- The matrix in .X is already normalized, log-transformed, or scaled — GeneVector's discretization and MI joint histograms expect raw counts, so loading such data here is the wrong choice
- The input is a bulk RNA-seq feature table or a precomputed co-expression/gene-similarity matrix rather than cell-by-gene single-cell counts
- The goal is the Pearson/Spearman/Jaccard/cosine target on already-processed expression vectors without the raw-count discretization step

## Inputs

- Raw scRNA-seq count matrix in AnnData (.h5ad or Scanpy-readable object) with counts in .X
- Gene subset specification (e.g. 1000–2000 highly variable genes via seurat_v3 flavor, or entropy_threshold=1.0 QC)

## Outputs

- Scanpy AnnData object with raw counts in .X, optionally subset to highly variable genes
- genevector.data.GeneVectorDataset with per-gene discretized integer bins ready for MI or Pearson target computation

## How to apply

Load the dataset with Scanpy's sc.read_h5ad into an AnnData object, ensuring .X holds raw counts because GeneVector discretizes these values directly (zero maps to bin 0; nonzero values are split into 10 quantile-based bins). Optionally subset genes either with Scanpy's seurat_v3 flavor highly-variable-gene selection (e.g. n_top_genes=2000, as in the TICA 23,764-cell subset) or with GeneVectorDataset.quality_control(adata, entropy_threshold=1.0). Pass the AnnData directly into GeneVectorDataset(adata) — the constructor triggers per-gene discretization and the O(G^2 x N) pairwise target computation, with backend auto-dispatch (CUDA > Rust > Numba > NumPy) when target='mi'. Rationale: the mutual-information target is computed from binned raw expression, so normalized, log-transformed, or scaled matrices in .X would distort the joint histograms and downstream MI estimates.

## Related tools

- **Scanpy** (Reads the .h5ad count file into an AnnData object and performs seurat_v3 highly-variable-gene subsetting before GeneVectorDataset construction)
- **AnnData** (Container format holding raw counts in .X that GeneVectorDataset consumes for discretization)
- **GeneVector** (Provides GeneVectorDataset.quality_control (entropy_threshold=1.0 gene filtering) and GeneVectorDataset, which discretizes raw counts and computes pairwise co-expression targets) — https://github.com/nceglia/genevector

## Examples

```
import scanpy as sc; adata = sc.read_h5ad("your_data.h5ad"); sc.pp.highly_variable_genes(adata, flavor="seurat_v3", n_top_genes=2000); adata = adata[:, adata.var.highly_variable]
```

## Evaluation signals

- adata.X contains integer raw counts (not log-normalized values), verified before calling GeneVectorDataset
- Gene subset size matches intent: 2,000 HVGs for the TICA subset or 1,000 HVGs for the pbmc_ifnb run, selected via seurat_v3 flavor or entropy_threshold=1.0 QC
- After discretization, zero expression maps to bin 0 and nonzero values fall into 10 quantile-based bins per gene
- genes.vec is produced on disk containing 100-dimensional embedding vectors for the loaded genes after training (emb_dimension=100, 1000 iterations, threshold=1e-6)
- Computed MI scores are cached under ~/.genevector/cache/ with keys incorporating the expression matrix, gene list, target function, and parameters

## Limitations

- Empirical MI estimated from binned raw counts is limited by the available number of cells, fidelity of the counts, and the discretization strategy — no improved estimator is provided
- Legacy (v0.2) MI computation on 2,000 genes × 5,000 cells takes ~26 hours; accelerated backends (Rust ~1-2 minutes, Numba ~2-3 minutes) require extra installation steps (maturin develop --release or pip install genevector[fast])
- Loading substandard or heavily filtered matrices propagates directly into the O(G^2 x N) pairwise target computation, so gene QC (seurat_v3 or entropy threshold 1.0) should be done at load time

## Evidence

- [readme] GeneVector uses Scanpy AnnData objects and requires raw count data in the `.X` matrix. It's recommended to subset genes using the `seurat_v3` flavor in Scanpy or GeneVector's entropy-based quality control.: [evidence span withheld — anchor status: external_doc]
- [methods] Zero expression maps to bin 0. Nonzero values are split into quantile-based bins (default 10).: "Zero expression maps to bin 0. Nonzero values are split into quantile-based bins (default 10)."
- [readme] sc.pp.highly_variable_genes(adata, flavor="seurat_v3", n_top_genes=2000): [evidence span withheld — anchor status: external_doc]
- [methods] adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0): "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
