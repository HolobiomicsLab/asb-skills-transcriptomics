---
name: umap-visualization-of-cell-type-annotations
description: Use when a scanpy AnnData query (e.g. GSE171524, 116,314 cells) has scNovel
  novelty labels and cell-type annotations, and visual confirmation is needed that
  known types (B cell, T cell) are rarely flagged novel while AT1/AT2 are detected
  as novel.
when_to_use_negative:
- When the query set has no reference-trained annotations or novelty scores yet —
  UMAP coloring would have no scNovel labels to validate
- When quantitative AUROC comparison against other novel-detection methods is the
  goal; UMAP rendering is a qualitative validation step, not a benchmark
- When data are not single-cell expression matrices handled in scanpy AnnData format
  (e.g. bulk RNA-seq or feature tables)
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scNovel
  role: Produces the per-cell novelty scores (> lambda 0.5) and cell-type annotations
    that the UMAP panels visualize
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Computes neighbors/UMAP embeddings and renders annotation-colored UMAP plots
    on AnnData objects
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:bfe849c111c39919dde1593261f276c4a7b3217ca95272030544fec4e2c2eae2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# UMAP visualization of cell-type annotations

## Summary

Renders 2D UMAP embeddings in SCANPY to visually validate scNovel's novel-cell detection results on the GSE171524 COVID-19 dataset, overlaying cell-type annotation, patient annotation, and typical-versus-novel cell distributions (Figure 6A-D).

## When to use

Use when a scanpy AnnData query (e.g. GSE171524, 116,314 cells) has scNovel novelty labels and cell-type annotations, and visual confirmation is needed that known types (B cell, T cell) are rarely flagged novel while AT1/AT2 are detected as novel.

## When NOT to use

- When the query set has no reference-trained annotations or novelty scores yet — UMAP coloring would have no scNovel labels to validate
- When quantitative AUROC comparison against other novel-detection methods is the goal; UMAP rendering is a qualitative validation step, not a benchmark
- When data are not single-cell expression matrices handled in scanpy AnnData format (e.g. bulk RNA-seq or feature tables)

## Inputs

- Processed scanpy AnnData of the query set (e.g. GSE171524, 116,314 cells) with per-cell novelty scores/labels from scNovel
- Cell-type annotation column from the scNovel classifier (reference-trained with cross-entropy loss)
- Patient/sample annotation column (7 healthy controls and 20 COVID-19 samples)
- Leiden cluster assignments on detected novel cells

## Outputs

- UMAP panels of cell-type annotation, patient annotation, and typical-vs-novel distributions (Figure 6A-D)
- Saved figure file (e.g. figure6_novel_detection.png)
- Visual verification of AT1/AT2 as detected novel cells and CD163-high Macrophage candidate clusters 3, 5, 12

## How to apply

After the scNovel query stage flags cells as novel (novelty score > lambda = 0.5) and Leiden clustering is applied on the detected novel cells, compute neighbors and a UMAP embedding on the processed (normalize_total target_sum=1e4, log1p) AnnData. Render UMAPs colored by cell-type annotation, patient annotation, and typical-versus-novel status to reproduce Figure 6A-D; correct output shows B cell and T cell rarely predicted as novel while AT1 and AT2 appear as novel cells. Inspect novel-cluster marker expression — clusters 3, 5, and 12 with high CD163 expression are candidate Macrophage subtypes — and save the composite figure (e.g. figure6_novel_detection.png).

## Related tools

- **scNovel** (Produces the per-cell novelty scores (> lambda 0.5) and cell-type annotations that the UMAP panels visualize) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Computes neighbors/UMAP embeddings and renders annotation-colored UMAP plots on AnnData objects)

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata); sc.tl.umap(adata); sc.pl.umap(adata, color=['cell_type','patient','novel'], save='_novel_detection.png')
```

## Evaluation signals

- UMAP colored by cell type shows AT1 and AT2 cells falling in the flagged-novel region, while B cell and T cell are rarely predicted as novel
- UMAP colored by patient annotation spans all 7 healthy controls and 20 COVID-19 samples
- Novel clusters 3, 5, and 12 on the UMAP show high CD163 marker expression (candidate Macrophage subtypes)
- Figure file (figure6_novel_detection.png) is generated and matches the Figure 6A-D panel structure
- Input AnnData was preprocessed with normalize_total(target_sum=1e4) and log1p before embedding

## Limitations

- The article notes the novelty score function 'could be improved', so UMAP novel-region boundaries may shift with future scoring changes
- Visual UMAP agreement is qualitative; the article's quantitative validation is AUROC (averaged above 94%), which UMAP plots do not substitute for
- Scaling to the 116,314-cell COVID-19 query and the 1.5-million-cell reference atlas can make per-cell UMAP rendering slow/memory-heavy

## Evidence

- [results] Render UMAP visualizations of cell-type annotation, patient annotation, and typical-versus-novel cell distributions (Figure 6A-D): [evidence span withheld — anchor status: not_found]
- [results] flag cells as novel when the novelty score exceeds the threshold lambda = 0.5: [evidence span withheld — anchor status: not_found]
- [readme] scanpy normalization in README: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [abstract] three novel Macrophage subtypes on clinical COVID-19 dataset: "Applying scNovel on a clinical COVID-19 dataset, three potential novel subtypes of Macrophages are identified"
