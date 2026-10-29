---
name: singlesample-gsea-scoring
description: Use when per-cell scRNA-seq counts (h5ad AnnData) must be scored against
  a two-part UP/DN .gmt signature with a ranked, single-sample method, especially
  when sparsity blurs signal and neighbor-graph smoothing is an option.
when_to_use_negative:
- The input is already aggregated bulk expression — the nearest-neighbor smoothing
  and per-cell ranked scoring target sparse single-cell data (mini-pseudobulk profiles
  per cell).
- You need an unranked score such as mean_z, robust_std, average_score, median_score,
  or summed_up, which the README recommends for raw counts rather than rank-ordered
  data.
- There is no meaningful grouping structure to build the neighbor graph within (groupby),
  e.g. too few cells per group to support recompute_neighbors.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: gssnng
  role: 'Provides score_cells.with_gene_sets with score_method=''ssGSEA'' (method_params={''omega'':
    0.75}) and smoothing.smooth_adata for nearest-neighbor graph smoothing of the
    count matrix'
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the h5ad AnnData object and supplies the neighborhood graph / grouping
    used for smoothing
  repo: https://github.com/IlyaLab/gssnng
- name: AnnData
  role: Container format holding counts, obs groupings, and gssnng score outputs in
    obsm
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:9d0354520522766f9275a1a048d36b82eb4127555468a83d86aef09e8175ef13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# singlesample-gsea-scoring

## Summary

Scores single cells with ssGSEA, a single-sample GSEA method operating on per-cell ranked expression, inside gssnng's score_cells.with_gene_sets workflow (method_params={'omega': 0.75}). Applied to nearest-neighbor-graph smoothed Scanpy AnnData, mini-pseudobulk profiles raise spinal-injury endothelial AUC discrimination from 0.69 (unsmoothed) to 0.8 (smoothed).

## When to use

Use when per-cell scRNA-seq counts (h5ad AnnData) must be scored against a two-part UP/DN .gmt signature with a ranked, single-sample method, especially when sparsity blurs signal and neighbor-graph smoothing is an option.

## When NOT to use

- The input is already aggregated bulk expression — the nearest-neighbor smoothing and per-cell ranked scoring target sparse single-cell data (mini-pseudobulk profiles per cell).
- You need an unranked score such as mean_z, robust_std, average_score, median_score, or summed_up, which the README recommends for raw counts rather than rank-ordered data.
- There is no meaningful grouping structure to build the neighbor graph within (groupby), e.g. too few cells per group to support recompute_neighbors.

## Inputs

- Scanpy AnnData object (h5ad) with raw single-cell gene expression counts
- Gene set file (.gmt) with UP/DN gene sets, e.g. 7-UP / 5-DN spinal injury signature using _UP/_DN nomenclature
- obs column (groupby) defining cell groups for neighbor-graph construction, e.g. endothelial cell grouping
- Score method parameters: score_method='ssGSEA', method_params={'omega': 0.75}, ranked=True

## Outputs

- Per-cell ssGSEA gene set scores stored as columns in adata.obsm['gssnng_estimate']
- AUC values quantifying injured vs control endothelial separation, written with scorer/condition labels to auc_injury_signature.csv (ssGSEA unsmoothed AUC 0.69, smoothed AUC 0.8)

## How to apply

Read the AnnData (h5ad) with scanpy and rank-order gene expression counts per cell, since ssGSEA is a ranked-data method. Supply the gene set as a .gmt file using MSigDB-style _UP/_DN nomenclature (e.g. the 7-UP / 5-DN spinal injury signature) and call score_cells.with_gene_sets with score_method='ssGSEA' and method_params={'omega': 0.75}. To combat sparsity, first build mini-pseudobulk profiles with smoothing.smooth_adata using groupby on the cell grouping, smooth_mode='connectivity' (weights close neighbors more), and a fixed recompute_neighbors N; re-rank the smoothed counts before rescoring. Decision point: run the scorer on both unsmoothed and smoothed matrices and compare AUCs of injured vs control separation — in the paper ssGSEA improved from 0.69 to 0.8. Each gene set appears as a column in adata.obsm['gssnng_estimate'].

## Related tools

- **gssnng** (Provides score_cells.with_gene_sets with score_method='ssGSEA' (method_params={'omega': 0.75}) and smoothing.smooth_adata for nearest-neighbor graph smoothing of the count matrix) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Reads the h5ad AnnData object and supplies the neighborhood graph / grouping used for smoothing) — https://github.com/IlyaLab/gssnng
- **AnnData** (Container format holding counts, obs groupings, and gssnng score outputs in obsm) — https://github.com/IlyaLab/gssnng

## Examples

```
score_cells.with_gene_sets(adata, gene_set_file='injury_signature.gmt', score_method='ssGSEA', method_params={'omega': 0.75}, ranked=True, groupby='cell_type', smooth_mode='connectivity', recompute_neighbors=0)
```

## Evaluation signals

- score_method='ssGSEA' is run with ranked=True, since the README states ssGSEA is 'Single sample GSEA based on ranked data'
- method_params includes the documented ssGSEA parameter {'omega': 0.75}
- Smoothing is applied with smooth_mode='connectivity' and an explicit recompute_neighbors setting within the groupby grouping before re-scoring and re-ranking
- Discrimination check: AUC of injured vs control endothelial cells should improve on smoothed vs unsmoothed data (paper values 0.69 → 0.8)
- auc_injury_signature.csv contains all four labeled AUC values (RBO/ssGSEA × unsmoothed/smoothed)

## Limitations

- Sparsity and technical noise of single-cell expression measures make methods designed for bulk data difficult to apply; without the smoothing transformation scores may be unreliable.
- ssGSEA requires rank-ordered expression; feeding unranked counts violates the method's assumption.
- Smoothing operates within user-defined groupby groups, so results depend on that grouping; groups can be processed in parallel via cores.

## Evidence

- [readme] ssGSEA:         Single sample GSEA based on ranked data.: "ssGSEA:         Single sample GSEA based on ranked data."
- [methods] By operating on the nearest neighbor graph, we create a mini-pseudobulk expression profile of neighbors for each cell, thus smoothing the count data.: "By operating on the nearest neighbor graph, we create a mini-pseudobulk expression profile of neighbors for each cell, thus smoothing the count data."
- [readme] ssGSEA: {'omega': 0.75}: "ssGSEA: {'omega': 0.75}"
- [intro] This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq.: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring"
- [readme] adjacency weights all neighbors equally, connectivity weights close neighbors more: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
