---
name: lancaster-p-value-combination
description: Use when building reference cell-type markers from multiple DE tests
  and you need a significance-driven consensus (p-value combination) rather than rank-only
  aggregation of top-N markers.
when_to_use_negative:
- When only the top-N markers per DE test are needed and rank-only robust rank aggregation
  (marker_identification_fast) suffices without p-value combination
- When the reference data lacks cell-type labels or the DE tests do not produce p-values
  suitable for Lancaster combination
- When the goal is annotating unseen cell types absent from the reference — MASI must
  be combined with SCCAF for that
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Provides marker_identification_LC for Lancaster p-value combination of significant
    markers across DE tests, alongside marker_identification/marker_identification_fast
    for the rank-aggregation path
  repo: https://github.com/hayatlab/MASI
- name: RobustRankAggreg
  role: R package required by MASI for the complementary robust rank aggregation path
    over per-DE-test ranked lists
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:4d41611ed3bc4a45d936979770537ea3f355ad6663f905a015fa83cb6796b591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - RobustRankAggreg
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# lancaster-p-value-combination

## Summary

MASI's ensemble marker-identification path that aggregates statistically significant markers from N differential expression tests into a single final ranked cell-type marker list using Lancaster's method for combining p-values, implemented as marker_identification_LC.

## When to use

Use when building reference cell-type markers from multiple DE tests and you need a significance-driven consensus (p-value combination) rather than rank-only aggregation of top-N markers.

## When NOT to use

- When only the top-N markers per DE test are needed and rank-only robust rank aggregation (marker_identification_fast) suffices without p-value combination
- When the reference data lacks cell-type labels or the DE tests do not produce p-values suitable for Lancaster combination
- When the goal is annotating unseen cell types absent from the reference — MASI must be combined with SCCAF for that

## Inputs

- Processed reference single-cell expression dataset (AnnData) with cell-type labels in a 'cell_type' obs column
- Per-cell-type differential expression test results (gene rankings with p-values) from MASI's marker_identification / marker_identification_fast functions

## Outputs

- cell_markers: a dict of per-cell-type consensus ranked marker lists (e.g. writable to cell_markers.csv), each cell type's markers ordered by combined significance
- Downstream inputs for masi.gene2cell cell-type score matrix computation

## How to apply

Load the processed reference single-cell dataset from the Figshare deposit and normalize raw counts via LogNormalization (scale factor 10,000 followed by log-transformation). Run per-cluster differential expression tests to rank genes and obtain p-values for each cell type. For each cell type, compile the statistically significant markers across the N DE tests and combine their p-values with Lancaster's method (MASI's marker_identification_LC) to produce a consensus ranked marker list; this complements the rank-based path that compiles the top 20 markers from each DE test via robust rank aggregation (which depends on the R packages Seurat and RobustRankAggreg). The resulting dict of per-cell-type ranked markers feeds MASI's gene2cell scoring and MACA-based label transfer. Judge correctness by checking that each cell type's output markers are ordered by consensus significance and derive from the pooled per-test candidate set.

## Related tools

- **MASI** (Provides marker_identification_LC for Lancaster p-value combination of significant markers across DE tests, alongside marker_identification/marker_identification_fast for the rank-aggregation path) — https://github.com/hayatlab/MASI
- **RobustRankAggreg** (R package required by MASI for the complementary robust rank aggregation path over per-DE-test ranked lists)

## Examples

```
import MASI as masi; cell_markers = masi.marker_identification_LC(source_data=source)
```

## Evaluation signals

- Output marker table contains a per-cell-type ranked list in which each cell type has markers ordered by consensus (combined-significance) rank
- Markers in the final list come from the pooled candidate set of markers passing the DE tests' significance criteria, not from a single test
- Cross-check against the RRA path: markers ranked via Lancaster combination should be consistent with the top-20-per-DE-test compilation for well-supported cell types
- Resulting cell_markers dict is accepted by masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) without key errors
- Downstream MACA label transfer using these markers reproduces the paper's marker-assisted annotation behavior

## Limitations

- Annotation accuracy depends on reference data quality; the choice and resolution of the reference are critical to downstream analysis
- MASI (including this marker-construction step) cannot annotate unseen cell types not present in the reference; combine with SCCAF to identify subtypes within major cell types
- The exact significance cutoff and the applied PlinerScore percentile (described as variable 'X-percentile' with a default of 0.25) are not fully specified in the article, a reproducibility gap

## Evidence

- [discussion] Ensemble aggregation over N DE tests: "the top 20 marker genes from each DE test were compiled together"
- [readme] MASI marker identification and label transfer overview: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
- [discussion] Input normalization requirement: "Raw gene expression counts data were ‘LogNormalized’, which divides the total count in that cell and multiplies it by a scale factor of 10,000"
- [readme] R dependency for rank aggregation: "install.packages('Seurat')
    install.packages('RobustRankAggreg')"
