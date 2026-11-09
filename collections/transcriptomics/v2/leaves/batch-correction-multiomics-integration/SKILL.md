---
name: batch-correction-multiomics-integration
description: Use when paired multi-omics data (e.g. scRNA-seq with scATAC-seq or CITE-seq
  via 10X Multiome) are profiled across multiple batches or centers and you need to
  train, evaluate, and compare batch-corrected versus uncorrected Mowgli embeddings.
when_to_use_negative:
- When data come from a single batch/center and no batch effect needs correcting —
  the uncorrected Mowgli model suffices
- When the integration is unimodal or unpaired, since Mowgli is designed for paired
  multi-omics data with any type and number of omics
- When you need a straightforward way to choose the number of latent dimensions k,
  which Mowgli does not offer — several values of k must be tested separately
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Mowgli
  role: Base integrative NMF + optimal transport model whose training objective is
    extended with batch-aware terms; installed via 'pip install mowgli'
  repo: https://github.com/cantinilab/mowgli
- name: Muon
  role: Loads the paired multi-omics MuData object and provides modality-specific
    preprocessing (filter_var, tf_idf, clr)
- name: Scanpy
  role: Preprocessing (normalize_total, log1p, highly_variable_genes) and downstream
    analysis including the enrich interface; supports kNN graph and clustering workflows
- name: scikit-learn
  role: Computes silhouette score and Adjusted Rand Index on corrected vs uncorrected
    embeddings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:4e3835e95831ad03d0b05def2c1933f32421307844eef78bea9346eebe72b30f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Mowgli
  - Muon
  - Python
  - Scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# batch-correction-multiomics-integration

## Summary

A reusable evaluation framework for extending Mowgli with batch correction on paired multi-omics data profiled across centers, leveraging the method's existing embedding and clustering metrics (silhouette score, ARI, purity score). The article explicitly flags batch correction of paired multi-omics integration as an open avenue for future work.

## When to use

Use when paired multi-omics data (e.g. scRNA-seq with scATAC-seq or CITE-seq via 10X Multiome) are profiled across multiple batches or centers and you need to train, evaluate, and compare batch-corrected versus uncorrected Mowgli embeddings.

## When NOT to use

- When data come from a single batch/center and no batch effect needs correcting — the uncorrected Mowgli model suffices
- When the integration is unimodal or unpaired, since Mowgli is designed for paired multi-omics data with any type and number of omics
- When you need a straightforward way to choose the number of latent dimensions k, which Mowgli does not offer — several values of k must be tested separately

## Inputs

- Paired multi-omics dataset as a Muon object (e.g. 10X Multiome RNA+ATAC or CITE-seq RNA+ADT) with per-cell batch/center annotations
- Baseline Mowgli model hyperparameters (latent dimensions k, sparsity parameters rho and mu)
- Batch/center label column per cell

## Outputs

- Uncorrected and batch-corrected cell embedding matrices (shared latent representation, e.g. 50 latent dimensions)
- Leiden cluster assignments across resolutions for both embeddings
- Comparison table of silhouette score, Adjusted Rand Index, and kNN purity score (e.g. batch_correction_comparison.csv)

## How to apply

Load the paired multi-omics dataset as a Muon object with per-cell batch annotations and apply per-modality preprocessing (QC filtering on mitochondrial proportion, expressed genes, and counts; normalize_total to 10000 plus log1p; top 2500 highly variable genes; TF-IDF for ATAC; CLR for ADT). Train a baseline uncorrected Mowgli model using entropic-regularized optimal transport (Sinkhorn with cosine ground cost) with alternating L-BFGS optimization on the dual variables. Implement a batch-corrected variant of the Mowgli objective (e.g. batch-aware ground-cost reweighting or per-batch simplex alignment) keeping all other hyperparameters identical to ensure a controlled comparison. Build kNN graphs (k=20, Euclidean) on both embeddings, cluster with Leiden across resolutions, and compute the silhouette score, Adjusted Rand Index, and kNN purity score with scikit-learn and Scanpy. Assemble the three metrics for corrected and uncorrected conditions into a comparison table (e.g. batch_correction_comparison.csv) to judge whether batch correction improves mixing without destroying biological structure.

## Related tools

- **Mowgli** (Base integrative NMF + optimal transport model whose training objective is extended with batch-aware terms; installed via 'pip install mowgli') — https://github.com/cantinilab/mowgli
- **Muon** (Loads the paired multi-omics MuData object and provides modality-specific preprocessing (filter_var, tf_idf, clr))
- **Scanpy** (Preprocessing (normalize_total, log1p, highly_variable_genes) and downstream analysis including the enrich interface; supports kNN graph and clustering workflows)
- **scikit-learn** (Computes silhouette score and Adjusted Rand Index on corrected vs uncorrected embeddings)

## Examples

```
pip install mowgli; import muon as mu; mdata = mu.read('multiome.h5mu'); from mowgli.model import MowgliModel; model = MowgliModel(n_latent_dimensions=50, rho_rna=0.01, rho_adt=0.01, rho_atac=0.1, mu=0.001); model.train(mdata)
```

## Evaluation signals

- Batch-corrected embedding should improve silhouette score relative to the uncorrected baseline if batch mixing increases
- Adjusted Rand Index across Leiden resolutions should not degrade after correction, indicating biological structure (cell populations) is preserved
- kNN purity score should remain stable or improve, confirming clusters still align with known cell-type annotations
- All hyperparameters other than the batch-aware terms (e.g. 50 latent dimensions, rho_rna=0.01/rho_adt=0.01/rho_atac=0.1, mu=0.001) must be identical between corrected and uncorrected runs for a fair comparison
- Output comparison table must contain all three metrics (silhouette, ARI, purity) for both corrected and uncorrected conditions

## Limitations

- Mowgli does not handle batch correction when integrating paired multi-omics data — this is a limitation the authors identify for future work, not an existing feature
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values must be tested
- Optimal transport is inherently expensive to compute compared to Euclidean distance; GPU availability is required for Mowgli to scale to the larger datasets presented
- Lack of high-quality biological annotation of cells in paired multi-omics benchmark datasets complicates independent validation of corrected embeddings

## Evidence

- [discussion] Mowgli does not handle batch correction when integrating paired multi-omics data: [evidence span withheld — anchor status: not_found]
- [results] Benchmark metrics: silhouette score, ARI, purity score: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: silhouette score, Adjusted Rand Index (ARI), and"
- [readme] Mowgli is a Python package integrated in the scverse ecosystem: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
- [methods] Preprocessing thresholds: 10000 counts normalization, log1p, top 2500 variable genes: [evidence span withheld — anchor status: not_found]
- [methods] Training via alternating optimization with L-BFGS: "we alternate between minimizing (Eq. 5) on H p ð Þ and W"
