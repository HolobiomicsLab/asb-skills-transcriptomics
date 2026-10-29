---
name: library-size-batch-effect-simulation
description: Use when an scRNA-seq AnnData with raw counts in .X (e.g. TICA) must
  be stress-tested for cell type annotation robustness under a synthetic library-size
  batch effect and competing preprocessing (log, normalized, normalized+log).
when_to_use_negative:
- The goal is correcting real batches whose differences reflect cell type composition
  — the article notes batch correction is unreliable when batch-specific differences
  in cell type composition exist, and users must manually verify correction vectors
  against cell type signatures
- Input is not raw count data in AnnData .X (e.g. already log-normalized feature matrices),
  since the benchmark is specifically about restoring raw-count input to GeneVector
- The task is metagene discovery or vector arithmetic (e.g. HGSOC site-specific metagenes,
  TNBC PDX TGF-beta programs) rather than preprocessing/robustness benchmarking of
  cell type prediction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: GeneVector
  role: Trains mutual-information-based gene embeddings per preprocessing condition
    (GeneVectorDataset.quality_control, GeneVector, CellEmbedding, phenotype_probability)
    and supplies the entropy QC filter
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads AnnData with raw counts in .X, performs gene subsetting and cell subsampling,
    applies log/normalize transformations, and renders UMAPs of annotated embeddings
- name: AnnData
  role: Container format holding raw counts in .X for each preprocessing variant and
    the batch-effect subsets
- name: PyTorch
  role: Neural embedding backend used by GeneVector model.train for the co-expression
    target
- name: NumPy
  role: Vectorized numeric handling of expression matrices when constructing preprocessing
    variants
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:a6d8ea800356b39e7d197d12101b2d613b81e270e0bc2d72602f4826275e2048
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# library-size-batch-effect-simulation

## Summary

Generates Scanpy-subsampled TICA subsets carrying an artificial library-size batch effect and benchmarks GeneVector cell type prediction across four preprocessing variants (raw counts, log-transformed, normalized, normalized+log). The skill tests robustness of latent-space cell type assignment and reproduces the Supplementary Fig. 2C, D finding that raw counts outperform normalized inputs.

## When to use

Use when an scRNA-seq AnnData with raw counts in .X (e.g. TICA) must be stress-tested for cell type annotation robustness under a synthetic library-size batch effect and competing preprocessing (log, normalized, normalized+log).

## When NOT to use

- The goal is correcting real batches whose differences reflect cell type composition — the article notes batch correction is unreliable when batch-specific differences in cell type composition exist, and users must manually verify correction vectors against cell type signatures
- Input is not raw count data in AnnData .X (e.g. already log-normalized feature matrices), since the benchmark is specifically about restoring raw-count input to GeneVector
- The task is metagene discovery or vector arithmetic (e.g. HGSOC site-specific metagenes, TNBC PDX TGF-beta programs) rather than preprocessing/robustness benchmarking of cell type prediction

## Inputs

- TICA scRNA-seq dataset as a Scanpy AnnData object with raw counts in .X
- Curated per-cell-type marker gene sets (e.g. CD3D, CD3G, TRAC for T cells; LYZ, CST3, CD68 for myeloid)
- Published TICA cell type annotations used as ground truth
- Scanpy-subsampled cell subsets for artificial batch-effect introduction

## Outputs

- tica_preprocessing_benchmark.csv tabulating cell type assignment accuracy for each preprocessing condition
- Comparison figure reproducing Supplementary Fig. 1-2 (accuracy across raw/log/normalized/normalized+log, full and batch-effect subsets)
- Trained gene embedding files (genes.vec) per condition
- Predicted cell type labels and pseudo-probabilities from phenotype_probability

## How to apply

Load the TICA AnnData with raw counts in .X and filter genes with GeneVectorDataset.quality_control(adata, entropy_threshold=1.0). Build four expression-matrix variants — raw, log-transformed, normalized, normalized plus log-transformed — plus Scanpy-subsampled subsets into which an artificial library-size batch effect is introduced. For each condition, train a GeneVector model on co-expression targets (emb_dimension=100, model.train(1000, threshold=1e-6), output_file='genes.vec'), then build CellEmbedding objects and assign types via cell_embed.phenotype_probability(adata_embedded, markers) with curated marker gene sets (e.g. T Cell: CD3D/CD3G/CD3E/TRAC; Myeloid: LYZ/CST3/CD68/C1QA-C1QC). Compare predicted labels against the published TICA annotations to compute assignment accuracy per condition, tabulate in tica_preprocessing_benchmark.csv, and render the comparison figure. The rationale is that MI-based embeddings depend on count fidelity, so the benchmark should expose degradation when normalized or log-scaled inputs replace raw counts.

## Related tools

- **GeneVector** (Trains mutual-information-based gene embeddings per preprocessing condition (GeneVectorDataset.quality_control, GeneVector, CellEmbedding, phenotype_probability) and supplies the entropy QC filter) — https://github.com/nceglia/genevector
- **Scanpy** (Loads AnnData with raw counts in .X, performs gene subsetting and cell subsampling, applies log/normalize transformations, and renders UMAPs of annotated embeddings)
- **AnnData** (Container format holding raw counts in .X for each preprocessing variant and the batch-effect subsets)
- **PyTorch** (Neural embedding backend used by GeneVector model.train for the co-expression target)
- **NumPy** (Vectorized numeric handling of expression matrices when constructing preprocessing variants)

## Examples

```
adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0); sc.pp.subsample(adata, n_obs=5000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); dataset = GeneVectorDataset(adata); model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100); model.train(1000, threshold=1e-6)
```

## Evaluation signals

- Raw-count inputs yield significantly higher cell type assignment accuracy than normalized and log-normalized inputs, matching the direction and significance pattern of Supplementary Fig. 2C, D
- Predicted labels from phenotype_probability agree with published TICA annotations at the accuracies recorded in tica_preprocessing_benchmark.csv
- Entropy QC with entropy_threshold=1.0 has been applied before training in every condition
- The artificial library-size batch effect is present only in the Scanpy-subsampled subsets, with the full dataset retained as the unmodified control condition
- Training converges under the reported settings (emb_dimension=100, 1000 epochs, threshold=1e-6) for each of the four preprocessing variants

## Limitations

- Batch correction is challenging in the presence of batch-specific differences in cell type composition, so a library-size batch effect must not be conflated with composition-driven batch differences
- Empirical MI estimation is limited by the available number of cells, fidelity of the counts, and the discretization strategy; subsampling for batch-effect simulation further reduces cell counts and can degrade MI estimates
- Benchmarking metrics for related robustness analyses are referenced only as Supplementary Fig. 6A–L figures not included in the main text, a reproducibility gap when validating against published numbers

## Evidence

- [methods] GeneVector uses Scanpy AnnData objects with raw count data in ``.X``: "GeneVector uses Scanpy AnnData objects with raw count data in ``.X``"
- [readme] It's recommended to subset genes using the seurat_v3 flavor in Scanpy or GeneVector's entropy-based quality control.: [evidence span withheld — anchor status: external_doc]
- [supplementary] Raw counts significantly outperformed all other preprocessing inputs for cell type prediction accuracy by a large margin: [evidence span withheld — anchor status: not_found]
- [discussion] Batch correction is challenging in the presence of batch-specific differences in cell type composition: "Batch correction is challenging in the presence of batch-specific differences in cell type composition"
