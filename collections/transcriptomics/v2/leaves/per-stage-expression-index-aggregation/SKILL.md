---
name: per-stage-expression-index-aggregation
description: Use when a per-cell gene age-weighted or gene-bin-weighted expression
  index has already been computed on a Scanpy AnnData object and the goal is to summarize
  it per developmental stage or cell-type group.
when_to_use_negative:
- Only per-cell index values are needed and no stage/cell-type grouping exists in
  the AnnData metadata — aggregation over stages would be meaningless
- Gene IDs in the mapping table have not yet been matched to the scRNA data set identifiers;
  unmatched genes would silently drop out of the stage-level index
- The goal is to compute the index itself from raw counts rather than to summarize
  an already-computed index per stage
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: oggmap
  role: provides the TEI calculation on Scanpy objects, get_bins for alternative gene-category
    weights, and visualization of category expression profiles per stage
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: holds the AnnData scRNA object on which the weighted index is calculated and
    whose cell metadata supplies the developmental-stage grouping
- name: Python
  role: implementation language; oggmap v0.0.2 was developed using Python 3.10
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:cbf5c6a19f3f92935c8322abf93e6c87ced5ed636b469ae616e2faa590afd06f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# per-stage-expression-index-aggregation

## Summary

Aggregates a per-cell weighted expression index (the transcriptome evolutionary index, TEI, or its binned analogue computed from get_bins gene groups) over cells grouped by developmental stage, producing a summary table (e.g. binned_index_per_stage.tsv) that exposes stage-level shifts in weighted expression.

## When to use

Use when a per-cell gene age-weighted or gene-bin-weighted expression index has already been computed on a Scanpy AnnData object and the goal is to summarize it per developmental stage or cell-type group.

## When NOT to use

- Only per-cell index values are needed and no stage/cell-type grouping exists in the AnnData metadata — aggregation over stages would be meaningless
- Gene IDs in the mapping table have not yet been matched to the scRNA data set identifiers; unmatched genes would silently drop out of the stage-level index
- The goal is to compute the index itself from raw counts rather than to summarize an already-computed index per stage

## Inputs

- Scanpy AnnData scRNA count object with a developmental-stage cell annotation (e.g. nematode scRNA data)
- Per-cell weighted expression index (TEI or binned index) computed from an orthomap or get_bins binning_map.tsv
- Gene-to-group mapping table (orthomap or binning_map.tsv) with IDs matched to the scRNA gene/transcript identifiers

## Outputs

- Stage-level summary table of the aggregated index (e.g. binned_index_per_stage.tsv)
- Visualization of the gene category expression profile across developmental stages (e.g. zebrafish TEI or nematode pi plot)

## How to apply

First compute the per-cell index on the Scanpy object: weights come from the gene age classes in the orthomap, or from arbitrary gene-metric bins produced by oggmap's get_bins (e.g. Tajima's D or nucleotide diversity bins in binning_map.tsv), with gene/transcript IDs matched to the scRNA data set identifiers. The index itself is the TEI analogue — a weighted mean of expression where weights are the gene category (age class or bin group). Then group cells by the developmental-stage annotation stored in the AnnData metadata and aggregate the per-cell index values within each stage (mean/aggregate index per stage). Finally write the stage-level summary as a table such as binned_index_per_stage.tsv and inspect whether the index rises or falls across developmental time, as in the nematode and zebrafish examples. oggmap can also visualize the category expression profiles to verify stage-resolved trends.

## Related tools

- **oggmap** (provides the TEI calculation on Scanpy objects, get_bins for alternative gene-category weights, and visualization of category expression profiles per stage) — https://github.com/kullrich/oggmap
- **Scanpy** (holds the AnnData scRNA object on which the weighted index is calculated and whose cell metadata supplies the developmental-stage grouping)
- **Python** (implementation language; oggmap v0.0.2 was developed using Python 3.10)

## Examples

```
import scanpy as sc; from oggmap import tei; adata = sc.read_h5ad('nematode_counts.h5ad'); adata = tei.tei(adata, gene_age='binning_map.tsv'); adata.to_df('tei').groupby(adata.obs['stage']).mean().to_csv('binned_index_per_stage.tsv')
```

## Evaluation signals

- binned_index_per_stage.tsv contains one aggregated index value per developmental stage, with no stages missing
- Every gene contributing to the per-cell index has an ID matched to the scRNA object (no unmapped genes silently excluded)
- Per-stage index values fall within the plausible range for a weighted expression index given the gene group labels used as weights
- Stage-level trend (e.g. rising index across development in the nematode/zebrafish examples) is reproducible when re-run and consistent with the per-cell index distribution
- The oggmap profile plot of the gene category matches the aggregated per-stage table

## Limitations

- oggmap is the index/weighting layer only — the downstream Scanpy toolkit is required for the scRNA object handling, so pipeline failures can stem from either component
- Weights are limited to the discrete gene groups in the mapping table (age classes or get_bins bins); a continuous metric must first be discretized into bins
- The discussion section provides no limitations content or versioned dependency details for the 0.0.1 release, so release-specific edge cases are not documented in the source

## Evidence

- [other] TEI calculation on Scanpy objects: [evidence span withheld — anchor status: not_found]
- [methods] calculate and visualize TEI: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [other] aggregate per stage: [evidence span withheld — anchor status: not_found]
- [abstract] visualization of category profiles: "oggmap can visualize the individual gene category (e.g. age class, nucleotide diversity bin) and their corresponding expression profiles to investigate scRNA-based cell type assignments"
- [methods] ID matching prerequisite: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
