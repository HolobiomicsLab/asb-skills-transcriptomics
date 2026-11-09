---
name: embedding-silhouette-quality-evaluation
description: Use when an integrated scRNA-seq/scATAC-seq joint embedding has been
  produced and you need to verify that cells of the same type (or same omics) cluster
  together, e.g. to reproduce scBridge-vs-baseline comparisons.
when_to_use_negative:
- When the analysis goal is label-transfer accuracy only — per-cell-type F1-score
  is the appropriate metric, not silhouette
- When the embedding is not jointly integrated across omics (e.g., separate RNA and
  ATAC embeddings), as the omics silhouette is then meaningless as an integration
  metric
- When ground-truth cell-type labels are unavailable for the target cells, preventing
  cell-type silhouette computation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Produces the joint embedding (.obsm['Embedding']) and predicted labels (.obs['Prediction'])
    whose quality is scored
  repo: https://github.com/XLearning-SCU/scBridge
- name: scikit-learn
  role: Provides the Gaussian Mixture Model and silhouette-score computation used
    to score embedding quality
- name: harmonypy
  role: Harmonizes the joint embedding across omics before computing silhouette scores,
    as done in the paper's evaluation
  repo: https://github.com/slowkow/harmonypy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:820f992849320e7fd1746193954479673fdc05972aa96667058438da2be2ddbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - PyTorch
  - Python
  - harmonypy
  - scBridge
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# embedding-silhouette-quality-evaluation

## Summary

Evaluates the quality of a joint multi-omics embedding (e.g., .obsm['Embedding'] from scBridge integration of scRNA-seq and scATAC-seq) by computing cell-type and omics silhouette scores, optionally after harmonizing the embeddings with harmonypy. This provides an integration-quality metric complementary to label-transfer F1.

## When to use

Use when an integrated scRNA-seq/scATAC-seq joint embedding has been produced and you need to verify that cells of the same type (or same omics) cluster together, e.g. to reproduce scBridge-vs-baseline comparisons.

## When NOT to use

- When the analysis goal is label-transfer accuracy only — per-cell-type F1-score is the appropriate metric, not silhouette
- When the embedding is not jointly integrated across omics (e.g., separate RNA and ATAC embeddings), as the omics silhouette is then meaningless as an integration metric
- When ground-truth cell-type labels are unavailable for the target cells, preventing cell-type silhouette computation

## Inputs

- Integrated AnnData object with joint embeddings in .obsm['Embedding'] and transferred labels in .obs['Prediction']
- Per-cell cell-type annotations and omics (RNA/ATAC) identifiers
- Reliability scores in .obs['Reliability'] (scBridge output)

## Outputs

- Cell-type silhouette score of the joint embedding
- Omics silhouette score (RNA vs ATAC) of the joint embedding
- Optionally, harmonized embedding matrix and re-computed silhouette scores

## How to apply

After running the integration (e.g., scBridge main.py with reliability_threshold=0.95, batch_size=512, train_epoch=20, learning_rate=5e-4), extract the joint embeddings from .obsm['Embedding'] and the transferred labels from .obs['Prediction']. Compute the silhouette score of the embedding grouped by cell type and, separately, by omics (RNA vs ATAC) to quantify whether the integration removes omics separation without destroying biological structure. If the omics silhouette is high (embeddings not harmonized across modalities), harmonize the embedding with harmonypy (v0.0.5) before scoring. For the mouse atlas benchmark (~102,103 cells, 18 common cell types, 12,689 common genes), the harmonized silhouette values should match those reported in Fig. 3, alongside the scBridge 89.66% F1 versus scJoint 64.60% and Seurat 44.64%.

## Related tools

- **scBridge** (Produces the joint embedding (.obsm['Embedding']) and predicted labels (.obs['Prediction']) whose quality is scored) — https://github.com/XLearning-SCU/scBridge
- **scikit-learn** (Provides the Gaussian Mixture Model and silhouette-score computation used to score embedding quality)
- **harmonypy** (Harmonizes the joint embedding across omics before computing silhouette scores, as done in the paper's evaluation) — https://github.com/slowkow/harmonypy

## Examples

```
from sklearn.metrics import silhouette_score; s_ct = silhouette_score(adata.obsm['Embedding'], adata.obs['Prediction'])
```

## Evaluation signals

- For the Tabula-Muris / mouse atlas scATAC-seq benchmark, harmonized cell-type and omics silhouette scores should match the values reported in Fig. 3 of the paper; otherwise flag the run as a reproduction failure
- Cell-type silhouette should be substantially higher for scBridge embeddings than for scJoint and Seurat baselines, consistent with the 89.66% vs 64.60% vs 44.64% F1 ordering
- Omics silhouette should be near-zero after harmonization, indicating RNA and ATAC cells are mixed in the joint embedding
- Silhouette scores must be computed on the full ~102,103-cell mouse atlas embedding, restricted to the 18 common cell types and 12,689 common genes

## Limitations

- Silhouette scores are only a proxy for biological correctness — the paper also validates with per-cell-type F1-score and wall-clock/peak-memory scaling measurements
- Embeddings may need harmonization before scoring; scoring raw embeddings can conflate omics separation with poor integration
- Results are only interpretable when modalities share common cell types and genes (e.g., 18 types / 12,689 genes for the mouse atlas); unmatched annotations invalidate the comparison

## Evidence

- [full_text] Evaluate label transfer performance with F1-score per cell type and joint embedding quality with the silhouette score, harmonizing embeddings if required.: [evidence span withheld — anchor status: not_found]
- [intro] We evaluate the data integration performance of scBridge on seven multi-omics datasets in terms of joint embedding quality and label transfer accuracy: [evidence span withheld — anchor status: not_found]
- [readme] harmonypy is a Python package for the [Harmony] algorithm for integrating multiple high-dimensional datasets.: [evidence span withheld — anchor status: not_found]
- [full_text] Extract the transferred labels (.obs['Prediction']) and reliability scores (.obs['Reliability']) from the integrated target AnnData along with the joint embeddings (.obsm['Embedding']).: [evidence span withheld — anchor status: not_found]
