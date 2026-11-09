---
name: scrna-seq-network-activity-scoring
description: Use when you have a normalized scRNA-seq expression matrix (genes as
  rows, cells as columns) and need per-cell activity scores for biological processes
  such as cell differentiation, proliferation, or immune response.
when_to_use_negative:
- The expression matrix is raw counts rather than a normalized expression matrix,
  since activity() expects normalized values
- You need pathway scores for a biological process with no bundled edge set and lack
  a gene ID list plus the full_human_ppin to build a custom PPIN
- The analysis goal is cell-type clustering from the transcriptome rather than scoring
  predefined biological-process network activity
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package providing the activity() and build_ppin() functions plus 18 pre-built
    PPIN edge sets and the full_human_ppin adjacency list
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: used to install ORIGINS2 from GitHub via remotes::install_github()
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: package dependency used to filter/prune PPIN edge lists (e.g., removing edges
    exclusive to negative regulators)
- name: magrittr
  role: package dependency providing the %>% pipe for edge-list filtering workflows
merged_aliases:
- protein-network-activity-scoring-scrna-seq
merged_alias_records:
- alias: scrna-seq-gene-set-activity-scoring
  slug: scrna-seq-gene-set-activity-scoring
  cosine_score: 0.8587
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:556f4e6c4559a4b6512291dfff2bc1a5da05bcac236e811a7b498af05f5a1df4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielasenraoka/ORIGINS2
  repo_url: https://github.com/danielasenraoka/ORIGINS2
  tools:
  - ORIGINS
  - ORIGINS2
  - R
  - dplyr
  - magrittr
  - remotes
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-network-activity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell activity of protein-protein interaction networks (PPIN) associated with biological processes from normalized scRNA-seq expression matrices using the ORIGINS2 R package, enabling pathway-level scoring and cell annotation.

## When to use

Use when you have a normalized scRNA-seq expression matrix (genes as rows, cells as columns) and need per-cell activity scores for biological processes such as cell differentiation, proliferation, or immune response.

## When NOT to use

- The expression matrix is raw counts rather than a normalized expression matrix, since activity() expects normalized values
- You need pathway scores for a biological process with no bundled edge set and lack a gene ID list plus the full_human_ppin to build a custom PPIN
- The analysis goal is cell-type clustering from the transcriptome rather than scoring predefined biological-process network activity

## Inputs

- normalized scRNA-seq expression matrix (genes as rows, cells as columns)
- PPIN adjacency edge set for the biological process of interest (e.g., differentiation_edges) or a gene ID list plus full_human_ppin to build a custom PPIN
- normalization logical parameter (TRUE/FALSE)

## Outputs

- per-cell activity score vector (sum of all PPIN edge activities per cell when normalization = FALSE, or edge-count-normalized scores when normalization = TRUE)
- cells x processes activity score table for multiple biological processes

## How to apply

Install ORIGINS2 from GitHub with remotes::install_github('danielasenraoka/ORIGINS2') and load it with library(ORIGINS2). Call activity(expression_matrix, adjacency_edges, normalization) where expression_matrix is the normalized matrix (rows = genes, columns = cells) and adjacency_edges is one of the 18 pre-built PPIN edge sets bundled with the package (e.g., differentiation_edges, dna_repair_edges, adaptive_immune_response_edges). Set normalization = FALSE to return the unnormalized sum of all PPIN edge activities per cell, or TRUE to normalize by the number of PPIN edges. For processes not covered by the bundled edge sets, build a custom PPIN with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), using the full Human PPIN adjacency list included in the package. Per-cell score vectors can then be overlaid on UMAP embeddings or correlated with other scoring methods such as AUCell and mean-expression (Average).

## Related tools

- **ORIGINS2** (R package providing the activity() and build_ppin() functions plus 18 pre-built PPIN edge sets and the full_human_ppin adjacency list) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (used to install ORIGINS2 from GitHub via remotes::install_github()) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (package dependency used to filter/prune PPIN edge lists (e.g., removing edges exclusive to negative regulators))
- **magrittr** (package dependency providing the %>% pipe for edge-list filtering workflows)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- With normalization = FALSE, each activity vector value must equal the sum of all PPIN edge contributions for that cell and be finite
- With normalization = TRUE, scores are normalized by the number of edges of the PPIN, so scores should be comparable across PPINs of different sizes
- Overlaying activity on UMAP should reproduce expected patterns, e.g., mitotic cell cycle, DNA replication and DNA repair correlated and highly activated in the tumor-cell region of the TNBC sample
- ORIGINS2 scores should positively correlate with AUCell and mean-expression (Average) scores across the same BP gene sets (reported mean Pearson rho: ORIGINS2-AUCell = 0.59, ORIGINS2-Average = 0.79, AUCell-Average = 0.66)
- build_ppin output should be a data frame of adjacency edges restricted to the supplied gene ID list and the full Human PPIN

## Limitations

- Excluding exclusive negative regulators from the PPIN lowers absolute activity values (though relative results on the UMAP color scale remain similar)
- Only 18 biological-process PPIN edge sets are pre-loaded in the package; other processes require build_ppin with the included full_human_ppin
- Tumor cells forming a single UMAP group can keep diverse biological programs active in different subregions, so cluster-level summaries may mask intra-cluster heterogeneity

## Evidence
