---
name: cell-type-aggregation-of-per-cell-scores
description: Use when per-cell stemness scores (ORIGINS activity, LandSCENT SR, CytoTRACE)
  have been computed on annotated scRNA-seq data and you need per-type averages and
  rankings to benchmark which method correctly identifies stem cells.
when_to_use_negative:
- When no cell-type annotations exist — aggregation requires a prior per-cell type
  label; use the scores directly or perform clustering/annotation first
- When the goal is trajectory inference rather than benchmarking — ORIGINS scores
  are intended to select origin cells, and aggregation into per-type means discards
  single-cell resolution
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dplyr
  role: R package used to group per-cell scores by annotated cell type and compute
    mean scores per type (listed under Imports)
- name: ORIGINS
  role: Computes the per-cell differentiation activity score that is aggregated per
    cell type
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Provides Signal Rank (SR) per-cell stemness scores benchmarked via per-type
    aggregation
- name: CytoTRACE
  role: Provides per-cell stemness scores benchmarked via per-type aggregation
- name: R
  role: Environment in which scores are computed and aggregated with dplyr
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:6689dc8ff30bee1c0d48be27d31a58d40d8ed53a04c9d1dab8c6cbbfb46ec031
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - dplyr
  - remotes
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cell-type-aggregation-of-per-cell-scores

## Summary

Aggregate per-cell pluripotency/stemness scores (ORIGINS activity, LandSCENT SR, CytoTRACE) into average scores per annotated cell type with dplyr, then rank cell types to assess whether expected stem/progenitor populations attain the highest scores.

## When to use

Use when per-cell stemness scores (ORIGINS activity, LandSCENT SR, CytoTRACE) have been computed on annotated scRNA-seq data and you need per-type averages and rankings to benchmark which method correctly identifies stem cells.

## When NOT to use

- When no cell-type annotations exist — aggregation requires a prior per-cell type label; use the scores directly or perform clustering/annotation first
- When the goal is trajectory inference rather than benchmarking — ORIGINS scores are intended to select origin cells, and aggregation into per-type means discards single-cell resolution

## Inputs

- Per-cell stemness score vector from ORIGINS activity(expression_matrix, differentiation_edges)
- Per-cell scores from LandSCENT Signal Rank (SR) and CytoTRACE run on the same expression matrix
- Cell-type annotation table mapping each cell (column) to an annotated type (e.g., stem, transit-amplifying, differentiated lineages)
- Normalized expression matrix (genes as rows, cells as columns), e.g., from GEO accession GSE125970 or GSE161383

## Outputs

- Per-cell-type table of average scores under each scoring method (e.g., colon_stemness_scores.csv, lung_stemness_scores.csv)
- Ranking of cell types by mean score for each of the four methods (SR, CytoTRACE, activity, HVF activity)

## How to apply

After computing per-cell scores with activity(expression_matrix, differentiation_edges), LandSCENT SR, and CytoTRACE on the same expression matrix (genes as rows, cells as columns), join the per-cell score vectors to the cell-type annotation table. Use dplyr (group_by cell type, summarise mean) to compute average scores per annotated type — e.g., stem cells, transit-amplifying (TA) cells, and differentiated lineages in the colon dataset (GSE125970), or AT2/club-like cells among ~30 types in the lung dataset (GSE161383). Rank cell types under each of the four scores (LandSCENT SR, CytoTRACE, ORIGINS activity, HVF-derived activity) to compare methods. Write per-cell and per-type results to a CSV (e.g., colon_stemness_scores.csv or lung_stemness_scores.csv). The acceptance check is that the biologically expected stem/progenitor type ranks first by average score; on the colon dataset the paper instead found TA cells rank first, revealing the methods' limits.

## Related tools

- **dplyr** (R package used to group per-cell scores by annotated cell type and compute mean scores per type (listed under Imports))
- **ORIGINS** (Computes the per-cell differentiation activity score that is aggregated per cell type) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Provides Signal Rank (SR) per-cell stemness scores benchmarked via per-type aggregation)
- **CytoTRACE** (Provides per-cell stemness scores benchmarked via per-type aggregation)
- **R** (Environment in which scores are computed and aggregated with dplyr) — https://github.com/danielasenraoka/ORIGINS

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges); per_type <- scores_df %>% group_by(cell_type) %>% summarise(mean_activity = mean(diff_activity), mean_SR = mean(SR), mean_cytotrace = mean(cytoTRACE)) %>% arrange(desc(mean_activity))
```

## Evaluation signals

- Per-type averages computed after group_by/summarise match per-cell scores joined on cell IDs (no dropped or duplicated cells)
- On colon (GSE125970, Colon-2): transit-amplifying (TA) cells rank first by average score for each method, reproducing the paper's reported finding
- On lung (GSE161383, D122): AT2/club-like cells have the highest mean SR (LandSCENT) and activity (ORIGINS) scores; expected stem/progenitor types fall within the top 8 mean ranks for SR, CytoTRACE and activity
- HVF-derived activity fails to order cell types by expected pluripotency, serving as a negative control in the benchmark
- Per-cell and per-type results are saved (e.g., colon_stemness_scores.csv) and rank ordering is consistent across methods

## Limitations

- On the colon sample, all four methods (LandSCENT, CytoTRACE, activity, activity HVF) were limited in finding the stem cells: TA cells, not stem cells, showed the highest average scores
- The HVF approximation fails to order cell types by expected pluripotency, so HVF-derived networks should not be relied on for stemness ordering
- ORIGINS2's activity() assumes a normalized expression matrix and normalization behavior depends on the normalization parameter (FALSE returns the sum of all PPIN edges; TRUE normalizes by the number of PPIN edges), affecting cross-dataset comparability

## Evidence

- [results] All four stemness methods (LandSCENT, CytoTRACE, activity (ORIGINS) and activity HVF (ORIGINS)) were limited in finding the stem cells in the colon sample: the transit amplifying (TA) cells, not the stem cells, showed the highest score values on average.: [evidence span withheld — anchor status: not_found]
- [results] In the lung sample, among the approximately 30 cell types analyzed, AT2/club-like cells had the highest average SR (LandSCENT) and activity (ORIGINS) pluripotency scores.: [evidence span withheld — anchor status: not_found]
- [readme] expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell. differentiation_edges is a data frame containing the edges of the Protein-protein Interaction Network (PPIN): "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
