---
name: root-cell-selection-from-activity-scores
description: 'Use when a scRNA-seq count matrix needs a biologically motivated trajectory
  root: cluster identities are known (e.g., basal vs. differentiated epithelial cells)
  but the origin for pseudotime inference must be chosen objectively from pluripotency/differentiation
  activity scores.'
when_to_use_negative:
- When the goal is annotating cells by biological process rather than choosing a trajectory
  root — use ORIGINS2, which computes PPIN activity for 18 biological processes and
  supports custom PPINs via build_ppin()
- When the origin cluster (e.g., basal/stem compartment) cannot be identified in advance
  and root selection must be data-driven across the full dataset
- When a Python workflow is required and ORIGINS R dependencies are impractical —
  consider PyOrigins instead of forcing the R path
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: ORIGINS
  role: R package that computes the differentiation PPIN activity score used to rank
    cells and select the trajectory root
  repo: https://github.com/danielasenraoka/ORIGINS
- name: remotes
  role: Used to install ORIGINS from GitHub via remotes::install_github
- name: R
  role: Runtime environment for ORIGINS and the downstream Slingshot trajectory inference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:9118e2fadaa82bdb8f3d8c76bc2b803446b97a8cd9e5276a55539ee8539068f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
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
yes_rate: 0.75
n_claims: 8
---

# root-cell-selection-from-activity-scores

## Summary

Selects trajectory root cells by computing differentiation PPI network activity scores with the ORIGINS R package on a scRNA-seq expression matrix, then designating the highest-activity cells within the stem/progenitor (e.g., basal) cluster as the Slingshot trajectory origin.

## When to use

Use when a scRNA-seq count matrix needs a biologically motivated trajectory root: cluster identities are known (e.g., basal vs. differentiated epithelial cells) but the origin for pseudotime inference must be chosen objectively from pluripotency/differentiation activity scores.

## When NOT to use

- When the goal is annotating cells by biological process rather than choosing a trajectory root — use ORIGINS2, which computes PPIN activity for 18 biological processes and supports custom PPINs via build_ppin()
- When the origin cluster (e.g., basal/stem compartment) cannot be identified in advance and root selection must be data-driven across the full dataset
- When a Python workflow is required and ORIGINS R dependencies are impractical — consider PyOrigins instead of forcing the R path

## Inputs

- Gene-by-cell scRNA-seq expression/count matrix (rows = genes, columns = cells), e.g. GSE113197 breast epithelial cells
- differentiation_edges data frame of PPIN edges (bundled within the ORIGINS package)
- Cell cluster labels identifying the candidate origin cluster (e.g., basal cluster)

## Outputs

- Per-cell differentiation activity score vector (diff_activity)
- A set of root cells (highest-activity cells within the origin cluster) for Slingshot
- Inferred trajectory lineages rooted at the selected cells (e.g., basal→L1 and basal→L2 through an intermediate state)

## How to apply

Load the gene-by-cell count matrix (e.g., breast epithelial dataset GSE113197) in R and install/load ORIGINS via remotes::install_github("danielasenraoka/ORIGINS"). Compute per-cell scores with diff_activity <- activity(expression_matrix, differentiation_edges), where differentiation_edges is the differentiation-associated PPIN bundled in the package's data directory and the matrix has genes as rows and cells as columns. Rank cells by differentiation activity within the cluster expected to contain the origin (e.g., the basal cluster of the breast epithelium) and designate the cells with the highest scores as the trajectory root. Pass these root cells to Slingshot so the inferred trajectory starts at the origin and progresses outward; in the breast example this yields a basal-to-intermediate bifurcation into L1 and L2 lineages. Rationale: ORIGINS was created to identify stem and progenitor cells as trajectory origins, and is more efficient than LandSCENT and requires less RAM than LandSCENT and CytoTRACE.

## Related tools

- **ORIGINS** (R package that computes the differentiation PPIN activity score used to rank cells and select the trajectory root) — https://github.com/danielasenraoka/ORIGINS
- **remotes** (Used to install ORIGINS from GitHub via remotes::install_github)
- **R** (Runtime environment for ORIGINS and the downstream Slingshot trajectory inference)

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- diff_activity returns one score per cell, with length equal to the number of columns (cells) of the expression matrix
- The selected root cells fall inside the expected origin cluster (e.g., basal) and hold the maximum activity scores within it
- Slingshot run with these roots produces a trajectory starting at the origin and bifurcating through a shared intermediate state into the expected lineages (basal→L1 and basal→L2 for the breast dataset)
- Activity scores are normalized by PPIN edge count and mean expression, so values are comparable across cells and can be benchmarked against LandSCENT/CytoTRACE outputs
- A tradeSeq test along the two inferred lineages yields lineage-specific differential expression patterns consistent with basal→L1/basal→L2 progression

## Limitations

- The differentiation PPIN is the only network bundled with ORIGINS; PPINs for other biological processes require the extended ORIGINS2 package or a custom build_ppin() call
- ORIGINS2's activity() takes a normalization flag: normalization=FALSE returns the sum over PPIN edges, normalization=TRUE normalizes by the number of PPIN edges — the choice affects cross-dataset comparability
- Root selection presumes the origin population is captured within a single cluster (the highest-activity cells within the basal cluster in the breast example)

## Evidence

- [readme] This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference.: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
- [methods] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [methods] differentiation_edges is a data frame containing the edges of the Protein-protein Interaction Network (PPIN) asociated with differentiation: "differentiation_edges is a data frame containing the edges of the Protein-protein Interaction Network (PPIN) asociated with differentiation, it is already loaded within the package."
- [abstract] We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs.: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [readme] A normalization factor of e · μ_i is introduced to account for graph size differences and differences in the mean expression.: [evidence span withheld — anchor status: external_doc]
