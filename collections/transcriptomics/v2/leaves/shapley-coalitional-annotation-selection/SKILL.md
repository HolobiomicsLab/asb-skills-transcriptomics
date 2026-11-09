---
name: shapley-coalitional-annotation-selection
description: Use when a single-cell dataset has two or more conflicting per-cell annotation
  columns (different clustering algorithms, resolutions, or reference label transfers)
  that must be reconciled into one definitive per-cell cluster label.
when_to_use_negative:
- When you only have a single clustering solution or a single already-validated annotation
  with no conflicting assignments to reconcile
- When integration requires a joint latent-space or geometric graph over raw multi-modal
  data values rather than decision-level label reconciliation — scTriangulate makes
  no assumption about raw value distributions and works only on annotations
- When more than 15 annotation sets are provided and exact Shapley coalitional iteration
  is required — the program substitutes a combined rank-based selection to reduce
  runtime
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scTriangulate
  role: Core framework implementing the Shapley-value coalitional annotation selection,
    stability metric computation (reassign, TFIDF5/10, SCCAF), and winning-fraction
    pruning (lazy_run, version 0.9.0)
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: 'Preprocessing engine behind scanpy_recipe: QC filtering, log CPTT normalization,
    3000 hypervariable gene selection, PCA (n_pcs=50), neighbors (n_neighbors=15),
    and Leiden clustering at multiple resolutions'
- name: Azimuth
  role: Supplies external reference annotations (e.g., PBMC L2 labels, HLCA human-lung-v2)
    imported as a conflicting player column via scTriangulate.preprocessing.add_annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:1e7621deed5edc2e8e7cc073b3f0f53250515e97b955c5e891252fb544f6cd97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - Azimuth
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# shapley-coalitional-annotation-selection

## Summary

A decision-level integration method from scTriangulate that reconciles conflicting single-cell cluster annotations (e.g., Leiden resolutions 1/2/3 plus Azimuth label transfer) by computing per-cell Shapley values over annotation coalitions combined with stability metrics (reassign, TFIDF5, TFIDF10, SCCAF), then pruning unstable winning clusters.

## When to use

Use when a single-cell dataset has two or more conflicting per-cell annotation columns (different clustering algorithms, resolutions, or reference label transfers) that must be reconciled into one definitive per-cell cluster label.

## When NOT to use

- When you only have a single clustering solution or a single already-validated annotation with no conflicting assignments to reconcile
- When integration requires a joint latent-space or geometric graph over raw multi-modal data values rather than decision-level label reconciliation — scTriangulate makes no assumption about raw value distributions and works only on annotations
- When more than 15 annotation sets are provided and exact Shapley coalitional iteration is required — the program substitutes a combined rank-based selection to reduce runtime

## Inputs

- AnnData (h5ad) with normalized .X (log CPTT RNA or CLR ADT), no adata.raw attribute
- At least two conflicting per-cell annotation columns in adata.obs (e.g., sctri_rna_leiden_1, sctri_rna_leiden_2, sctri_rna_leiden_3, Azimuth L2 reference labels)
- adata.obsm['X_umap'] coordinates for automatic visualization

## Outputs

- Final per-cell pruned cluster labels (e.g., exported to sctriangulate_pbmc10k_labels.csv)
- Per-cell winning-fraction confidence values
- Per-cluster stability metric scores (reassign, TFIDF5, TFIDF10, SCCAF)
- UMAP visualizations of reconciled annotations in the specified output directory

## How to apply

Prepare an AnnData object whose .X is normalized (log CPTT for RNA, CLR for ADT) with at least two conflicting annotation columns in adata.obs and adata.obsm['X_umap']. Run scTriangulate (v0.9.0), which computes four embedded stability scores per cluster — the reassign score (nearest-neighbor distance to PCA-projected centroid using top-30 markers ranked by t-test p-value and log-fold-change), TFIDF5, TFIDF10, and the SCCAF multinomial logistic regression confusion-matrix diagonal score. For each cell, compute Shapley values across all annotation coalitions (using a 0.01 offset for near-tie ranks and all_or_none winner-takes-all rank adjustment) and assign the cell to the annotation with the highest Phi value. Prune raw winning clusters with winning fraction < win_fraction_cutoff (default 0.25) or fewer than abs_thresh (default 10) cells, reassigning their cells via a nearest-neighbor strategy. Note that Shapley selection is automatically substituted by combined rank-based selection when more than 15 annotation sets are provided, to reduce runtime.

## Related tools

- **scTriangulate** (Core framework implementing the Shapley-value coalitional annotation selection, stability metric computation (reassign, TFIDF5/10, SCCAF), and winning-fraction pruning (lazy_run, version 0.9.0)) — https://github.com/frankligy/scTriangulate
- **scanpy** (Preprocessing engine behind scanpy_recipe: QC filtering, log CPTT normalization, 3000 hypervariable gene selection, PCA (n_pcs=50), neighbors (n_neighbors=15), and Leiden clustering at multiple resolutions)
- **Azimuth** (Supplies external reference annotations (e.g., PBMC L2 labels, HLCA human-lung-v2) imported as a conflicting player column via scTriangulate.preprocessing.add_annotation)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- All cells receive exactly one final pruned label (binary assignment matrix X of shape I*N is fully resolved, no unassigned cells)
- No surviving cluster has winning fraction < 0.25 or fewer than 10 cells after pruning
- Per-cluster stability scores for reassign, TFIDF5, TFIDF10, and SCCAF are populated for every cluster of every input annotation before selection
- Final reconciled labels show measurable agreement with an external reference (e.g., Azimuth L2 annotations on pbmc_10k_v3)
- Output directory contains the per-cell labels CSV, winning-fraction confidence values, and UMAP plots

## Limitations

- Integrating cluster predictions from dozens of annotation sources significantly increases Shapley runtime; over 15 annotation sets triggers substitution of Shapley selection with combined rank-based selection
- Programmatic default analytical settings (e.g., win_fraction_cutoff=0.25, abs_thresh=10) may not be optimal for all study designs
- adata.raw must be deleted before running, since it interferes with how scanpy calculates marker genes for the reassign score

## Evidence

- [intro] scTriangulate uses the Shapley Value from coalitional game theory combined with cluster stability metrics to mix-and-match cellular clusters: [evidence span withheld — anchor status: not_found]
- [methods] scTriangulate uses four embedded scores: "scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score."
- [methods] the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment: "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined) and re-classified using a nearest-neighbor strategy."
- [discussion] the program will automatically substitute Shapley's selection of final clusters with a combined rank-based selection, when over 15 annotation sets are provided to reduce runtime: "the program will automatically substitute Shapley’s selection of final clusters with a combined rank-based selection, when over 15 annotation sets are provided to reduce runtime"
- [readme] at least two columns representing conflicting annotations in adata.obs, passed to query argument: [evidence span withheld — anchor status: external_doc]
