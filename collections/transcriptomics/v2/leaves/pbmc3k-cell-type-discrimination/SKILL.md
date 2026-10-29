---
name: pbmc3k-cell-type-discrimination
description: Use when evaluating whether gene set scores computed on sparsified/downsampled
  single-cell data remain discriminative between cell types, or when comparing smoothed
  vs non-smoothed scoring robustness.
when_to_use_negative:
- The data are bulk RNA-seq or already pseudobulked — the smoothing rationale addresses
  single-cell sparsity and technical noise, which bulk data do not exhibit.
- The goal is cell-type clustering or annotation itself rather than robustness testing
  of gene set scores; the workflow assumes pre-assigned Louvain groups and known B-cell
  labels.
- The AnnData lacks a neighborhood graph or grouping column (groupby='louvain') and
  cannot support nearest neighbor graph construction within groups.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Performs nearest neighbor graph smoothing and gene set scoring (score_method='singscore',
    smooth_mode='connectivity' or 'off', recompute_neighbors=32)
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads the processed pbmc3k AnnData and provides the louvain grouping used
    for graph construction
- name: AnnData
  role: Data container (h5ad-backed) holding the cells, counts, and obsm gene set
    scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:a984b0454a38b35719f258e6b541b115fde76c3b5b98994cf4cfcee8e248c1b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pbmc3k-cell-type-discrimination

## Summary

Benchmarking workflow that quantifies how nearest neighbor graph smoothing (gssnng) makes single-cell gene set scores robust to count-matrix downsampling, using AUC of the 'B cells naive' signature to discriminate B cells from other PBMC types in the processed pbmc3k dataset.

## When to use

Use when evaluating whether gene set scores computed on sparsified/downsampled single-cell data remain discriminative between cell types, or when comparing smoothed vs non-smoothed scoring robustness.

## When NOT to use

- The data are bulk RNA-seq or already pseudobulked — the smoothing rationale addresses single-cell sparsity and technical noise, which bulk data do not exhibit.
- The goal is cell-type clustering or annotation itself rather than robustness testing of gene set scores; the workflow assumes pre-assigned Louvain groups and known B-cell labels.
- The AnnData lacks a neighborhood graph or grouping column (groupby='louvain') and cannot support nearest neighbor graph construction within groups.

## Inputs

- Processed pbmc3k dataset as a Scanpy AnnData object (sc.datasets.pbmc3k_processed())
- Gene set file cibersort_lm22.gmt containing the 'B cells naive' signature
- Series of downsampling fractions (95%, 90%, ..., 5%) applied via the Zhang sequencing-depth reduction recipe

## Outputs

- Per-cell 'B cells naive' signature scores per downsampling fraction and condition (smoothed vs non-smoothed), e.g. in adata.obsm['gssnng_estimate']
- auc_vs_downsampling.csv with AUC values versus downsampling fraction for both conditions

## How to apply

Load the processed pbmc3k dataset via sc.datasets.pbmc3k_processed() into a Scanpy AnnData object, then downsample the count matrix to a series of total-count fractions (95% down to 5%) using the Zhang recipe for sequencing-depth reduction. For each fraction, score cells with gssnng using the 'B cells naive' signature from cibersort_lm22.gmt with score_method='singscore', ranked=True, and groupby='louvain'. Run two conditions: the smoothed condition with smooth_mode='connectivity' and recompute_neighbors=32, and the control with smooth_mode='off'. For each fraction and condition, compute the AUC of the per-cell 'B cells naive' score for discriminating naive B cells from other PBMC cell types, and assemble the AUC-vs-fraction curves into auc_vs_downsampling.csv. Rationale: smoothing on the nearest neighbor graph creates mini-pseudobulk expression profiles per cell, mitigating sparsity and technical noise so scores stay stable as counts are removed.

## Related tools

- **gssnng** (Performs nearest neighbor graph smoothing and gene set scoring (score_method='singscore', smooth_mode='connectivity' or 'off', recompute_neighbors=32)) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Loads the processed pbmc3k AnnData and provides the louvain grouping used for graph construction)
- **AnnData** (Data container (h5ad-backed) holding the cells, counts, and obsm gene set scores)

## Examples

```
import scanpy as sc; q = sc.datasets.pbmc3k_processed(); # then gssnng scores_cells.with_gene_sets(adata=q, gene_set_file='cibersort_lm22.gmt', groupby='louvain', smooth_mode='connectivity', recompute_neighbors=32, score_method='singscore', ranked=True)
```

## Evaluation signals

- With K=32 connectivity smoothing, the AUC for B-cell vs non-B-cell discrimination stays above 0.95 until fewer than 10% of total counts remain.
- The non-smoothed (smooth_mode='off') control AUC degrades immediately as the retained count fraction decreases, providing the expected contrast.
- Each downsampled fraction and condition produces a per-cell score column for 'B cells naive' from cibersort_lm22.gmt.
- auc_vs_downsampling.csv contains paired AUC values across the full 95%-to-5% fraction grid for both conditions.
- singscore runs with ranked=True, so gene expression counts must be rank-ordered before scoring.

## Limitations

- The benchmark is demonstrated on the pbmc3k dataset with the 'B cells naive' signature; the 0.95 AUC threshold at <10% retained counts is specific to this configuration.
- Smoothing assumes the nearest neighbor graph groups are biologically coherent; graphs are constructed within user-defined groups (groupby parameter), so poor groupings could dilute cell-type signal.
- The gssnng article's discussion section is missing from the indexed source, so broader caveats on smoothing artifacts are not documented here.

## Evidence

- [results] Smoothed gene set scores remain robust to severe downsampling: [evidence span withheld — anchor status: not_found]
- [abstract] The sparsity and technical noise of single-cell expression measures create difficulties for these methods: [evidence span withheld — anchor status: not_found]
- [readme] singscore: Normalised mean (median centered) ranks (requires ranked data): "singscore:      Normalised mean (median centered) ranks (requires ranked data)"
- [readme] smooth_mode parameters and neighbor weighting: "Dictates how to use the neighborhood graph. `adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [intro] mini-pseudobulk rationale: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
