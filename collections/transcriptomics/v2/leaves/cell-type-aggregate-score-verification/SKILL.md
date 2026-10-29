---
name: cell-type-aggregate-score-verification
description: Use when per-cell pluripotency/differentiation scores from multiple methods
  (ORIGINS, LandSCENT, CytoTRACE) must be averaged within annotated cell types and
  their orderings compared for consistency, e.g. on the breast epithelial sample.
when_to_use_negative:
- The expression matrix is not oriented as genes (rows) x cells (columns) or is not
  normalized — the ORIGINS activity() function expects that layout and the PyOrigins
  formula normalizes by edge count and mean cell expression
- The goal is PPIN activity for a biological process other than differentiation —
  use ORIGINS2 or PyOrigins with a custom PPIN instead of the bundled differentiation_edges
- Only a single scoring method is used with no cross-method ordering to verify — the
  skill's core value is comparing rank orderings across ORIGINS, LandSCENT and CytoTRACE
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: ORIGINS
  role: Computes per-cell differentiation activity of the bundled differentiation
    PPIN via activity(expression_matrix, differentiation_edges); installed with remotes::install_github('danielasenraoka/ORIGINS')
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Benchmark method providing the SR (single-cell pluripotency) score used as
    the reference ordering against which ORIGINS activity is compared
- name: CytoTRACE
  role: Benchmark pluripotency method; its ordering (L1 > L2 > basal) is the discordant
    result the verification step must detect and report
- name: R
  role: Language in which ORIGINS is distributed as an open source R package; all
    aggregation and CSV export performed in R
  repo: https://github.com/danielasenraoka/ORIGINS
- name: remotes
  role: R package required to install ORIGINS from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:8e2c4d512a24b91c8c058b52f44b2898d9709a9ef8e70373bdc4e886e9c63fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - remotes
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-aggregate-score-verification

## Summary

A verification procedure that aggregates per-cell pluripotency scores (ORIGINS activity, activity-HVF, LandSCENT SR, CytoTRACE) within annotated cell types of the breast epithelial scRNA-seq dataset (GSE113197) and cross-checks whether the methods produce consistent cell-type orderings, flagging disagreement such as CytoTRACE ranking L1 > L2 > basal while ORIGINS/LandSCENT rank basal highest.

## When to use

Use when per-cell pluripotency/differentiation scores from multiple methods (ORIGINS, LandSCENT, CytoTRACE) must be averaged within annotated cell types and their orderings compared for consistency, e.g. on the breast epithelial sample.

## When NOT to use

- The expression matrix is not oriented as genes (rows) x cells (columns) or is not normalized — the ORIGINS activity() function expects that layout and the PyOrigins formula normalizes by edge count and mean cell expression
- The goal is PPIN activity for a biological process other than differentiation — use ORIGINS2 or PyOrigins with a custom PPIN instead of the bundled differentiation_edges
- Only a single scoring method is used with no cross-method ordering to verify — the skill's core value is comparing rank orderings across ORIGINS, LandSCENT and CytoTRACE

## Inputs

- Annotated breast epithelial scRNA-seq count matrix from GSE113197, oriented genes (rows) x cells (columns)
- Cell-type annotations for breast epithelial clusters (basal, L1, L2)
- differentiation_edges data frame (PPIN bundled within the ORIGINS package)
- Per-cell pluripotency scores from LandSCENT (SR) and CytoTRACE computed on the same expression matrix

## Outputs

- Per-cell differentiation activity vector (ORIGINS activity) and activity-HVF variant
- Cell-type-level averaged score table comparing SR, activity, activity-HVF and CytoTRACE (e.g. pluripotency_scores_breast.csv)
- Reported cell-type ordering per method (basal highest for SR/activity/activity-HVF; L1 > L2 > basal for CytoTRACE)

## How to apply

Retrieve the annotated breast epithelial count matrix (GEO accession GSE113197) and orient it as genes (rows) x cells (columns), as required by the ORIGINS activity() function. Compute per-cell differentiation activity with activity(expression_matrix, differentiation_edges), where differentiation_edges is the bundled differentiation PPIN data frame loaded with the ORIGINS package; also compute the activity-HVF variant restricted to highly variable features. Run LandSCENT (SR score) and CytoTRACE on the same expression matrix to obtain comparison pluripotency scores. Average each score within the annotated cell types (basal, L1, L2) and verify the expected orderings: basal highest for SR, activity and activity-HVF with L1 second for activity/HVF, while CytoTRACE orders L1 > L2 > basal. Write the aggregated summary table (e.g. pluripotency_scores_breast.csv) and treat method discordance, not just concordance, as the reportable finding.

## Related tools

- **ORIGINS** (Computes per-cell differentiation activity of the bundled differentiation PPIN via activity(expression_matrix, differentiation_edges); installed with remotes::install_github('danielasenraoka/ORIGINS')) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Benchmark method providing the SR (single-cell pluripotency) score used as the reference ordering against which ORIGINS activity is compared)
- **CytoTRACE** (Benchmark pluripotency method; its ordering (L1 > L2 > basal) is the discordant result the verification step must detect and report)
- **R** (Language in which ORIGINS is distributed as an open source R package; all aggregation and CSV export performed in R) — https://github.com/danielasenraoka/ORIGINS
- **remotes** (R package required to install ORIGINS from GitHub)

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- ORIGINS activity output is a per-cell vector of the same length as the number of columns (cells) of the input expression matrix
- Within-type means reproduce the reported ordering: basal highest for SR, activity and activity-HVF, with L1 second for activity/activity-HVF
- CytoTRACE cell-type means reproduce the discordant ordering L1 > L2 > basal on the breast sample — its presence confirms the comparison was run on identical inputs
- Cell-type means are computed over exactly the three annotated breast clusters (basal, L1, L2) and the summary table exports without missing groups
- All four methods were run on the same expression matrix (GSE113197), so scores are comparable across methods

## Limitations

- Methods disagree on this dataset: CytoTRACE does not coincide with the ranking of the other methods, so agreement across tools cannot be assumed
- ORIGINS activity score depends on the bundled differentiation PPIN; its edges are fixed within the package and the score is normalized by edge count and mean expression (per the PyOrigins formula) — changing the network changes the score
- ORIGINS requires the expression matrix oriented as genes x cells; wrong orientation silently produces invalid results

## Evidence

- [methods] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell.: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [abstract] We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020): "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020)"
- [readme] differentiation_edges is a data frame containing the edges of the Protein-protein Interaction Network (PPIN) asociated with differentiation, it is already loaded within the package.: "differentiation_edges is a data frame containing the edges of the Protein-protein Interaction Network (PPIN) asociated with differentiation, it is already loaded within the package."
