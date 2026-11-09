---
name: optimal-transport-matrix-factorization
description: Use when you have paired multi-omics single-cell data (e.g. scRNA+scATAC
  from scCAT-seq or 10X Multiome, CITE-seq, or tri-omic TEA-seq Muon objects) and
  need a joint cell embedding with interpretable, cell-type-specific factors.
when_to_use_negative:
- Data are unpaired multi-omics or require batch correction across batches — the paper
  states Mowgli does not handle batch correction when integrating paired multi-omics
  data
- You need a principled, data-driven choice of the number of latent dimensions — Mowgli
  does not offer a straightforward way to define k, so several values must be tested
- No GPU is available for very large datasets — optimal transport is inherently expensive
  to compute compared to the Euclidean distance, and GPU availability is required
  to scale
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Mowgli
  role: 'Core method: combines integrative NMF and entropic-regularized optimal transport
    to jointly embed paired multi-omics cells'
  repo: https://github.com/cantinilab/mowgli
- name: Python
  role: Mowgli is implemented as a Python package within the scverse ecosystem
- name: Scanpy
  role: Preprocessing (normalize_total, log1p, highly_variable_genes), kNN graph,
    Leiden clustering, UMAP, and gProfiler enrichment via enrich
- name: Muon
  role: Multi-omics container and modality-specific filtering/normalization (filter_var,
    tfidf, clr)
- name: Signac
  role: Motif Enrichment Analysis on the 100 top peaks per factor using JASPAR2022
- name: gProfiler
  role: Gene set enrichment via its API on the 150 top genes per factor (GO:CC/MF/BP,
    Azimuth, Immune SigDB)
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:caea034f0eebb29e7fd50018a74f429c129cb77babf50249549ea6e23937991e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Azimuth
  - Cobolt
  - Human Protein Atlas
  - MOFA+
  - Mowgli
  - Multigrate
  - Muon
  - Python
  - Scanpy
  - Seurat v4
  - Signac
  - gProfiler
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# optimal-transport-matrix-factorization

## Summary

Integrate paired single-cell multi-omics data with Mowgli, which combines integrative Nonnegative Matrix Factorization with entropic-regularized Optimal Transport (solved via GPU-enabled Sinkhorn) to produce cell embeddings that improve both clustering and factor interpretability.

## When to use

Use when you have paired multi-omics single-cell data (e.g. scRNA+scATAC from scCAT-seq or 10X Multiome, CITE-seq, or tri-omic TEA-seq Muon objects) and need a joint cell embedding with interpretable, cell-type-specific factors.

## When NOT to use

- Data are unpaired multi-omics or require batch correction across batches — the paper states Mowgli does not handle batch correction when integrating paired multi-omics data
- You need a principled, data-driven choice of the number of latent dimensions — Mowgli does not offer a straightforward way to define k, so several values must be tested
- No GPU is available for very large datasets — optimal transport is inherently expensive to compute compared to the Euclidean distance, and GPU availability is required to scale

## Inputs

- Paired multi-omics single-cell data as a Muon/AnnData object (scRNA-seq + scATAC-seq, optionally ADT), e.g. from scCAT-seq, PBMC 10X Multiome, OP Multiome, BM CITE, OP CITE, or TEA-seq
- Number of latent factors k and omic-specific sparsity weights (rho_rna, rho_adt, rho_atac, mu)
- GPU for the Sinkhorn algorithm on larger datasets

## Outputs

- Shared cell embedding matrix W across modalities (used for UMAP, kNN/Leiden clustering)
- Per-modality factor loading matrices H(rna), H(atac), H(adt) with ranked top genes/peaks/proteins per factor
- Clustering labels and benchmark metrics (silhouette score, ARI across resolutions, kNN purity)
- Factor specificity scores, gene set enrichment (gProfiler, Bonferroni-adjusted p < 0.05), and JASPAR2022 motif enrichments per factor

## How to apply

Preprocess each modality in a Muon object (RNA: normalize_total to 10,000 counts and Scanpy log1p; keep top 2500 highly variable genes, or 1500 for Liu scCAT-seq; ADT: Muon CLR normalization; ATAC: peak QC and optionally TF-IDF). Train Mowgli with the chosen number of latent factors (e.g. 5 on controlled cell-line settings, 50 on TEA-seq) using omic-specific cosine-distance ground costs, entropic-regularized OT solved by the GPU-enabled Sinkhorn algorithm, sparsity weights rho_rna=rho_adt=0.01, rho_atac=0.1, mu=0.001, and alternating L-BFGS optimization of H and W. Downstream, build a kNN graph (k=20, Euclidean) on the embedding with Scanpy, cluster with Leiden, and evaluate with silhouette score, ARI across resolutions, and kNN purity against other methods. For interpretability, compute factor specificity scores (a_i - b_i)/max_j a_j from mean factor weights within/outside each cell type, then run gProfiler GSEA on the top 150 genes per factor and Signac motif enrichment on the top 100 peaks per factor against JASPAR2022.

## Related tools

- **Mowgli** (Core method: combines integrative NMF and entropic-regularized optimal transport to jointly embed paired multi-omics cells) — https://github.com/cantinilab/mowgli
- **Python** (Mowgli is implemented as a Python package within the scverse ecosystem)
- **Scanpy** (Preprocessing (normalize_total, log1p, highly_variable_genes), kNN graph, Leiden clustering, UMAP, and gProfiler enrichment via enrich)
- **Muon** (Multi-omics container and modality-specific filtering/normalization (filter_var, tfidf, clr))
- **Signac** (Motif Enrichment Analysis on the 100 top peaks per factor using JASPAR2022)
- **gProfiler** (Gene set enrichment via its API on the 150 top genes per factor (GO:CC/MF/BP, Azimuth, Immune SigDB))

## Examples

```
pip install mowgli
```

## Evaluation signals

- Clustering performance (silhouette score, maximum ARI across Leiden resolutions, kNN purity with k=20) should be greater than or equal to baselines (MOFA+, Seurat v4, integrative NMF, Cobolt, Multigrate) in all controlled settings except the 90% sparse dataset
- Factor–cell-type concordance: on TEA-seq, factors should show one-to-one associations with immune cell types with high specificity scores (a_i - b_i)/max_j a_j, unlike MOFA+ and integrative NMF which aggregate multiple factors per cell type
- Mowgli should correctly separate the three cell-line populations (HCT116, HeLa-S3, K562) in the 96% sparse setting where MOFA+, Seurat v4, Cobolt, and Multigrate confuse them
- For each factor, the top 150 genes yield Bonferroni-adjusted gProfiler enrichment p < 0.05 for expected immune/GO terms, and the top 100 peaks yield coherent JASPAR2022 TF motif enrichments
- Training converges via alternating L-BFGS updates of H and W with stable Sinkhorn OT solutions on the chosen entropic-regularized objective

## Limitations

- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values should be tested
- Optimal transport is inherently expensive to compute compared to the Euclidean distance; GPU availability is required to scale to larger datasets
- Mowgli does not handle batch correction when integrating paired multi-omics data
- On the 90% sparse controlled dataset, integrative NMF attains a better silhouette score than Mowgli (though ARI and purity are equal)

## Evidence

- [abstract] Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing both clustering performance and interpretability: "Mowgli combines integrative Nonnegative Matrix Factorization and Optimal Transport, enhancing at the same time the clustering performance and interpretability of integrative Nonnegative Matrix"
- [methods] Entropic OT computed with GPU-enabled Sinkhorn: [evidence span withheld — anchor status: not_found]
- [methods] Alternating L-BFGS optimization of H and W: "we alternate between minimizing (Eq. 5) on H p ð Þ and W"
- [methods] Interpretability: 150 top genes for gProfiler, 100 top peaks for Signac: "The 150 top genes for every factor were then used as an unordered input to g Pro fi ler."
- [results] Benchmark result on controlled cell-line settings: "Indeed, in all datasets except 90% sparse, Mowgli has a performance greater or equal to that of other methods."
- [readme] Install via PyPI within the scverse ecosystem: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
