---
name: cross-condition-accuracy-comparison
description: Use when comparing a model's prediction/assignment accuracy across input
  preprocessing conditions or batch-effect regimes, e.g. testing whether raw scRNA-seq
  counts outperform normalized inputs for cell type labeling.
when_to_use_negative:
- The goal is clustering or metagene discovery rather than quantifying label-assignment
  accuracy against published annotations
- The input is not a scRNA-seq count matrix (e.g. already a feature table, bulk RNA-seq,
  or embedding coordinates) so preprocessing variants are not meaningful
- Only one input preprocessing condition exists, so no cross-condition accuracy comparison
  is possible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: GeneVector
  role: Trains gene embeddings per preprocessing condition and assigns cell types
    via CellEmbedding.phenotype_probability
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Provides AnnData objects with raw counts in .X and subsampling used to build
    batch-effect subsets
  repo: https://github.com/nceglia/genevector
- name: AnnData
  role: Container format for the expression matrices and annotated outputs of each
    condition
  repo: https://github.com/nceglia/genevector
- name: PyTorch
  role: Backend for the GeneVector neural embedding training (1000 epochs, threshold=1e-6)
- name: NumPy
  role: Numerical handling of expression-matrix preprocessing variants and accuracy
    computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:a8a4dac3d38869eabdb6d0e8b506484e25d54e2f2b54733c8603810fd0a3ad7b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-condition-accuracy-comparison

## Summary

Benchmarks GeneVector cell type prediction accuracy across multiple input preprocessing variants (raw counts, log-transformed, normalized, normalized+log-transformed) on the TICA dataset, optionally with an artificially introduced library-size batch effect, and validates each condition against published TICA annotations.

## When to use

Use when comparing a model's prediction/assignment accuracy across input preprocessing conditions or batch-effect regimes, e.g. testing whether raw scRNA-seq counts outperform normalized inputs for cell type labeling.

## When NOT to use

- The goal is clustering or metagene discovery rather than quantifying label-assignment accuracy against published annotations
- The input is not a scRNA-seq count matrix (e.g. already a feature table, bulk RNA-seq, or embedding coordinates) so preprocessing variants are not meaningful
- Only one input preprocessing condition exists, so no cross-condition accuracy comparison is possible

## Inputs

- TICA scRNA-seq dataset as a Scanpy AnnData object with raw counts in .X
- Curated marker gene sets per cell type for phenotype_probability
- Published TICA cell type annotations as ground-truth labels

## Outputs

- tica_preprocessing_benchmark.csv tabulating assignment accuracy per preprocessing condition
- Comparison figure reproducing Supplementary Fig. 1-2 (full-dataset and artificial-batch-effect accuracies)
- Per-condition trained gene embeddings (genes.vec) and annotated AnnData objects with pseudo-probability columns

## How to apply

Load the TICA scRNA-seq dataset as a Scanpy AnnData object with raw counts in .X and filter genes with GeneVectorDataset.quality_control(adata, entropy_threshold=1.0). Generate the four expression-matrix variants (raw, log-transformed, normalized, normalized+log-transformed) plus Scanpy-subsampled subsets into which an artificial library-size batch effect is introduced. For each condition, train a GeneVector model on the co-expression targets with emb_dimension=100, 1000 epochs, and threshold=1e-6, writing embeddings to genes.vec; then build CellEmbedding objects and assign cell types with phenotype_probability using curated marker gene sets per condition. Finally, compare predicted labels against the published TICA annotations, compute assignment accuracy per condition, tabulate to tica_preprocessing_benchmark.csv, and render the figure reproducing Supplementary Fig. 1-2.

## Related tools

- **GeneVector** (Trains gene embeddings per preprocessing condition and assigns cell types via CellEmbedding.phenotype_probability) — https://github.com/nceglia/genevector
- **Scanpy** (Provides AnnData objects with raw counts in .X and subsampling used to build batch-effect subsets) — https://github.com/nceglia/genevector
- **AnnData** (Container format for the expression matrices and annotated outputs of each condition) — https://github.com/nceglia/genevector
- **PyTorch** (Backend for the GeneVector neural embedding training (1000 epochs, threshold=1e-6))
- **NumPy** (Numerical handling of expression-matrix preprocessing variants and accuracy computation)

## Examples

```
adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0); model = GeneVector(GeneVectorDataset(adata), output_file="genes.vec", emb_dimension=100); model.train(1000, threshold=1e-6)
```

## Evaluation signals

- Raw-count inputs yield significantly higher cell type assignment accuracy than normalized and log-normalized inputs, matching the direction and significance pattern of Supplementary Fig. 1-2 (Supplementary Fig. 2C, D)
- Accuracy for each condition is computed against published TICA annotations, so per-condition accuracies can be diffed against the benchmark table
- Each trained embedding converges within 1000 epochs at threshold=1e-6 (model convergence check)
- phenotype_probability produces Pseudo-probability columns in adata.obs for every marker-defined cell type per condition

## Limitations

- Mutual information calculated empirically from the histogram of binned expression counts is limited by the available number of cells, fidelity of the counts, and discretization strategy
- The MI calculation is computationally expensive (legacy v0.2 took ~26 hours for 2,000 genes x 5,000 cells; Numba/Rust backends reduce this to minutes)
- Benchmarking metrics for related analyses are referenced only as Supplementary Fig. 6A-L, which are not included in the main text (reproducibility gap)

## Evidence

- [results] Raw counts significantly outperformed all other preprocessing inputs for cell type prediction accuracy by a large margin: [evidence span withheld — anchor status: not_found]
- [methods] GeneVector uses raw counts in .X: "GeneVector uses Scanpy AnnData objects with raw count data in ``.X``"
- [methods] Entropy-based quality control: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [methods] Training parameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [readme] Recommended gene subsetting: [evidence span withheld — anchor status: external_doc]
