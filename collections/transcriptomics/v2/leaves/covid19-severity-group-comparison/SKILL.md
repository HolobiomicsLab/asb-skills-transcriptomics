---
name: covid19-severity-group-comparison
description: Use when moderate-vs-severe group abundance differences must be quantified
  at single-cell resolution on scMerge2-integrated COVID-19 data, e.g. to find which
  annotated cell types carry the highest proportion of severe-outcome-associated cells.
when_to_use_negative:
- The expression matrix has not been batch-integrated (e.g. raw counts from a single
  study or unintegrated multi-study counts), since severity-group abundance comparisons
  would then be confounded by dataset variability
- The input is already aggregated to sample-level or pseudo-bulk feature summaries,
  so per-cell DA-seq scores cannot be computed or overlaid on a per-cell UMAP
- The goal is cell-type-level differential expression of gene signatures rather than
  differential abundance of cell states across severity groups
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix and integrated embedding of the COVID-19
    collection on which the 30-PC DA-seq comparison of moderate vs severe patients
    is run
  repo: https://github.com/SydneyBioX/scMerge
- name: scMerge
  role: R/Bioconductor package family providing scMerge2; source of vignettes and
    SEG lists used for the integration step upstream of the severity comparison
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:c282821b9459ab5bd189eea3d12a93c733873c115d193c43cfbb25b91de3c6e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge
  - scMerge2
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# covid19-severity-group-comparison

## Summary

Identifies cell types associated with COVID-19 disease severity by running DA-seq differential abundance analysis on the scMerge2-integrated COVID-19 single-cell data between moderate and severe patients, then aggregating per-cell DA scores by annotated cell type.

## When to use

Use when moderate-vs-severe group abundance differences must be quantified at single-cell resolution on scMerge2-integrated COVID-19 data, e.g. to find which annotated cell types carry the highest proportion of severe-outcome-associated cells.

## When NOT to use

- The expression matrix has not been batch-integrated (e.g. raw counts from a single study or unintegrated multi-study counts), since severity-group abundance comparisons would then be confounded by dataset variability
- The input is already aggregated to sample-level or pseudo-bulk feature summaries, so per-cell DA-seq scores cannot be computed or overlaid on a per-cell UMAP
- The goal is cell-type-level differential expression of gene signatures rather than differential abundance of cell states across severity groups

## Inputs

- scMerge2-adjusted single-cell expression matrix from the COVID-19 data collection (20 studies, ~5 million cells, 963 donors, 1298 samples)
- Cell-type annotations for the integrated cells
- Disease severity labels (moderate vs severe) per donor/sample

## Outputs

- Per-cell differential abundance score file from DA-seq (k swept 50–500)
- Cell-type DA-proportion table ranking cell types by proportion of severe-outcome-associated cells
- UMAP embedding of the adjusted data overlaid with per-cell DA scores

## How to apply

Load the scMerge2-adjusted COVID-19 single-cell expression matrix (scMerge2 integrated five million cells from over 1000 samples across 20 studies) and reduce it to the top 30 principal components to form the embedding. Run DA-seq on the 30-PC embedding comparing moderate versus severe patients, sweeping the neighborhood size k from 50 to 500 so per-cell differential abundance scores are computed across multiple scales rather than at a single k. Overlay the per-cell DA scores on the UMAP embedding of the adjusted data to visually confirm that severe-outcome-associated regions are spatially coherent rather than scattered batch artifacts. Aggregate per-cell DA scores by annotated cell type to compute, for each cell type, the proportion of cells belonging to severe-outcome-associated DA regions; the expected result is that neutrophils hold the highest proportion, consistent with neutrophil accumulation marking critical COVID-19 illness. Export both the per-cell DA score file and the cell-type DA-proportion table as the deliverables.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix and integrated embedding of the COVID-19 collection on which the 30-PC DA-seq comparison of moderate vs severe patients is run) — https://github.com/SydneyBioX/scMerge
- **scMerge** (R/Bioconductor package family providing scMerge2; source of vignettes and SEG lists used for the integration step upstream of the severity comparison) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge")
```

## Evaluation signals

- Neutrophils show the highest proportion of severe-outcome-associated cells in the cell-type DA-proportion table, consistent with neutrophil accumulation marking critical COVID-19 illness
- Per-cell DA scores were computed across the k sweep from 50 to 500, not at a single neighborhood size
- Overlaying per-cell DA scores on the UMAP of the adjusted data shows severe-associated regions occupying coherent spatial neighborhoods
- The DA-seq embedding was built from exactly the top 30 principal components of the scMerge2-adjusted matrix
- Both the per-cell DA score file and the cell-type DA-proportion table were exported

## Limitations

- The comparison is built on the scMerge2-integrated COVID-19 data; DA results depend on the quality of that integration and on disease-severity label availability per donor
- Only moderate vs severe is contrasted; other severity strata or contrasts require re-running the DA-seq workflow
- Embedding-only outputs can be harder to interpret for downstream features, and per-cell DA scores should be interpreted against cell-type annotations rather than treated as gene-level signatures

## Evidence

- [intro] scMerge2 integration of the COVID-19 collection provides the embedding used for severity comparison: "the integration using scMerge2 improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity and facilitates diverse single-cell downstream"
- [discussion] Scale of the COVID-19 dataset used in the severity comparison: "Our algorithm enabled the atlas-scale integration of 20 global COVID-19 studies with around 5 million cells from 963 donors, 1298 samples."
- [abstract] Cell-type-derived signatures better discriminate disease progression: [evidence span withheld — anchor status: not_found]
- [readme] scMerge package provides the implementation and vignettes: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
- [readme] Installation route for the tooling used in the workflow: [evidence span withheld — anchor status: external_doc]
