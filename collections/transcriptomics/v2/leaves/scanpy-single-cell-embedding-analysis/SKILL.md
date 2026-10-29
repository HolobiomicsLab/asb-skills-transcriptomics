---
name: scanpy-single-cell-embedding-analysis
description: Use when you have precomputed single-cell embeddings (e.g. mvTCR, RNA-only,
  TCR-only, Harmony-corrected) and need to quantify how well Leiden clusters align
  with antigen-specificity annotations on the 10x Genomics >60k-cell dataset.
when_to_use_negative:
- Input is raw count data rather than precomputed/pretrained embeddings — mvTCR training
  and preprocessing is a separate skill
- The goal is reference mapping or new-query projection (use mvTCR combined with scArches
  instead of de novo Leiden clustering)
- Data lacks pMHC specificity annotations, so NMI against specificity cannot be computed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Builds kNN graphs and runs Leiden clustering over mvTCR/shared embeddings
    in standard single-cell workflows
- name: mvTCR
  role: Provides the multiview VAE joint TCR+transcriptome embedding whose clustering
    is evaluated against antigen specificity
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Deep-learning backend used to train mvTCR models whose embeddings are clustered
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:d34425b9298cb2852de96f0633842a87fddb9a4f628d61a637c25cb897c4661c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# scanpy-single-cell-embedding-analysis

## Summary

Build kNN graphs over cell embeddings (TCR-only, RNA-only, Harmony-corrected, or mvTCR joint) in Scanpy, run Leiden clustering with a resolution sweep, and evaluate how well clusters recover pMHC antigen specificity using NMI and per-cluster antigen purity.

## When to use

Use when you have precomputed single-cell embeddings (e.g. mvTCR, RNA-only, TCR-only, Harmony-corrected) and need to quantify how well Leiden clusters align with antigen-specificity annotations on the 10x Genomics >60k-cell dataset.

## When NOT to use

- Input is raw count data rather than precomputed/pretrained embeddings — mvTCR training and preprocessing is a separate skill
- The goal is reference mapping or new-query projection (use mvTCR combined with scArches instead of de novo Leiden clustering)
- Data lacks pMHC specificity annotations, so NMI against specificity cannot be computed

## Inputs

- Pretrained cell embeddings (TCR-only, RNA-only, Harmony-corrected RNA, mvTCR joint) as Scanpy-format AnnData for the 10x dataset (>60k cells)
- Per-cell pMHC antigen specificity annotations (e.g. GILGFVFTL)

## Outputs

- Leiden cluster assignments per embedding variant across the resolution sweep
- NMI values between clusters and specificity annotations per embedding (target: mvTCR 0.535, RNA 0.456, TCR 0.408, Harmony RNA 0.212)
- Per-cluster antigen purity values (e.g. GILGFVFTL cluster at 96.5% coverage within a 99.2%-purity cluster)
- nmi_clustering_results.csv with all computed metrics

## How to apply

Load the pretrained cell embeddings from the SchubertLab/mvTCR_reproducibility repository for the preprocessed 10x dataset (>60k cells). In Scanpy, construct a k-nearest-neighbor graph over each embedding and run Leiden clustering, sweeping the resolution parameter to select the granularity that best separates specificity. Compute normalized mutual information (NMI) between Leiden cluster assignments and pMHC specificity annotations per embedding, and compute per-cluster antigen purity (fraction of cells carrying the dominant specificity, e.g. GILGFVFTL). Benchmarks to reproduce are TCR 0.408, RNA 0.456, Harmony-corrected RNA 0.212, and mvTCR 0.535 NMI, with mvTCR highest; write metrics to nmi_clustering_results.csv.

## Related tools

- **Scanpy** (Builds kNN graphs and runs Leiden clustering over mvTCR/shared embeddings in standard single-cell workflows)
- **mvTCR** (Provides the multiview VAE joint TCR+transcriptome embedding whose clustering is evaluated against antigen specificity) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Deep-learning backend used to train mvTCR models whose embeddings are clustered)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('mvtcr_embedding.h5ad'); sc.pp.neighbors(adata); sc.tl.leiden(adata, resolution=res, key_added='leiden'); nmi = sklearn.metrics.normalized_mutual_info_score(adata.obs['leiden'], adata.obs['specificity'])
```

## Evaluation signals

- Computed NMI values reproduce the reported ranking: mvTCR highest (0.535) and Harmony-corrected RNA lowest (0.212)
- Per-embedding NMI matches benchmarks: TCR 0.408, RNA 0.456, mvTCR 0.535
- The mvTCR clustering contains a GILGFVFTL-dominated cluster at 96.5% of specific cells within a 99.2%-purity cluster, versus three RNA clusters
- All metrics are written to nmi_clustering_results.csv with one row per embedding variant

## Limitations

- Leiden resolution is selected to maximize NMI on specificity, which optimizes the reported metric rather than providing an unsupervised clustering
- UMAP visualization may not be fully recreated due to reproducibility issues in the underlying library, per the mvTCR_reproducibility README
- Models retrained on different hardware/CUDA versions may diverge from the publication models; pretrained models must be obtained from Zenodo (doi.org/10.5281/zenodo.7215447)

## Evidence

- [discussion] mvTCR's representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51: "mvTCR’s representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51"
- [methods] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [readme] README: evaluation notebooks derive the shared embedding and save intermediate results for creating the figures: [evidence span withheld — anchor status: external_doc]
- [readme] README: reproducibility experiments ran with mvTCR v0.1.3 together with PyTorch v1.8.0: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
