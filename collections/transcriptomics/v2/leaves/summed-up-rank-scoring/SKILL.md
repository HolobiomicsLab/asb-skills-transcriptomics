---
name: summed-up-rank-scoring
description: Use when per-cell gene set scores are needed from smoothed scRNA-seq
  AnnData and a simple rank/count-sum statistic should serve as the score, or when
  a one-dimensional score threshold (e.g., the median) must classify cells by condition
  such as LPS treatment.
when_to_use_negative:
- The input is already a gene set score table or bulk expression matrix — summed_up
  operates on per-cell single-cell count/rank matrices via the neighborhood graph
- A differential or ranked-statistic gene set score is required (e.g., ssGSEA, singscore,
  rank_biased_overlap) rather than a simple sum of counts/ranks
- Cells lack a meaningful groupby grouping for neighbor recomputation, or the neighborhood
  graph should not be built within biological groups
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Provides the summed_up scoring function and nearest-neighbor graph smoothing
    (smooth_mode='connectivity', recompute_neighbors) on Scanpy AnnData objects
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the .h5ad AnnData and supplies the neighborhood graph / obsm storage
    for scores
- name: AnnData
  role: Container format holding counts, obs metadata (LPS labels), and per-cell score
    results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:e5b09b1ca56fd52e0929c8736724519a663379975666cd7714a025b4a0a2f0ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# summed-up-rank-scoring

## Summary

Applies the gssnng 'summed_up' scoring method — summing ranks or counts of genes in a gene set per cell on nearest-neighbor-smoothed single-cell data — and can use the per-cell summed_up scores with a median cut point as a simple binary classifier (e.g., predicting LPS treatment status in the Hagai phagocyte dataset).

## When to use

Use when per-cell gene set scores are needed from smoothed scRNA-seq AnnData and a simple rank/count-sum statistic should serve as the score, or when a one-dimensional score threshold (e.g., the median) must classify cells by condition such as LPS treatment.

## When NOT to use

- The input is already a gene set score table or bulk expression matrix — summed_up operates on per-cell single-cell count/rank matrices via the neighborhood graph
- A differential or ranked-statistic gene set score is required (e.g., ssGSEA, singscore, rank_biased_overlap) rather than a simple sum of counts/ranks
- Cells lack a meaningful groupby grouping for neighbor recomputation, or the neighborhood graph should not be built within biological groups

## Inputs

- Scanpy AnnData object (.h5ad) with raw single-cell counts
- Gene set file in .gmt format (one gene set per line, MSigDB nomenclature)
- obs metadata column with condition labels (e.g., LPS treatment status)
- Per-cell summed_up gene set score tables (unsmoothed and smoothed, smooth_mode='connectivity')

## Outputs

- Per-cell summed_up gene set scores (columns in adata.obsm['gssnng_estimate'] or per-cell score tables)
- Per-condition classification accuracies (e.g., lps_accuracy_report.csv with ~0.88 unsmoothed, ~0.97 smoothed)

## How to apply

Score cells with gssnng's scores_cells.with_gene_sets using score_method='summed_up', which sums the ranks or counts for genes in each gene set; the summed_up score is recommended on unranked (count) data after nearest-neighbor graph smoothing. Run smoothing with smooth_mode='connectivity' (weights close neighbors more than 'adjacency') and recompute neighbors within each groupby group (recompute_neighbors > 0) so mini-pseudobulk profiles are built within biological groups. Scores land per cell (e.g., in adata.obsm['gssnng_estimate'] or exported per-cell tables). For classification, compute the median of the summed_up scores across cells and use it as the binary cut point, assigning predicted condition labels per cell and comparing to true labels (e.g., LPS treatment) to compute accuracy. Rationale: smoothing creates mini-pseudobulk expression profiles per cell that reduce single-cell sparsity, improving threshold-based classification; in the Hagai phagocyte benchmark, median-cut classification of LPS status improved from 0.88 (unsmoothed) to 0.97 (smoothed).

## Related tools

- **gssnng** (Provides the summed_up scoring function and nearest-neighbor graph smoothing (smooth_mode='connectivity', recompute_neighbors) on Scanpy AnnData objects) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Reads the .h5ad AnnData and supplies the neighborhood graph / obsm storage for scores)
- **AnnData** (Container format holding counts, obs metadata (LPS labels), and per-cell score results)

## Examples

```
scores_cells.with_gene_sets(adata=q, gene_set_file='genes.gmt', groupby='sample', smooth_mode='connectivity', recompute_neighbors=30, score_method='summed_up', ranked=False, cores=8)
```

## Evaluation signals

- Smoothed-condition accuracy must exceed unsmoothed accuracy for LPS status classification; expected values are ~0.97 smoothed vs ~0.88 unsmoothed; flag the run as failing the paper's qualitative acceptance check otherwise
- Each gene set appears as a column in adata.obsm['gssnng_estimate'] after scoring
- summed_up scores should be computed on unranked count data (the method sums counts or ranks; unranked is the natural mode for a sum)
- Verify neighbors were recomputed within each groupby group (recompute_neighbors > 0) and smooth_mode is 'connectivity' when reproducing the 0.97 result
- Median cut point must be recomputed per condition (unsmoothed vs smoothed tables separately) before assigning predicted labels

## Limitations

- summed_up is a plain sum of ranks or counts and lacks the normalization of methods like singscore or robust_std; gene sets with different sizes or overlap fractions (geneset_overlap) are not directly comparable
- Accuracy depends on smoothing parameters — results (0.88 vs 0.97) assume smooth_mode='connectivity' with neighbors recomputed within each groupby group; other settings are not validated here
- The median cut point is an unsupervised threshold; it presumes roughly balanced class sizes and does not generalize to imbalanced LPS labeling
- No discussion-section content was available in the source document, so failure modes beyond those stated are undocumented

## Evidence

- [readme] summed_up: "**summed_up**:      Sum up the ranks or counts."
- [abstract] smoothing rationale: "we use the nearest neighbor graph of cells for matrix smoothing to produce high quality gene set scores on a per-cell, per-group, level"
- [intro] pseudobulk rationale: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [readme] smooth_mode parameter: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [methods] score output location: "3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [full_text] benchmark finding: [evidence span withheld — anchor status: not_found]
