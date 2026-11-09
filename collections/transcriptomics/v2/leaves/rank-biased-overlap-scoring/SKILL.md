---
name: rank-biased-overlap-scoring
description: Use when scoring single-cell RNA-seq cells (Scanpy AnnData h5ad) for
  membership in a gene set using a rank-based statistic, e.g. testing whether an _UP/_DN
  signature separates two cell states such as injured vs control endothelial cells.
when_to_use_negative:
- Data is bulk RNA-seq or already aggregated pseudobulk, where the nearest-neighbor
  smoothing of gssnng is unnecessary
- The chosen scoring statistic is one gssnng recommends for unranked data (e.g. mean_z,
  robust_std), which conflict with ranked=True
- The goal is pathway activity inference from a model/resource (Decoupler/Omnipath
  style) rather than scoring a user-supplied .gmt signature
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Provides the score_cells.with_gene_sets API with score_method='rank_biased_overlap'
    and the smoothing.smooth_adata function that builds mini-pseudobulk profiles on
    the nearest neighbor graph
  repo: github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the h5ad AnnData object and handles per-cell data structures that gssnng
    operates on
- name: AnnData
  role: The object/file format (h5ad) holding cells, counts, and obsm score outputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:bebb353e6e2d23013879389ac49129abc53a54010c9d531bb1a350a6baf0a126
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rank-biased-overlap-scoring

## Summary

Ranks per-cell gene expression and applies the rank_biased_overlap (RBO) scorer from gssnng — a weighted average of agreement between sorted ranks and a gene set — to estimate gene set activity per single cell, optionally on nearest-neighbor-smoothed counts.

## When to use

Use when scoring single-cell RNA-seq cells (Scanpy AnnData h5ad) for membership in a gene set using a rank-based statistic, e.g. testing whether an _UP/_DN signature separates two cell states such as injured vs control endothelial cells.

## When NOT to use

- Data is bulk RNA-seq or already aggregated pseudobulk, where the nearest-neighbor smoothing of gssnng is unnecessary
- The chosen scoring statistic is one gssnng recommends for unranked data (e.g. mean_z, robust_std), which conflict with ranked=True
- The goal is pathway activity inference from a model/resource (Decoupler/Omnipath style) rather than scoring a user-supplied .gmt signature

## Inputs

- Scanpy AnnData object (h5ad file) of single-cell counts
- .gmt gene set file (MSigDB nomenclature, one gene set per line; _UP/_DN nomenclature for directional signatures)
- groupby column label (or list/dict) in adata.obs defining neighborhoods (e.g. endothelial cell grouping)
- Per-cell ranked expression matrix (ranked=True)

## Outputs

- Per-cell RBO gene set scores in adata.obsm['gssnng_estimate'] (one column per gene set)
- Condition-separation metrics such as AUC values comparing unsmoothed vs smoothed scoring (e.g. auc_injury_signature.csv with labeled scorer/smoothing conditions)

## How to apply

Load the AnnData (h5ad) with scanpy and rank-order gene expression counts per cell, since RBO requires ranked data. Call gssnng's score_cells.with_gene_sets with score_method='rank_biased_overlap', ranked=True, and optionally method_params={'rbo_depth': n}, where n is the depth descended down the ranks at each step of overlap measurement with the gene set. For improved robustness to single-cell sparsity, first smooth the count matrix with gssnng smoothing.smooth_adata (groupby on a cell grouping, smooth_mode='connectivity', recompute_neighbors>0) to build mini-pseudobulk profiles per cell, re-rank, and re-score. Scores land per gene set in adata.obsm['gssnng_estimate'] and can then be compared across conditions (e.g. AUC of injured vs control). Rationale: sparsity and technical noise of single-cell measures hurt bulk-derived gene set scorers, and nearest-neighbor smoothing remedied the RBO AUC from 0.62 (unsmoothed) to 0.71 (smoothed) on the Squair spinal-injury endothelial data.

## Related tools

- **gssnng** (Provides the score_cells.with_gene_sets API with score_method='rank_biased_overlap' and the smoothing.smooth_adata function that builds mini-pseudobulk profiles on the nearest neighbor graph) — github.com/IlyaLab/gssnng
- **Scanpy** (Reads the h5ad AnnData object and handles per-cell data structures that gssnng operates on)
- **AnnData** (The object/file format (h5ad) holding cells, counts, and obsm score outputs)

## Examples

```
from gssnng import score_cells; score_cells.with_gene_sets(adata, gene_set_file='injury_signature.gmt', groupby='cell_type', score_method='rank_biased_overlap', method_params={'rbo_depth': 100}, ranked=True, recompute_neighbors=0, cores=1)
```

## Evaluation signals

- Scores appear as gene set columns in adata.obsm['gssnng_estimate'] after score_cells.with_gene_sets
- ranked=True is set and RBO runs without complaining about unranked input, since RBO is defined on sorted ranks
- method_params['rbo_depth'] is a positive integer n controlling rank descent depth
- On the Squair spinal-injury endothelial task, smoothing should raise the RBO AUC from ~0.62 (unsmoothed) to ~0.71 (smoothed), i.e. smoothed ≥ unsmoothed
- Output CSV (auc_injury_signature.csv) contains all four labeled AUC values (unsmoothed vs smoothed x RBO vs ssGSEA)

## Limitations

- RBO requires ranked expression data; it cannot be run on raw unranked counts
- Unsmoothed single-cell data is sparse and noisy, which degrades RBO scores (AUC 0.62 on the injury signature)
- The rbo_depth parameter n must be chosen by the user; it determines how far down the ranks overlap with the gene set is measured

## Evidence

- [methods] rank_biased_overlap:  RBO, Weighted average of agreement between sorted ranks and gene set.: "rank_biased_overlap:  RBO, Weighted average of agreement between sorted ranks and gene set."
- [readme] rbo_depth parameter definition: "rank_biased_overlap:  {'rbo_depth', n}  (n: int)
Here, n is the depth that is decended down the ranks, where at each step, the overlap with the gene set is measured and added to the score."
- [readme] mini-pseudobulk rationale: "The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing. This essentially creates 
mini-pseudobulk expression profiles for each cell, which can be scored by using"
- [abstract] sparsity motivation: [evidence span withheld — anchor status: not_found]
