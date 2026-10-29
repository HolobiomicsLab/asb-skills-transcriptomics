---
name: log-l2-normalization-preprocessing
description: Use when raw scRNA-seq count matrices (UMI or read count, e.g. CSV with
  cells x genes) must be normalized before PCA/RMT-based signal detection and you
  need to prevent log normalization from creating false variability from TGC bias.
when_to_use_negative:
- When the goal is to impute zeros rather than detect signals — the article notes
  imputation approaches 'risk biasing data, leading to false signal'
- When the analysis explicitly requires feature selection of highly variable genes,
  since selection based on high variance 'potentially compromise[s] the local structure
  contained in low variance genes'
- When data are already normalized and L2-scaled (e.g. prior scLENS output saved as
  AnnData .h5ad or PCA CSV); re-running would double-normalize the cell vectors
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Julia package that performs the QC (preprocess), log normalization with scaling
    factor 1, gene scaling, L2 normalization, and RMT-based signal counting internally
    via scLENS.read_file, scLENS.preprocess, and scLENS.sclens
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python environment/conda setup (environment.yml plus CUDA-enabled PyTorch)
    supporting GPU or CPU execution of the scLENS normalization pipeline
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:96dea4d14c423334cb25f4500cebe22301fb1f7d5261feac051c7a9ba696d491
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ACTIONet
  - Monocle3
  - ParallelPCA
  - Randomly
  - SHARP
  - Scanpy
  - Seurat
  - ZINB-WaVE
  - scDHA
  - scLENS
  - scLENSpy
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# log-l2-normalization-preprocessing

## Summary

A scRNA-seq preprocessing scheme in which log normalization (scaling factor L=1) followed by per-gene scaling and L2 normalization of cell vector lengths removes the artificial signals (TGC-bias-driven distortion) that log normalization alone introduces before dimensionality reduction.

## When to use

Use when raw scRNA-seq count matrices (UMI or read count, e.g. CSV with cells x genes) must be normalized before PCA/RMT-based signal detection and you need to prevent log normalization from creating false variability from TGC bias.

## When NOT to use

- When the goal is to impute zeros rather than detect signals — the article notes imputation approaches 'risk biasing data, leading to false signal'
- When the analysis explicitly requires feature selection of highly variable genes, since selection based on high variance 'potentially compromise[s] the local structure contained in low variance genes'
- When data are already normalized and L2-scaled (e.g. prior scLENS output saved as AnnData .h5ad or PCA CSV); re-running would double-normalize the cell vectors

## Inputs

- Raw UMI or read count matrix (CSV with first row gene names/IDs, first column cell IDs; or JLD2 containing a 'data' DataFrame with a :cell column), e.g. data/Z8eq.csv.gz
- Optional gene dictionary CSV with 'gene' and 'gene_ID' columns for renaming genes
- Optional QC thresholds: min 200 expressed genes per cell, min 15 cells per gene, max 5% mitochondrial fraction

## Outputs

- Gene-scaled, L2-normalized cell embedding-ready matrix suitable for downstream PCA/RMT noise filtering
- Eigenvalue spectrum of the cell similarity matrix G = X X^T / N with counts of signals above the Tracy-Widom (0.05) cutoff
- Signal-count comparison table (e.g. fig2_signal_detection_results.csv) showing log-normalization-only vs log+L2-normalization counts

## How to apply

First apply QC filtering (discard cells with fewer than 200 expressed genes, genes expressed in fewer than 15 cells; scLENS also drops cells with mitochondrial fraction above 5%). Then log-normalize with the scaling factor set to 1, scale each gene by subtracting its mean and dividing by its standard deviation, and finally L2-normalize each cell vector by dividing by its Euclidean length. The rationale is that log normalization amplifies gaps between zero and non-zero values and fails to uniformly normalize genes across expression levels, producing artificial signals under TGC bias; uniformizing cell vector lengths with L2 normalization eliminates these, verified by counting eigenvalues of G = X X^T / N exceeding the Marchenko-Pastur/Tracy-Widom (0.05) threshold. On the TGC-biased sparse synthetic matrix (first 400 rows halved), log normalization alone yields roughly 400 artificial signals, which drop to 0 with L2 normalization; on the real Z8eq dataset signals drop from 33 to 6.

## Related tools

- **scLENS** (Julia package that performs the QC (preprocess), log normalization with scaling factor 1, gene scaling, L2 normalization, and RMT-based signal counting internally via scLENS.read_file, scLENS.preprocess, and scLENS.sclens) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python environment/conda setup (environment.yml plus CUDA-enabled PyTorch) supporting GPU or CPU execution of the scLENS normalization pipeline) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_="cpu")
```

## Evaluation signals

- On the TGC-biased sparse synthetic matrix (2000x2000 Poisson mean-2 dense part plus 2000x8000 sparse binary part, first 400 rows halved), log normalization alone should detect approximately 400 artificial signals, and log+L2 normalization should detect 0
- On the real Z8eq.csv.gz dataset, detected signals must drop from 33 (log only) to 6 (log+L2)
- Eigenvalue counts must use all eigenvalues of G = X X^T / N with the Marchenko-Pastur fit and Tracy-Widom 0.05 probability cutoff
- All counts written to fig2_signal_detection_results.csv must reproduce the log-only vs log+L2 pattern
- QC filtering should remove cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells before normalization

## Limitations

- RMT-based noise filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- Operating on large data with whole genes still requires a computer with high specifications; median scaling replaces z-score gene scaling to reduce memory demand
- L2 normalization is a very simple approach — effective for removing TGC-bias-driven artificial signals but evaluated only within the scLENS pipeline and its benchmarks

## Evidence

- [abstract] L2 normalization removes artificial signals: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [discussion] Scaling factor of 1: "we set a scaling factor of 1 during the first step of log normalization"
- [discussion] L2 in preprocessing: "scLENS reduces signal distortion during the preprocessing step by incorporating L2 normalization"
- [readme] QC thresholds in README: "During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out"
- [readme] Data formats supported: "scLENS currently supports CSV and jld2 files for data loading"
