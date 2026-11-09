---
name: geneset-overlap-scoring
description: Use when scoring single-cell RNA-seq cells for membership in a gene signature
  (e.g. LM22 cell-type sets from a .gmt file) by counting genes expressed above a
  threshold, especially on sparse pbmc3k-style count matrices where smoothing should
  boost detection.
when_to_use_negative:
- Data is already ranked (ranked expression values) — geneset_overlap counts thresholded
  expression, while rank-based methods like singscore or ssGSEA require ranked data
- The goal is magnitude-aware pathway activity rather than binary detection of signature
  genes above a threshold
- Analysis is on bulk RNA-seq samples with no per-cell nearest-neighbor graph, where
  mini-pseudobulk smoothing is not defined
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Provides score_cells.with_gene_sets with score_method='geneset_overlap', smooth_mode
    ('off'/'adjacency'/'connectivity'), and recompute_neighbors parameters for per-cell
    gene set overlap scoring
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads the h5ad AnnData (e.g. pbmc3k_processed) and supplies the obs groupby
    column (louvain) used to build neighborhood graphs
  repo: https://github.com/IlyaLab/gssnng
- name: AnnData
  role: Container for the count matrix and obsm['gssnng_estimate'] score output
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:f8e8ccda75cc70e624ba6a76f665c13a40cce05476443924a3665b50c3921575
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# geneset-overlap-scoring

## Summary

Computes per-cell gene set overlap scores with gssnng's 'geneset_overlap' method, counting (or fractioning) the number of signature genes expressed past a threshold, optionally on nearest-neighbor-smoothed AnnData to recover signal lost to sparsity.

## When to use

Use when scoring single-cell RNA-seq cells for membership in a gene signature (e.g. LM22 cell-type sets from a .gmt file) by counting genes expressed above a threshold, especially on sparse pbmc3k-style count matrices where smoothing should boost detection.

## When NOT to use

- Data is already ranked (ranked expression values) — geneset_overlap counts thresholded expression, while rank-based methods like singscore or ssGSEA require ranked data
- The goal is magnitude-aware pathway activity rather than binary detection of signature genes above a threshold
- Analysis is on bulk RNA-seq samples with no per-cell nearest-neighbor graph, where mini-pseudobulk smoothing is not defined

## Inputs

- Scanpy AnnData object (h5ad) with expression matrix in adata.X, e.g. sc.datasets.pbmc3k_processed()
- GMT gene set file with MSigDB-style gene sets and UP/DN direction suffixes, e.g. cibersort_lm22.gmt
- Grouping column in adata.obs (e.g. 'louvain') defining neighborhoods
- Expression threshold passed via method_params

## Outputs

- Per-cell gene set overlap score table (one row per cell, one column per gene set), e.g. geneset_overlap_scores.csv
- Column adata.obsm['gssnng_estimate'] per gene set when scored through the gssnng/decoupleR-style API
- Comparison table of overlap fractions across smoothing conditions, e.g. b_cell_naive_overlap_table.csv

## How to apply

Load the AnnData object (e.g. via sc.datasets.pbmc3k_processed()) and parse gene sets from a .gmt file following MSigDB nomenclature, including _UP/_DN direction suffixes. Call gssnng's score_cells.with_gene_sets with score_method='geneset_overlap', groupby set to a clustering column such as 'louvain', ranked=False, and method_params carrying the expression threshold. Build a baseline run with smooth_mode='off', then repeat with smooth_mode='connectivity' and recompute_neighbors set to a specific K (e.g. 8 or 32) so the neighbor graph is rebuilt within each group; 'connectivity' weights close neighbors more than the 'adjacency' mode's uniform weighting. The geneset_overlap metric counts, for each cell and gene set, how many signature genes exceed the threshold, optionally as a fraction in [0,1]. Tabulate per-cell scores (one column per gene set) to .csv and compare non-smoothed versus smoothed fractions to quantify the detection gain.

## Related tools

- **gssnng** (Provides score_cells.with_gene_sets with score_method='geneset_overlap', smooth_mode ('off'/'adjacency'/'connectivity'), and recompute_neighbors parameters for per-cell gene set overlap scoring) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Loads the h5ad AnnData (e.g. pbmc3k_processed) and supplies the obs groupby column (louvain) used to build neighborhood graphs) — https://github.com/IlyaLab/gssnng
- **AnnData** (Container for the count matrix and obsm['gssnng_estimate'] score output) — https://github.com/IlyaLab/gssnng

## Examples

```
q = sc.datasets.pbmc3k_processed(); gssnng.score_cells.with_gene_sets(adata=q, gene_set_file='cibersort_lm22.gmt', score_method='geneset_overlap', groupby='louvain', smooth_mode='connectivity', recompute_neighbors=32, ranked=False)
```

## Evaluation signals

- Output CSV has one row per cell and one score column per gene set parsed from the .gmt file
- Fraction-mode overlap values fall within [0, 1] for every cell
- Smoothed runs (smooth_mode='connectivity', recompute_neighbors=8 and 32) show increased overlap fractions versus the smooth_mode='off' baseline — e.g. approximately 2-fold increase in B.cell.naive overlap for louvain-annotated B cells in pbmc3k at K=32
- Scores land in adata.obsm['gssnng_estimate'] with each gene set as a column
- Neighbors are recomputed within each groupby group when recompute_neighbors>0 (no cross-group edges)

## Limitations

- Sparsity and technical noise of single-cell expression measures create difficulties for gene set scoring designed for bulk data, which is why smoothing is needed — unsmoothed thresholds undercount signature genes
- geneset_overlap is a thresholded count/fraction metric; it does not use expression magnitudes like rank-based methods (singscore, ssGSEA) do
- score_method 'geneset_overlap' with ranked=False requires choosing an expression threshold via method_params; the number of detected genes depends on this threshold
- Smoothing mode and neighbor count (recompute_neighbors K) are user decisions; wrong K can under- or over-smooth neighborhoods within groupby groups

## Evidence

- [readme] geneset_overlap definition: "geneset_overlap: For each geneset, number (or fraction) of genes expressed past a given threshold."
- [abstract] smoothing rationale: [evidence span withheld — anchor status: not_found]
- [intro] smoothing mechanics: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [readme] connectivity vs adjacency: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [results] overlap doubling finding: [evidence span withheld — anchor status: not_found]
