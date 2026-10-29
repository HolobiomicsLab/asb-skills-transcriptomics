---
name: leiden-clustering-of-novel-cells
description: Use when scNovel's query stage has flagged cells as novel (score > threshold
  λ = 0.5) and the flagged novel cells must be grouped into clusters for subtype identification
  via markers such as CD163.
when_to_use_negative:
- Clustering the full query set before novelty detection — Leiden should be applied
  only to cells scNovel flagged as novel (score > λ = 0.5), otherwise known reference
  types like B cell and T cell would be mixed in
- When the goal is known cell-type annotation rather than novel rare cell discovery
  — scNovel's trained classifier handles reference cell types directly
- When the query dataset has not been run through scNovel's barcode-preprocessed scoring
  mode first — there are no 'novel cells' to cluster
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scNovel
  role: 'Upstream novelty detection: its query stage with barcode preprocessing and
    λ = 0.5 threshold produces the novel-cell set that Leiden then clusters'
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Runs Leiden clustering on the subset of flagged novel cells and renders the
    UMAPs used to validate clusters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:a8ab1c66f1921451fafdfe81b9cf679df8cd4490ef9a2959b7d3ea01323dd979
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# leiden-clustering-of-novel-cells

## Summary

Subsets query cells flagged as novel by scNovel (novelty score > λ = 0.5) and partitions them into novel cell clusters with Leiden clustering in SCANPY, so that candidate novel subtypes can be recognized by marker expression.

## When to use

Use when scNovel's query stage has flagged cells as novel (score > threshold λ = 0.5) and the flagged novel cells must be grouped into clusters for subtype identification via markers such as CD163.

## When NOT to use

- Clustering the full query set before novelty detection — Leiden should be applied only to cells scNovel flagged as novel (score > λ = 0.5), otherwise known reference types like B cell and T cell would be mixed in
- When the goal is known cell-type annotation rather than novel rare cell discovery — scNovel's trained classifier handles reference cell types directly
- When the query dataset has not been run through scNovel's barcode-preprocessed scoring mode first — there are no 'novel cells' to cluster

## Inputs

- Query AnnData (e.g. GSE171524 COVID-19 matrix, 116,314 cells) normalized with target_sum=1e4 and log1p-transformed
- Per-cell novelty scores from scNovel's query stage with flag threshold λ = 0.5
- List/barcodes of cells predicted as novel by scNovel

## Outputs

- Leiden cluster labels ('leiden' in AnnData.obs) for the detected novel cells
- UMAP visualizations of cell-type annotation, patient annotation, and typical-versus-novel distributions (Figure 6A-D style, e.g. figure6_novel_detection.png)
- Marker expression summary per novel cluster identifying candidate subtypes (e.g. CD163-high clusters 3, 5, 12 as Macrophage subtypes)

## How to apply

After the scNovel query stage flags novel cells in the query AnnData (e.g. GSE171524, 116,314 cells, novelty score threshold λ = 0.5), subset only the flagged cells and run Leiden clustering on them in SCANPY to generate novel cell clusters. The cells should already be preprocessed with total-count normalization (target_sum=1e4) and log1p transformation before clustering. Cluster the novel cells only — the reference cell types (e.g. B cell, T cell) are annotated by scNovel's classifier and should not enter the Leiden step. Then inspect marker expression per Leiden cluster to name candidate subtypes: in the GSE171524 COVID-19 analysis, clusters 3, 5 and 12 showed high CD163 expression and were called candidate Macrophage subtypes. Render UMAPs of annotation, patient, and typical-versus-novel distributions (Figure 6A-D) to sanity-check the clustering.

## Related tools

- **scNovel** (Upstream novelty detection: its query stage with barcode preprocessing and λ = 0.5 threshold produces the novel-cell set that Leiden then clusters) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Runs Leiden clustering on the subset of flagged novel cells and renders the UMAPs used to validate clusters)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(novel_adata, target_sum=1e4); sc.pp.log1p(novel_adata); sc.tl.leiden(novel_adata); sc.tl.umap(novel_adata)
```

## Evaluation signals

- AnnData.obs gains a 'leiden' column populated only for cells with novelty score > 0.5; B cell and T cell clusters should be rarely represented among clustered novel cells
- AT1 and AT2 cells, known novel types in GSE171524, appear concentrated in the resulting novel clusters
- At least some clusters show coherent marker expression — in the reference analysis clusters 3, 5 and 12 show high CD163, identifying candidate Macrophage subtypes
- UMAP panels (Figure 6A-D analog) show novel-cell distribution consistent with patient annotation and typical-versus-novel labels
- Clustering is run on normalized (target_sum=1e4) log1p-transformed counts, matching scNovel's preprocessing

## Limitations

- Cluster quality depends on the upstream novelty threshold; the article notes the score function itself could be improved, which would change which cells enter Leiden
- Novel-subtype calls are candidate-level and require marker validation (e.g. CD163) — Leiden clusters alone do not confirm biology
- scNovel currently does not incorporate other biology information such as single-cell chromatin accessibility, which limits cluster interpretation

## Evidence

- [full_text] Apply Leiden clustering in scanpy on the detected novel cells to generate novel cell clusters: [evidence span withheld — anchor status: not_found]
- [full_text] Check the marker expression of the resulting novel clusters, confirming clusters 3, 5 and 12 show high CD163: [evidence span withheld — anchor status: not_found]
- [full_text] Flag cells as novel when the novelty score exceeds the threshold lambda = 0.5: [evidence span withheld — anchor status: not_found]
- [readme] Scanpy (Compatible with all versions): "- Scanpy (Compatible with all versions)"
