---
name: cluster-stability-scoring
description: Use when multiple conflicting cell-cluster annotations (e.g. Leiden at
  resolutions 1/2/3, Seurat3, Monocle3, ICGS2, cellHarmony, Azimuth label transfer)
  exist for the same cells and each cluster's stability must be scored before reconciliation.
when_to_use_negative:
- You need a consensus clustering across many (>15) annotation sets and cannot afford
  Shapley coalitional iteration — scTriangulate automatically substitutes a combined
  rank-based selection in that regime, changing the stability-selection behavior
- You want data-level integration via a joint latent space or geometric graph (e.g.
  Seurat-WNN, TotalVI) rather than decision-level reconciliation of existing cluster
  labels
- Your input is raw unnormalized counts without prior clustering — stability scoring
  requires pre-existing conflicting annotation columns in adata.obs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_4040
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: Computes the four embedded cluster stability scores (reassign, TFIDF5, TFIDF10,
    SCCAF) and combines them via Shapley value to reconcile conflicting cluster annotations
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Provides the scanpy_recipe wrapper for QC, log-CPTT normalization, HVG selection,
    and UMAP computation feeding the stability scoring
  repo: https://github.com/frankligy/scTriangulate
- name: SCCAF
  role: Single Cell Clustering Assessment Framework algorithm supplying one of the
    four embedded stability scores (multinomial logistic regression confusion-matrix
    diagonal)
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:38f625c6c792e4274ab03704920231eaf5059611911ae06022fdd0870a50d709
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - SCCAF
  - cellHarmony
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_4040
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cluster-stability-scoring

## Summary

Quantify the reliability of every cluster produced by conflicting single-cell clustering solutions using scTriangulate's four embedded stability metrics (reassign, TFIDF5, TFIDF10, SCCAF), so that a Shapley-value decision engine can mix-and-match clusters from different algorithms, modalities, and resolutions.

## When to use

Use when multiple conflicting cell-cluster annotations (e.g. Leiden at resolutions 1/2/3, Seurat3, Monocle3, ICGS2, cellHarmony, Azimuth label transfer) exist for the same cells and each cluster's stability must be scored before reconciliation.

## When NOT to use

- You need a consensus clustering across many (>15) annotation sets and cannot afford Shapley coalitional iteration — scTriangulate automatically substitutes a combined rank-based selection in that regime, changing the stability-selection behavior
- You want data-level integration via a joint latent space or geometric graph (e.g. Seurat-WNN, TotalVI) rather than decision-level reconciliation of existing cluster labels
- Your input is raw unnormalized counts without prior clustering — stability scoring requires pre-existing conflicting annotation columns in adata.obs

## Inputs

- AnnData object (adata.X log-CPTT normalized RNA or CLR-normalized ADT) with >=2 conflicting cluster annotation columns in adata.obs
- adata.obsm['X_umap'] UMAP coordinates (e.g. from scanpy preprocessing.scanpy_recipe)
- Leiden cluster annotations generated at multiple resolutions (e.g. 1, 2, 3 with n_neighbors=15, n_pcs=50) and/or external label-transfer annotations (Azimuth L2, cellHarmony, Hay et al. reference clusters)

## Outputs

- Per-cluster stability metrics (reassign, TFIDF5, TFIDF10, SCCAF scores) for every cluster of each input annotation
- Per-cell winning-fraction confidence scores and Shapley (Phi) values
- Final per-cell pruned cluster labels (e.g. exported to sctriangulate_pbmc10k_labels.csv) with unstable clusters reassigned to nearest-neighbor labels

## How to apply

Prepare an AnnData object with normalized expression (log CPTT for RNA or CLR for ADT), at least two conflicting annotation columns in adata.obs, and adata.obsm['X_umap'] (no adata.raw). Compute four embedded scores per cluster: the reassign score (reliability of a cell-to-cluster assignment compared to its own centroid via nearest-neighbor distance, top-30 markers ranked by combined t-test p-value and log-fold-change, PCA-projected centroids), TFIDF5 and TFIDF10 scores (both considered by default, added via add_metrics='tfidf5'), and the SCCAF score (multinomial logistic regression confusion-matrix diagonal). Feed these stability metrics plus the per-cell assignment matrix into Shapley-value coalitional scoring (offset 0.01 for near-tie ranks, all_or_none winner-takes-all rank adjustment) to select the winning annotation per cell, since scTriangulate picks the most stable solution through coalitional iteration rather than a consensus. Finally prune unstable clusters with win_fraction_cutoff < 0.25 (default) or fewer than abs_thresh=10 cells and reclassify their cells by nearest-neighbor strategy.

## Related tools

- **scTriangulate** (Computes the four embedded cluster stability scores (reassign, TFIDF5, TFIDF10, SCCAF) and combines them via Shapley value to reconcile conflicting cluster annotations) — https://github.com/frankligy/scTriangulate
- **scanpy** (Provides the scanpy_recipe wrapper for QC, log-CPTT normalization, HVG selection, and UMAP computation feeding the stability scoring) — https://github.com/frankligy/scTriangulate
- **SCCAF** (Single Cell Clustering Assessment Framework algorithm supplying one of the four embedded stability scores (multinomial logistic regression confusion-matrix diagonal))

## Examples

```
from sctriangulate import *; sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- All four stability scores (reassign, TFIDF5, TFIDF10, SCCAF) are computed for every cluster of every input annotation set without errors
- Unstable clusters are correctly flagged: winning fraction < win_fraction_cutoff (default 0.25) or cluster size < abs_thresh (default 10 cells)
- Every output cell carries exactly one final pruned label plus a winning-fraction confidence value, with unstable-cluster cells reassigned to nearest-neighbor cluster labels
- Final labels agree with an independent reference (e.g. Azimuth L2 annotations or Hay et al. reference clusters) and reproduce the agreement pattern reported in Supplementary Fig. 4d
- Scoring runs on all supplied annotation sets (e.g. ICGS2, Monocle3, Seurat3, curated) as in the bone marrow atlas reproduction

## Limitations

- Programmatic default analytical settings (e.g. win_fraction_cutoff 0.25, abs_thresh 10) may not be optimal for all study designs
- Integration of cluster predictions from dozens of sources significantly increases runtime; over 15 annotation sets triggers automatic substitution of Shapley selection with combined rank-based selection
- Population naming of scored/reconciled clusters remains largely non-standardized, an unmet challenge

## Evidence

- [methods] scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score.: "scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score."
- [methods] The reassign score quantifies the reliability of a cell-to-cluster assignment when compared to its own centroid (nearest neighbor distance).: "The reassign score quantifies the reliability of a cell-to-cluster assignment when compared to its own centroid (nearest neighbor distance)."
- [intro] Such stability metrics, including recent robust machine-learning approaches, such as the Single Cell Clustering Assessment Framework (SCCAF) algorithm: "Such stability metrics, including recent robust machine-learning approaches, such as the Single Cell Clustering Assessment Framework (SCCAF) algorithm"
- [abstract] Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration.: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [methods] the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined): "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined)"
- [readme] scTriangulate leverages cooperative game theory (Shapley Value) in conjunction with complimentary stability metrics (i.e. reassign score, TFIDF score and SCCAF score): "scTriangulate leverages cooperative game theory (Shapley Value) in conjunction with complimentary stability metrics (i.e. reassign score, TFIDF score and SCCAF score)"
