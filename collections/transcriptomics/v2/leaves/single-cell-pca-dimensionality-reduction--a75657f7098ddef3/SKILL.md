---
name: single-cell-pca-dimensionality-reduction
description: Use when preparing synthetic or real single-cell expression data (e.g.,
  dyntoy datasets of ~1000 cells) for trajectory inference, where methods like VIA
  consume low-dimensional PC embeddings rather than full gene-expression matrices.
when_to_use_negative:
- When VIA already has access to a precomputed embedding/PCA in the AnnData object
  and recomputing would discard prior preprocessing choices
- When the analysis requires full-resolution gene-level expression (e.g., differential
  expression on original features) rather than a reduced representation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Downstream consumer of the PC-reduced representation for trajectory inference
    via lazy-teleporting random walks
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clusters single cells on the graph built from the reduced representation;
    each node becomes a cluster in VIA's cluster-graph
- name: dynverse/dyntoy
  role: Generates the synthetic datasets (1000 cells) on which PCA is computed before
    running VIA and baselines
  repo: https://github.com/dynverse/dyntoy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:a1ab3b783da48361dbb6b9cace7015b39c06c7aef6d2d10f980fb56b195d2162
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - VIA
  - dynverse/dyntoy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-pca-dimensionality-reduction

## Summary

Reduce high-dimensional single-cell datasets to their leading principal components (e.g., first 10 PCs) as a compact, information-preserving input representation for downstream trajectory inference with VIA, PARC, and benchmarking baselines.

## When to use

Use when preparing synthetic or real single-cell expression data (e.g., dyntoy datasets of ~1000 cells) for trajectory inference, where methods like VIA consume low-dimensional PC embeddings rather than full gene-expression matrices.

## When NOT to use

- When VIA already has access to a precomputed embedding/PCA in the AnnData object and recomputing would discard prior preprocessing choices
- When the analysis requires full-resolution gene-level expression (e.g., differential expression on original features) rather than a reduced representation

## Inputs

- dyntoy synthetic single-cell trajectory datasets (e.g., Multifurc1, Cyclic1, Disconn1, Conn1; ~1000 cells, 1000 features)
- Preprocessed gene-expression count matrix (AnnData or equivalent)
- Gold-standard topology, pseudotime, and cell-fate labels from dyntoy for validation

## Outputs

- First 10 principal components per cell, stored as adata.obsm['X_pca']
- Dimensionally reduced representation suitable for VIA and baseline TI method inputs

## How to apply

Compute principal components on the preprocessed single-cell expression matrix and retain a small number of leading components — the source workflow retains the first 10 PCs per dyntoy dataset. Store the PC matrix in the AnnData structure so trajectory methods can access it directly, e.g. adata.obsm['X_pca']. Feed this PC representation into VIA (via.via_wrapper with knn=20 for connected topologies, via.via_wrapper_disconnected with root=[1,1] and preserve_disconnected=True for disconnected datasets) and into the baseline methods (Monocle3, Slingshot, STREAM, PAGA, Palantir) so all methods operate on the identical reduced representation, ensuring a fair dynverse benchmark comparison. The rationale is that PCA preserves global structure while removing noise dimensions; VIA's cluster-graph construction (PARC clustering) operates on the kNN graph built from this reduced space, so PC quality directly affects topology recovery.

## Related tools

- **VIA** (Downstream consumer of the PC-reduced representation for trajectory inference via lazy-teleporting random walks) — https://github.com/ShobiStassen/VIA
- **PARC** (Clusters single cells on the graph built from the reduced representation; each node becomes a cluster in VIA's cluster-graph)
- **dynverse/dyntoy** (Generates the synthetic datasets (1000 cells) on which PCA is computed before running VIA and baselines) — https://github.com/dynverse/dyntoy

## Examples

```
adata = sc.AnnData(data_genes); adata.obsm['X_pca'] = data.values  # then via.via_wrapper(adata, true_label, embedding, knn=20, ...)
```

## Evaluation signals

- PC matrix has the expected shape (n_cells × 10) and is stored in adata.obsm['X_pca'] before calling VIA
- VIA run on the same 10-PC input recovers the gold-standard dyntoy topology (e.g., detects the elusive M2 terminal state on four-leaf multifurcation)
- All baseline methods (Palantir, PAGA, Slingshot, STREAM, Monocle3) receive the identical reduced representation for fair comparison
- Downstream dynverse metrics (GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, F1 cell fate) computed against dyntoy gold standard validate the reduced representation did not destroy signal

## Limitations

- VIA's own paper notes that requiring extensive dimensionality reduction (e.g., via UMAP or t-SNE before parsing) can compromise global information; PCA is preferred as a lighter reduction
- Excessive or inappropriate dimensionality reduction before trajectory inference can lose global connectivity needed for correct topology recovery
- The synthetic dyntoy datasets are simplistic ('far from realistic'); PCA results there may not transfer to noisier real datasets

## Evidence

- [other] Reduce each dataset to its first 10 PCs and run VIA (pyVIA, knn=20, fixed random_seed) via via.via_wrapper for connected topologies: [evidence span withheld — anchor status: not_found]
- [methods] Load data as AnnData with PCA stored in adata.obsm['X_pca']: [evidence span withheld — anchor status: not_found]
- [intro] VIA is highly scalable with respect to number of cells (102 to >106 cells) and features, without requiring extensive dimensionality reduction or subsampling which compromise global information: "VIA is highly scalable with respect to number of cells (102 to >106 cells) and features, without requiring extensive dimensionality reduction or subsampling which compromise global information"
- [intro] Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14): "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
- [readme] Even though the model to generate the data is very simplistic (and far from realistic), it can simulate very complex trajectory models, such as large trees, convergences and loops.: [evidence span withheld — anchor status: external_doc]
