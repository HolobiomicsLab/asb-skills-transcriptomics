---
name: multi-omics-integration-benchmarking
description: Use when comparing integration methods on paired single-cell multi-omics
  data (scRNA+scATAC or RNA+ADT Muon objects) with clustering-quality metrics such
  as silhouette score, adjusted Rand Index, and kNN purity.
when_to_use_negative:
- The method being evaluated provides only clustering labels and no cell embedding
  (e.g. Seurat v4), so silhouette score cannot be computed for it and must be omitted
- Data are unpaired single-cell multi-omics batches requiring batch correction — the
  article notes Mowgli does not handle batch correction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: Mowgli
  role: Integration method under benchmarking; entropic-regularized OT + integrative
    NMF embedding with L-BFGS optimization
  repo: https://github.com/cantinilab/mowgli
- name: Seurat v4
  role: Baseline; WNN integration clustered with FindClusters (no embedding, so no
    silhouette score)
- name: MOFA+
  role: Baseline; multi-omic factor analysis embedding (15 factors; 5 for Liu) clustered
    via Scanpy/Leiden
- name: Cobolt
  role: Baseline; deep generative integration with learning rate 0.001, 200 epochs,
    30 latent dimensions
- name: Multigrate
  role: Baseline; variational integration with negative binomial loss for RNA and
    MSE for ATAC/ADT, 200 epochs
- name: Scanpy
  role: kNN graph construction (neighbors, k=20, euclidean) and Leiden clustering
    across resolutions on embeddings
- name: Muon
  role: Loading and preprocessing of the paired multi-omics MuData objects
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:1717772b2b125aef5c5a002d8591c1c25bbbd7b6f87c8588bed82c3038c82b76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Muon
  - Python
  - Scanpy
  - Seurat v4
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# multi-omics-integration-benchmarking

## Summary

Benchmark paired single-cell multi-omics integration methods (Mowgli, Seurat v4 WNN, MOFA+, integrative NMF, Cobolt, Multigrate) on embedding and clustering performance using silhouette score, ARI across Leiden resolutions, and kNN purity score.

## When to use

Use when comparing integration methods on paired single-cell multi-omics data (scRNA+scATAC or RNA+ADT Muon objects) with clustering-quality metrics such as silhouette score, adjusted Rand Index, and kNN purity.

## When NOT to use

- The method being evaluated provides only clustering labels and no cell embedding (e.g. Seurat v4), so silhouette score cannot be computed for it and must be omitted
- Data are unpaired single-cell multi-omics batches requiring batch correction — the article notes Mowgli does not handle batch correction

## Inputs

- Preprocessed Muon MuData objects for paired multi-omics datasets (e.g. Liu scCAT-seq RNA/ATAC, PBMC 10X Multiome, OP Multiome, OP CITE, BM CITE)
- Cell type ground-truth labels for ARI and kNN purity computation (e.g. Azimuth-based RNA annotation)

## Outputs

- Method x dataset x metric comparison table (e.g. benchmark_metrics.csv / controlled_benchmark_metrics.csv) with silhouette score, maximum ARI, and kNN purity per method and dataset
- Leiden clustering assignments across resolutions for each method's embedding

## How to apply

Load the preprocessed Muon objects for each dataset (e.g. Liu scCAT-seq, PBMC 10X, OP Multiome, OP CITE, BM CITE, or controlled settings derived from HCT116/HeLa-S3/K562 scCAT-seq data). Train each method with its best-performing settings: Mowgli with 50 latent dimensions (5 for Liu), entropic OT via GPU Sinkhorn with cosine-distance ground costs, and sparsity weights rho_rna=0.01, rho_adt=0.01, rho_atac=0.1, mu=0.001; MOFA+ with 15 factors; Seurat v4 WNN defaults; Cobolt (lr=0.001, 200 epochs, 30 latent dims); Multigrate (200 epochs, NB loss for RNA, MSE for ATAC/ADT, 50 latent dims); and TorchNMF integrative NMF with 30 latent dims. Build kNN graphs (k=20, euclidean) on Mowgli/integrative NMF/MOFA+ embeddings with Scanpy's neighbors and cluster with Leiden across varying resolutions; cluster Seurat v4 with FindClusters. Compute silhouette score and kNN purity with scikit-learn, and maximum ARI across resolutions with adjusted_rand_score, omitting silhouette for Seurat v4 since it provides no cell embedding. Assemble a method x dataset x metric table to judge whether the focal method's clustering is competitive with the state-of-the-art.

## Related tools

- **Mowgli** (Integration method under benchmarking; entropic-regularized OT + integrative NMF embedding with L-BFGS optimization) — https://github.com/cantinilab/mowgli
- **Seurat v4** (Baseline; WNN integration clustered with FindClusters (no embedding, so no silhouette score))
- **MOFA+** (Baseline; multi-omic factor analysis embedding (15 factors; 5 for Liu) clustered via Scanpy/Leiden)
- **Cobolt** (Baseline; deep generative integration with learning rate 0.001, 200 epochs, 30 latent dimensions)
- **Multigrate** (Baseline; variational integration with negative binomial loss for RNA and MSE for ATAC/ADT, 200 epochs)
- **Scanpy** (kNN graph construction (neighbors, k=20, euclidean) and Leiden clustering across resolutions on embeddings)
- **Muon** (Loading and preprocessing of the paired multi-omics MuData objects)

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, n_neighbors=20, metric='euclidean'); sc.tl.leiden(adata, resolution=r)
```

## Evaluation signals

- Per dataset, the focal method's kNN purity should match or exceed baseline purity (e.g. Mowgli outperforms others by purity on OP CITE and is comparable to MOFA+ and Multigrate on PBMC 10X)
- In the controlled cell-line settings, Mowgli's performance should be greater than or equal to all baselines in every setting except the 90% sparse dataset, where integrative NMF has a better silhouette but the same ARI and purity
- Seurat v4 rows must lack silhouette scores (no embedding provided) while ARI across resolutions is still reported
- The maximum ARI should be selected across Leiden resolutions at low resolutions, matching the figure-style comparison table
- In the 96% sparse dataset, only Mowgli is expected to correctly separate the three cell populations, while MOFA+, Seurat v4, Cobolt, and Multigrate confuse them

## Limitations

- Optimal transport is inherently expensive to compute compared to Euclidean distance; GPU availability is required for Mowgli to scale to larger datasets such as BM CITE (29,803 cells)
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values must be tested
- Mowgli does not handle batch correction when integrating paired multi-omics data
- Benchmark datasets lack high-quality independent biological annotation of cells; FACS-annotated paired multi-omics data are lacking

## Evidence

- [abstract] Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing at the same time the clustering performance and interpretability: "Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing at the same time the clustering performance and interpretability"
- [results] Benchmarking metrics: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: silhouette score, Adjusted Rand Index (ARI), and"
- [methods] Sinkhorn OT: [evidence span withheld — anchor status: not_found]
- [results] Seurat no silhouette: "The silhouette score cannot be com- puted for Seurat because it requires an embedding of the cells not provided by this method."
- [readme] scverse integration: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
