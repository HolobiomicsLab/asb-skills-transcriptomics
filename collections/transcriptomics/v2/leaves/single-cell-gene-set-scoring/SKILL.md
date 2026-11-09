---
name: single-cell-gene-set-scoring
description: Use when per-cell gene set scores on Scanpy AnnData (h5ad) scRNA-seq
  data are needed and count sparsity/technical noise is degrading detection — e.g.
  comparing smoothed vs unsmoothed scores or testing whether smoothing (K=4/8/32)
  improves separation of cell states or treatment labels.
when_to_use_negative:
- Input is bulk RNA-seq or already-aggregated pseudobulk profiles — smoothing over
  a cell-level nearest neighbor graph is unnecessary and inappropriate
- Ranked scoring methods (singscore, ssGSEA, rank_biased_overlap) are requested but
  the AnnData holds only sparse raw counts unsuitable for rank ordering without prior
  preparation (ranked=False methods like mean_z/robust_std are recommended for unranked
  data)
- No meaningful groupby structure exists and recompute_neighbors=0 would leave the
  graph computed across all cells, mixing biologically unrelated neighborhoods
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Performs nearest-neighbor-graph smoothing and single-sample gene set scoring
    via score_cells.with_gene_sets
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads h5ad AnnData objects and supplies datasets/datasets such as sc.datasets.pbmc3k_processed()
- name: AnnData
  role: Data container whose X, obs, and obsm['gssnng_estimate'] slots carry expression,
    group labels, and scores
- name: decoupleR
  role: Decoupler/OmniPath style API for obtaining gene set models and extracting
    activities with dc.get_acts
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:dc120159420be7f5a8900e66f694c1b2bc30f8ac9fafa24ae739818c7ddd268d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - decoupleR
  - gssnng
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-gene-set-scoring

## Summary

Score single-cell RNA-seq data for gene set activity using gssnng, which smooths the expression matrix over the nearest neighbor graph of cells (creating mini-pseudobulk profiles) before applying bulk-style single-sample scoring methods such as geneset_overlap or summed_up.

## When to use

Use when per-cell gene set scores on Scanpy AnnData (h5ad) scRNA-seq data are needed and count sparsity/technical noise is degrading detection — e.g. comparing smoothed vs unsmoothed scores or testing whether smoothing (K=4/8/32) improves separation of cell states or treatment labels.

## When NOT to use

- Input is bulk RNA-seq or already-aggregated pseudobulk profiles — smoothing over a cell-level nearest neighbor graph is unnecessary and inappropriate
- Ranked scoring methods (singscore, ssGSEA, rank_biased_overlap) are requested but the AnnData holds only sparse raw counts unsuitable for rank ordering without prior preparation (ranked=False methods like mean_z/robust_std are recommended for unranked data)
- No meaningful groupby structure exists and recompute_neighbors=0 would leave the graph computed across all cells, mixing biologically unrelated neighborhoods

## Inputs

- Scanpy AnnData object (h5ad file) with expression in adata.X and a groupby column in adata.obs (e.g. louvain clusters or treatment labels)
- Gene set file in GMT format (e.g. cibersort_lm22.gmt, MSigDB-style sets, or Okumura Inflammatory Response LPS set)
- Optional decoupler/OmniPath model obtained via the decoupler API

## Outputs

- Per-cell gene set score matrix stored in adata.obsm['gssnng_estimate'] (one column per gene set)
- Activities as an AnnData object via dc.get_acts(adata, obsm_key='gssnng_estimate')
- Per-condition summary tables (e.g. b_cell_naive_overlap_table.csv, lps_smoothing_auc_results.csv, lps_accuracy_report.csv) comparing unsmoothed vs K=4/8/32 smoothed scores

## How to apply

Read the AnnData object with scanpy (h5ad; e.g. sc.datasets.pbmc3k_processed() for pbmc3k) and load gene sets from a .gmt file (e.g. cibersort_lm22.gmt or the Okumura Inflammatory Response LPS set). Call gssnng score_cells.with_gene_sets, choosing score_method from the nine options (geneset_overlap, singscore, ssGSEA, rank_biased_overlap, robust_std, mean_z, average_score, median_score, summed_up), setting groupby to an obs column such as 'louvain', ranked=True/False depending on whether the method requires ranks, and smooth_mode to 'off' (baseline), 'adjacency' (equal neighbor weights), or 'connectivity' (closer neighbors weighted more). To reproduce the paper's smoothing benefit, run the baseline with recompute_neighbors=0, then rebuild the neighbor graph within each group at K=8 and K=32 (optionally K=4) using smooth_mode='connectivity', and compare per-cell scores across conditions. Scores land as columns in adata.obsm['gssnng_estimate'] and can be extracted as an AnnData of activities via dc.get_acts. Rationale: smoothing over the nearest neighbor graph shares expression information within transcriptomic neighborhoods, remedying sparsity-driven false negatives in gene set detection.

## Related tools

- **gssnng** (Performs nearest-neighbor-graph smoothing and single-sample gene set scoring via score_cells.with_gene_sets) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Loads h5ad AnnData objects and supplies datasets/datasets such as sc.datasets.pbmc3k_processed())
- **AnnData** (Data container whose X, obs, and obsm['gssnng_estimate'] slots carry expression, group labels, and scores)
- **decoupleR** (Decoupler/OmniPath style API for obtaining gene set models and extracting activities with dc.get_acts)

## Examples

```
from gssnng import score_cells; score_cells.with_gene_sets(adata=q, gene_set_file='cibersort_lm22.gmt', score_method='geneset_overlap', groupby='louvain', smooth_mode='connectivity', recompute_neighbors=32, ranked=False)
```

## Evaluation signals

- pbmc3k validation: geneset_overlap fraction for the B.cell.naive (LM22) signature in annotated B cells should approximately double under smooth_mode='connectivity', recompute_neighbors=32 vs smooth_mode='off'
- LPS AUC check: ROC AUC of summed_up scores separating LPS-treated vs untreated phagocytes should be ~0.95 unsmoothed and reach ~1.0 when smoothed (smoothed >= unsmoothed)
- Median-cut check: classification accuracy of LPS status using the median summed_up score as cut point should be ~0.88 unsmoothed and ~0.97 smoothed; flag the run as failing if smoothed accuracy does not exceed unsmoothed
- Schema check: each scored gene set appears as a column in adata.obsm['gssnng_estimate']
- Graph reconstruction check: with recompute_neighbors > 0, the nearest neighbor graph is rebuilt within each groupby group at the specified N (e.g. 8, 32) rather than reused

## Limitations

- No discussion section content was found in the source document, so caveats beyond the parameter documentation are not extracted
- smooth_mode choice changes semantics: 'adjacency' weights all neighbors equally while 'connectivity' weights close neighbors more — results are not interchangeable between modes
- Some methods require ranked data (singscore needs ranked data; rank_biased_overlap operates on sorted ranks), and others recommend unranked data (robust_std, mean_z)
- Nearest neighbor graphs are constructed per user-defined groupby group; poorly chosen groups constrain the neighborhoods over which smoothing occurs

## Evidence

- [abstract] smoothing over NNG for sparsity: [evidence span withheld — anchor status: not_found]
- [intro] method basis: "The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing."
- [readme] smooth_mode semantics: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [methods] output location: "Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] mini-pseudobulk rationale: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
