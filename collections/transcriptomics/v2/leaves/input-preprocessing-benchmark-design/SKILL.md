---
name: input-preprocessing-benchmark-design
description: Use when evaluating how input matrix preprocessing (raw vs log1p vs size-factor
  normalized variants) affects a downstream embedding or cell type annotation pipeline,
  or when validating that a method's raw-count requirement holds under batch perturbation.
when_to_use_negative:
- When the downstream method explicitly requires log-normalized or scaled inputs (e.g.
  methods built on Gaussian assumptions) — this benchmark specifically tests the raw-count
  regime of GeneVector's MI-based discretization.
- When the research goal is real batch-effect removal rather than benchmarking preprocessing
  sensitivity — use GeneVector's batch_correct workflow instead.
- When the dataset lacks published reference cell type annotations, since the acceptance
  criterion requires comparing predictions to ground-truth labels.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: GeneVector
  role: Trains the co-expression embedding model per preprocessing condition and assigns
    cell types via CellEmbedding.phenotype_probability
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads the TICA AnnData, applies seurat_v3-style subsetting, and generates
    the artificial library-size batch effect subsets
- name: AnnData
  role: Container format holding raw counts in .X and published annotations in .obs
    for each preprocessing variant
- name: PyTorch
  role: Backend for GeneVector neural embedding training (emb_dimension=100, 1000
    epochs)
- name: NumPy
  role: Matrix arithmetic for generating log-transformed and normalized expression
    variants
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:30be1f86b682e7846ab900ea582d38811a787fdd8901bb708de139c64b36030c
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# input-preprocessing-benchmark-design

## Summary

Design and execute a controlled benchmark comparing GeneVector cell type prediction accuracy across input preprocessing variants (raw counts, log-transformed, normalized, normalized+log-transformed) on the TICA scRNA-seq dataset, including subsets with an artificially generated library-size batch effect. This demonstrates that the entropy-based mutual information framework performs best on untransformed raw counts.

## When to use

Use when evaluating how input matrix preprocessing (raw vs log1p vs size-factor normalized variants) affects a downstream embedding or cell type annotation pipeline, or when validating that a method's raw-count requirement holds under batch perturbation.

## When NOT to use

- When the downstream method explicitly requires log-normalized or scaled inputs (e.g. methods built on Gaussian assumptions) — this benchmark specifically tests the raw-count regime of GeneVector's MI-based discretization.
- When the research goal is real batch-effect removal rather than benchmarking preprocessing sensitivity — use GeneVector's batch_correct workflow instead.
- When the dataset lacks published reference cell type annotations, since the acceptance criterion requires comparing predictions to ground-truth labels.

## Inputs

- TICA scRNA-seq dataset as a Scanpy AnnData object (.h5ad) with raw counts in .X
- Published TICA cell type annotations for validation
- Curated marker gene sets per cell type for phenotype_probability

## Outputs

- tica_preprocessing_benchmark.csv tabulating cell type assignment accuracy per preprocessing condition
- Comparison figure reproducing Supplementary Fig. 1-2 (and Supplementary Fig. 2C, D for artificial batch effect)
- Per-condition trained gene embedding files (genes.vec) and annotated cell embeddings

## How to apply

Load the TICA scRNA-seq dataset as a Scanpy AnnData object with raw counts in .X and apply GeneVector's entropy-based gene QC filter (GeneVectorDataset.quality_control with entropy_threshold=1.0). Derive four preprocessing variants of the expression matrix — raw, log-transformed, normalized, and normalized plus log-transformed — plus Scanpy-subsampled subsets into which an artificial library-size batch effect is introduced. For each condition, train a GeneVector model on co-expression targets (emb_dimension=100, 1000 epochs, threshold=1e-6) writing embeddings to genes.vec, then build cell embeddings via CellEmbedding and assign cell types with phenotype_probability using curated marker gene sets. Compare predicted labels against published TICA annotations to compute assignment accuracy per condition, tabulate into tica_preprocessing_benchmark.csv, and render figures reproducing Supplementary Fig. 1-2. The rationale is that GeneVector discretizes expression into bins and computes mutual information, so scale-distorting transforms change the discretization and can degrade embedding quality.

## Related tools

- **GeneVector** (Trains the co-expression embedding model per preprocessing condition and assigns cell types via CellEmbedding.phenotype_probability) — https://github.com/nceglia/genevector
- **Scanpy** (Loads the TICA AnnData, applies seurat_v3-style subsetting, and generates the artificial library-size batch effect subsets)
- **AnnData** (Container format holding raw counts in .X and published annotations in .obs for each preprocessing variant)
- **PyTorch** (Backend for GeneVector neural embedding training (emb_dimension=100, 1000 epochs))
- **NumPy** (Matrix arithmetic for generating log-transformed and normalized expression variants)

## Examples

```
adata = GeneVectorDataset.quality_control(sc.read_h5ad('tica.h5ad'), entropy_threshold=1.0); dataset = GeneVectorDataset(adata); model = GeneVector(dataset, output_file='genes.vec', emb_dimension=100); model.train(1000, threshold=1e-6)
```

## Evaluation signals

- Raw-count inputs yield significantly higher cell type assignment accuracy than normalized inputs, matching the direction and significance pattern of Supplementary Fig. 1-2.
- Normalized and log-normalized inputs show very poor performance on both the full and artificial-batch-effect TICA datasets (Supplementary Fig. 2C, D).
- tica_preprocessing_benchmark.csv contains one accuracy row per condition (raw, log, normalized, normalized+log) for both full and batch-perturbed subsets.
- Each condition uses identical training hyperparameters (emb_dimension=100, 1000 epochs, threshold=1e-6) so accuracy differences are attributable to preprocessing alone.

## Limitations

- GeneVector's expected-input regime is raw counts in .X; preprocessing variants are tested to characterize sensitivity, not recommended usage.
- Batch effect correction and behavior under perturbation is unreliable when batches differ in cell type composition; users must manually verify correction vectors against cell type signatures.
- Empirical MI estimation is limited by the available number of cells, fidelity of the counts, and the discretization strategy.
- Key benchmark figures are referenced only as supplementary figures (Supplementary Fig. 1-2, 2C-D), so exact accuracy values may not be recoverable from the main text.

## Evidence

- [results] Raw counts significantly outperformed all other preprocessing inputs for cell type prediction accuracy by a large margin: [evidence span withheld — anchor status: not_found]
- [methods] GeneVector uses Scanpy AnnData objects with raw count data in .X: "GeneVector uses Scanpy AnnData objects with raw count data in ``.X``"
- [readme] It's recommended to subset genes using the seurat_v3 flavor in Scanpy or GeneVector's entropy-based quality control.: [evidence span withheld — anchor status: external_doc]
- [methods] Model training snippet with concrete hyperparameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [methods] MI discretization rationale — preprocessing changes binning: "Continuous gene expression values are discretized into integer bins per gene."
