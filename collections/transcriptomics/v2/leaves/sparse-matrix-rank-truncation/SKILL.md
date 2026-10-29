---
name: sparse-matrix-rank-truncation
description: Use when converting a large-scale sparse single-cell expression matrix
  into a cell-by-pathway PAS matrix via within-cell rank statistics (UCell-style),
  but the dense rank matrix would exceed memory.
when_to_use_negative:
- The dataset is small enough that the dense within-cell rank matrix fits in memory
  without chunking or truncation
- Using scPAFA's fast_score_genes (AddModuleScore-style scoring) instead of fast_ucell,
  since that pathway does not require within-cell expression ranking
- The analysis requires cross-condition differential statistics (SCPA-style) rather
  than per-cell pathway activity scoring
tools:
- name: scPAFA
  role: Provides the fast_ucell implementation whose chunked rank-and-truncate strategy
    this skill describes
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Reference R method whose Mann-Whitney U / within-cell ranking PAS the truncation
    must reproduce exactly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:8685d7dea962f4275ce7a97f8e986f451a6d26ee395a6d73a1b6fd2db23685c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - UCell
  - scPAFA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sparse-matrix-rank-truncation

## Summary

A memory-bounded technique used inside scPAFA's fast_ucell to rank gene expression within each cell (capped at r_max=2000) on 100,000-cell chunks and then truncate the dense within-cell rank matrix back to sparse form, enabling scalable cell-level pathway activity scoring.

## When to use

Use when converting a large-scale sparse single-cell expression matrix into a cell-by-pathway PAS matrix via within-cell rank statistics (UCell-style), but the dense rank matrix would exceed memory.

## When NOT to use

- The dataset is small enough that the dense within-cell rank matrix fits in memory without chunking or truncation
- Using scPAFA's fast_score_genes (AddModuleScore-style scoring) instead of fast_ucell, since that pathway does not require within-cell expression ranking
- The analysis requires cross-condition differential statistics (SCPA-style) rather than per-cell pathway activity scoring

## Inputs

- Sparse single-cell gene expression matrix (cell x gene) for a large scRNA-seq dataset (e.g., 108,497 CRC tumor epithelial cells)
- Pathway dictionary: Python dict of pathway name -> list of genes (e.g., 1629 pathways from NCATS BioPlanet and 3CA signature set)
- Max rank threshold r_max=2000 and chunk size 100,000 cells

## Outputs

- Dense-free cell x pathway PAS matrix (values in [0, 1]), written e.g. to pas_matrix.csv

## How to apply

Split the sparse cell-gene expression matrix into chunks of 100,000 cells; for each chunk, densify and compute within-cell gene expression rankings as required by the Mann-Whitney U statistic underlying UCell-style PAS. Cap ranks at the max rank threshold r_max=2000, since genes ranked beyond this threshold contribute negligibly to the statistic. After ranking each chunk, truncate the dense rank matrix back to sparse format before merging, which is the key step that bounds memory. Partition the pathway dictionary (e.g., 1629 pathways from NCATS BioPlanet and the 3CA signature set) across multiple CPU cores so scoring proceeds in parallel within each cell chunk, then assemble the chunk results into the full cell-by-pathway PAS matrix (e.g., 108,497 cells x 1629 pathways).

## Related tools

- **scPAFA** (Provides the fast_ucell implementation whose chunked rank-and-truncate strategy this skill describes) — https://github.com/ZhuoliHuang/scPAFA
- **UCell** (Reference R method whose Mann-Whitney U / within-cell ranking PAS the truncation must reproduce exactly)

## Evaluation signals

- Output PAS matrix has exactly the expected shape (e.g., 108,497 rows x 1629 columns for the CRC tumor epithelial cell task)
- All PAS values fall within [0, 1]
- fast_ucell scores reproduce reference UCell PAS for the same cells and pathways (scPAFA reports identical PAS to UCell)
- Peak memory stays bounded, confirming the dense rank matrix was truncated back to sparse after each 100,000-cell chunk
- Ranks respect the r_max=2000 cap, matching UCell behavior when the same max rank threshold is set

## Limitations

- Truncating the dense within-cell rank matrix to sparse format discards rank information beyond the r_max threshold; PAS is rank-based and therefore insensitive to dataset composition, but very small r_max could affect scores
- Ranking and chunking add overhead relative to simple scoring (fast_score_genes) on small datasets
- The resulting PAS matrix is per-cell; downstream pseudobulk integration aggregates cells and loses single-cell information

## Evidence

- [methods] Process cells in chunks of 100,000 to bound memory, truncating the dense within-cell rank matrix back to sparse format after ranking each chunk.: [evidence span withheld — anchor status: not_found]
- [discussion] The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [readme] single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’: "In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
- [other] vectorized computations with efficient chunking and a concurrent computation process that fully utilizes multi-core CPUs: [evidence span withheld — anchor status: not_found]
