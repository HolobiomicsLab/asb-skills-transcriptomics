---
name: mutual-information-co-expression-target-computation
description: Use when building GeneVector gene embeddings from raw scRNA-seq counts
  in a Scanpy AnnData .X matrix and a co-expression training target is needed, especially
  to recover metagenes/transcriptional programs (e.g. TGF-beta metagene 24 in TNBC
  PDX cisplatin time series).
when_to_use_negative:
- Batches differ in cell type composition and the goal is batch correction — MI-based
  embeddings do not resolve composition-driven batch effects without manual verification
- The analysis requires distinguishing negative regulation or mutually exclusive expression
  from the MI target alone
- Input is already a similarity/distance matrix or feature table rather than raw per-cell
  counts in AnnData
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Python library implementing discretization, signed MI computation from joint
    histograms, and embedding training on the MI target
  repo: https://github.com/nceglia/genevector
- name: PyTorch
  role: Neural embedding training backend; CUDA GPU backend for scatter-based MI histograms
- name: Scanpy
  role: Loads AnnData with raw counts in .X and provides seurat_v3 HVG subsetting
    as an alternative gene filter
- name: NumPy
  role: Fallback MI backend via vectorized discretization; zero-copy arrays passed
    to the Rust backend
- name: Rust
  role: Native rayon-parallel MI backend (PyO3 + numpy-rs), built with maturin develop
    --release
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:a2714c0d8cf315e6af9cb555c15a28b4756239eefd40ee7924066746c8ff47aa
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
  - Rust
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# mutational-information-co-expression-target-computation

## Summary

Compute signed mutual information between gene pairs from discretized single-cell expression as the training target for GeneVector gene embeddings, enabling low-dimensional gene representations built from co-expression dependencies.

## When to use

Use when building GeneVector gene embeddings from raw scRNA-seq counts in a Scanpy AnnData .X matrix and a co-expression training target is needed, especially to recover metagenes/transcriptional programs (e.g. TGF-beta metagene 24 in TNBC PDX cisplatin time series).

## When NOT to use

- Batches differ in cell type composition and the goal is batch correction — MI-based embeddings do not resolve composition-driven batch effects without manual verification
- The analysis requires distinguishing negative regulation or mutually exclusive expression from the MI target alone
- Input is already a similarity/distance matrix or feature table rather than raw per-cell counts in AnnData

## Inputs

- Scanpy AnnData object with raw count data in .X
- Quality-controlled gene subset (entropy_threshold=1.0 or seurat_v3 HVGs)
- Target configuration (target='mi', signed_mi=True, 10 quantile bins)
- Optional backend selection (mi_backend='auto'|'gpu'|'rust'|'numba'|'numpy') and device for CUDA

## Outputs

- Signed pairwise gene-gene MI score matrix/dict consumed as the training target
- GeneVectorDataset object prepared for embedding training
- Cached score files under ~/.genevector/cache/

## How to apply

First subset genes via GeneVectorDataset.quality_control with entropy_threshold=1.0 (or Scanpy seurat_v3 highly variable genes) to overcome single-cell sparsity. Then discretize each gene's continuous expression into integer bins via discretize_genes(): zero expression maps to bin 0 and nonzero values are split into 10 quantile-based bins. For each gene pair, build a joint histogram over (bin_A, bin_B) cell counts and compute MI(A,B) = Σ p(a,b) log2[p(a,b)/(p(a)·p(b))]; with signed_mi=True (default) the MI is multiplied by the sign of the Pearson correlation so activation-vs-repression direction is retained. Choose a backend with mi_backend='auto' (CUDA GPU ~200–1000x, Rust ~100–800x, Numba ~100–500x, NumPy ~10–30x speedup over legacy) since the all-pairs MI calculation is computationally expensive; scores are cached to ~/.genevector/cache/ keyed on the expression matrix, gene list, target, and parameters. The resulting signed-MI target feeds GeneVector model training (emb_dimension=100, 1000 iterations, threshold=1e-6, output genes.vec).

## Related tools

- **GeneVector** (Python library implementing discretization, signed MI computation from joint histograms, and embedding training on the MI target) — https://github.com/nceglia/genevector
- **PyTorch** (Neural embedding training backend; CUDA GPU backend for scatter-based MI histograms)
- **Scanpy** (Loads AnnData with raw counts in .X and provides seurat_v3 HVG subsetting as an alternative gene filter)
- **NumPy** (Fallback MI backend via vectorized discretization; zero-copy arrays passed to the Rust backend)
- **Rust** (Native rayon-parallel MI backend (PyO3 + numpy-rs), built with maturin develop --release)

## Examples

```
dataset = GeneVectorDataset(adata, target='mi', signed_mi=True, mi_backend='auto'); model = GeneVector(dataset, output_file='genes.vec', emb_dimension=100); model.train(1000, threshold=1e-6)
```

## Evaluation signals

- Every gene's expression is discretized with zero expression mapped to bin 0 and 10 quantile-based bins for nonzero values
- Signed MI values match the sign of the Pearson correlation between each gene pair (signed_mi=True)
- MI scores are cached under ~/.genevector/cache/ with keys incorporating expression matrix, gene list, target, and parameters, so re-runs with identical configuration load instantly
- Training the embedding (emb_dimension=100, 1000 iterations, threshold=1e-6) converges and model.plot() shows decreasing loss
- Downstream clustering (get_adata resolution=40) recovers known metagenes, e.g. TGF-beta-enriched metagene 24 containing ID1–ID4, EPCAM, FOXP1 in the TNBC PDX time series

## Limitations

- Empirical MI estimated from binned expression histograms is limited by the number of cells, count fidelity, and discretization strategy; no improved estimator is provided
- The all-pairs MI calculation is computationally expensive (legacy v0.2 took ~26 hours for 2,000 genes × 5,000 cells), though accelerated backends and caching mitigate this
- MI is agnostic to the direction of statistical dependency, so GeneVector cannot distinguish activation from repression without the Pearson-sign signing

## Evidence

- [methods] Discretization of gene expression into bins: "Continuous gene expression values are discretized into integer bins per gene. Zero expression maps to bin 0. Nonzero values are split into quantile-based bins (default 10)."
- [methods] MI computation from joint distribution: "MI(A, B) = \sum_{a,b} p(a,b) \log_2 \frac{p(a,b)}{p(a) \cdot p(b)}"
- [methods] Signing of MI by Pearson correlation sign: "When ``signed_mi=True`` (default), MI is multiplied by the sign of the Pearson correlation between the two genes."
- [methods] Gene QC via entropy filter: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [readme] MI backend speedups: [evidence span withheld — anchor status: external_doc]
- [discussion] MI limitation: "Mutual information calculated empirically from the histogram of binned expression counts for gene pairs is limited by the available number of cells, fidelity of the counts, and discretization strategy"
