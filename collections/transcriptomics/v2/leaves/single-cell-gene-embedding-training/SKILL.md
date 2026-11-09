---
name: single-cell-gene-embedding-training
description: Use when you have single-cell RNA-seq count data (AnnData/.h5ad) and
  need low-dimensional gene embeddings or unsupervised metagene discovery of co-regulated
  transcriptional programs across conditions or timepoints.
when_to_use_negative:
- Input is not raw single-cell expression counts (e.g., already a feature table, bulk
  RNA-seq matrix, or pre-computed embeddings) — GeneVector requires raw counts in
  AnnData .X
- 'The goal is to distinguish activation from repression: MI-based embeddings are
  agnostic to direction of statistical dependency (though signed_mi multiplies by
  Pearson correlation sign)'
- Batches differ substantially in cell type composition, where batch correction is
  unreliable without manual verification against cell type signatures
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: 'Core library: GeneVectorDataset discretization/MI computation, GeneVector
    model training, GeneEmbedding clustering into metagenes'
  repo: https://github.com/nceglia/genevector
- name: PyTorch
  role: Neural embedding training backend for the GeneVector model; also powers the
    CUDA GPU MI backend
- name: Scanpy
  role: Reads .h5ad AnnData, supplies seurat_v3 highly-variable-gene filtering, and
    visualizes embedded UMAPs
- name: AnnData
  role: Data container holding raw counts in .X that GeneVectorDataset ingests
- name: NumPy
  role: Vectorized fallback MI backend and array handling for zero-copy handoff to
    the Rust backend
- name: SciPy
  role: Numerical dependency for statistical computations in the pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:11cad4e88a15c516729a05b65cdf2c918a8b6257bcf3e553dc0fcef5c7b177c7
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-gene-embedding-training

## Summary

Train GeneVector dense gene embeddings from scRNA-seq mutual-information targets and cluster the resulting vectors into metagenes, enabling identification of transcriptional programs (e.g., TGF-beta metagene 24 in cisplatin-treated TNBC PDXs).

## When to use

Use when you have single-cell RNA-seq count data (AnnData/.h5ad) and need low-dimensional gene embeddings or unsupervised metagene discovery of co-regulated transcriptional programs across conditions or timepoints.

## When NOT to use

- Input is not raw single-cell expression counts (e.g., already a feature table, bulk RNA-seq matrix, or pre-computed embeddings) — GeneVector requires raw counts in AnnData .X
- The goal is to distinguish activation from repression: MI-based embeddings are agnostic to direction of statistical dependency (though signed_mi multiplies by Pearson correlation sign)
- Batches differ substantially in cell type composition, where batch correction is unreliable without manual verification against cell type signatures

## Inputs

- Scanpy AnnData (.h5ad) object with raw count data in .X
- scRNA-seq datasets such as PBMC interferon-beta stimulation, TICA atlas, HGSOC multi-site, or TNBC PDX cisplatin time-series (timepoints X1-X4)
- Optional marker gene dictionaries for downstream cell type annotation

## Outputs

- genes.vec gene embedding file (input + output weights, averaged on load)
- Clustered metagene sets from get_metagenes (e.g., metagene 24 enriched for TGF-beta signaling: ID1-4, EPCAM, FOXP1)
- CellEmbedding AnnData with UMAP and optional phenotype probabilities
- Per-metagene module scores / regression coefficients (e.g., tnbc_metagene_coefficients.csv)

## How to apply

Load raw counts in the AnnData .X matrix with Scanpy and filter informative genes via GeneVectorDataset.quality_control with entropy_threshold=1.0 (or Scanpy seurat_v3 highly-variable-gene selection). Build a GeneVectorDataset with the default signed mutual-information target (signed_mi=True), which discretizes expression into 10 quantile-based bins (zero expression in bin 0) and signs MI by Pearson correlation sign, then train a GeneVector model with emb_dimension=100 for 1000 iterations (threshold=1e-6) writing genes.vec. Construct a GeneEmbedding with vector='average' (average of input and output weights is recommended for best results), cluster gene vectors with get_adata(resolution=40), and extract metagenes with get_metagenes. Compute per-cell metagene module scores to link programs to phenotypes, e.g., fitting beta_treated vs beta_untreated regressions across timepoints and calling treatment-specific metagenes at beta_treated > 0.1, Bonferroni p = 0.001, beta_treated > beta_untreated. Rationale: mutual information combined with vector space models captures gene-gene co-expression relationships that overcome single-cell sparsity.

## Related tools

- **GeneVector** (Core library: GeneVectorDataset discretization/MI computation, GeneVector model training, GeneEmbedding clustering into metagenes) — https://github.com/nceglia/genevector
- **PyTorch** (Neural embedding training backend for the GeneVector model; also powers the CUDA GPU MI backend)
- **Scanpy** (Reads .h5ad AnnData, supplies seurat_v3 highly-variable-gene filtering, and visualizes embedded UMAPs)
- **AnnData** (Data container holding raw counts in .X that GeneVectorDataset ingests)
- **NumPy** (Vectorized fallback MI backend and array handling for zero-copy handoff to the Rust backend)
- **SciPy** (Numerical dependency for statistical computations in the pipeline)

## Examples

```
from genevector.data import GeneVectorDataset; from genevector.model import GeneVector; adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0); dataset = GeneVectorDataset(adata, signed_mi=True); model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100); model.train(1000, threshold=1e-6)
```

## Evaluation signals

- Training converges within 1000 iterations at threshold=1e-6 and writes genes.vec containing input and output weight vectors
- Gene vectors cluster into metagenes at resolution=40, and known phenotype programs are recovered (e.g., an ISG metagene in interferon-stimulated PBMCs)
- Metagene 24 is identified as TGF-beta-signaling enriched with ID1, ID2, ID3, ID4, EPCAM, and FOXP1 among its top genes
- Treatment-specific calls satisfy beta_treated > 0.1, Bonferroni-corrected p = 0.001, and beta_treated > beta_untreated
- Per-metagene coefficients, Bonferroni p-values, and specificity calls are written to the output CSV (tnbc_metagene_coefficients.csv)

## Limitations

- GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency
- Empirical MI estimation is limited by available number of cells, fidelity of counts, and discretization strategy; no improved estimator is provided
- MI calculation is computationally expensive (legacy ~26 h for 2,000 genes x 5,000 cells), though numba/Rust/CUDA backends reduce this to minutes
- Batch correction is challenging in the presence of batch-specific differences in cell type composition and must be manually verified
- Minimum embedding dimension of 50 recommended; average of input and output weight vectors is recommended over either alone

## Evidence

- [methods] QC entropy threshold: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [methods] Discretization into quantile bins: "Continuous gene expression values are discretized into integer bins per gene. Zero expression maps to bin 0. Nonzero values are split into quantile-based bins (default 10)."
- [methods] Signed MI definition: "When ``signed_mi=True`` (default), MI is multiplied by the sign of the Pearson correlation between the two genes."
- [methods] Training parameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [discussion] Metagene clustering from vectors: "The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion"
- [discussion] TNBC PDX application: "In cisplatin treated TNBC PDXs, GeneVector uncovers transcriptional signatures active in drug resistance"
- [readme] Averaged vectors recommendation: [evidence span withheld — anchor status: external_doc]
- [discussion] MI directionality limitation: "GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency"
