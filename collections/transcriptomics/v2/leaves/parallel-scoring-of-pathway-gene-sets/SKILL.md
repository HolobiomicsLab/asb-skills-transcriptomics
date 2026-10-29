---
name: parallel-scoring-of-pathway-gene-sets
description: Use when a cell-by-gene expression matrix must be converted to a cell-by-pathway
  PAS matrix for >10^3 pathways across ≥10^5 cells and standard UCell/AddModuleScore
  would be too slow.
when_to_use_negative:
- The input is already a PAS/feature matrix or pseudobulk-aggregated table rather
  than a cell-level expression matrix — scoring would be redundant.
- The study has fewer than 15 samples and the downstream goal is MOFA-based multicellular
  pathway module identification (scPAFA's downstream step requires ≥15 samples).
- Only a small number of pathways (<10^3) and a small dataset are involved, where
  existing tools like UCell or AddModuleScore suffice without the parallel implementation.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scPAFA
  role: Python library whose fast_ucell function performs the parallel, chunked PAS
    computation that reproduces UCell scores
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: R reference method whose Mann-Whitney U rank-based PAS fast_ucell reimplements
    and matches exactly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:126142f73d85f11a75c46d8706380c669dfb7466d14375d9141bbcc5ec3d06c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# parallel-scoring-of-pathway-gene-sets

## Summary

Compute cell-level pathway activity scores (PAS) for large gene-set collections on large-scale scRNA-seq data using scPAFA's fast_ucell, a vectorized, multi-core Python reimplementation of the R UCell method based on within-cell expression rankings.

## When to use

Use when a cell-by-gene expression matrix must be converted to a cell-by-pathway PAS matrix for >10^3 pathways across ≥10^5 cells and standard UCell/AddModuleScore would be too slow.

## When NOT to use

- The input is already a PAS/feature matrix or pseudobulk-aggregated table rather than a cell-level expression matrix — scoring would be redundant.
- The study has fewer than 15 samples and the downstream goal is MOFA-based multicellular pathway module identification (scPAFA's downstream step requires ≥15 samples).
- Only a small number of pathways (<10^3) and a small dataset are involved, where existing tools like UCell or AddModuleScore suffice without the parallel implementation.

## Inputs

- Single-cell gene expression matrix (cell x gene, e.g., sparse matrix from an scRNA-seq dataset such as CRC tumor epithelial cells)
- Pathway dictionary (Python dict: pathway name -> list of genes), e.g., 1629 pathways from NCATS BioPlanet and the 3CA signature set
- Max rank threshold parameter (r_max=2000)
- Chunk size (100,000 cells) and CPU core count for parallelization

## Outputs

- Cell-by-pathway PAS matrix (e.g., 108,497 cells x 1629 pathways) with values in [0, 1], written to pas_matrix.csv

## How to apply

Load the single-cell gene expression matrix together with a Python pathway dictionary (pathway name -> list of genes), assembled from NCATS BioPlanet or MsigDB. Run scPAFA's fast_ucell, which computes per-cell Mann-Whitney U statistics for each pathway gene set on within-cell expression rankings with a maximum rank threshold of r_max=2000. Process cells in chunks of 100,000 to bound memory, truncating the dense within-cell rank matrix back to sparse format after ranking each chunk, and partition the pathway dictionary across multiple CPU cores so pathway scoring proceeds in parallel within each cell chunk. Assemble chunk results into the full cell-by-pathway PAS matrix (e.g., 108,497 cells x 1629 pathways) and write it out (e.g., pas_matrix.csv). The ranking-based approach makes scores insensitive to the surrounding dataset composition, and fast_ucell reproduces the same PAS as UCell while cutting runtime by over 40-fold versus existing tools.

## Related tools

- **scPAFA** (Python library whose fast_ucell function performs the parallel, chunked PAS computation that reproduces UCell scores) — https://github.com/ZhuoliHuang/scPAFA
- **UCell** (R reference method whose Mann-Whitney U rank-based PAS fast_ucell reimplements and matches exactly)

## Examples

```
from scpafa import fast_ucell; pas = fast_ucell(expr_matrix, pathway_dict, max_rank=2000, chunk_size=100000, n_cores=N); pas.to_csv('pas_matrix.csv')
```

## Evaluation signals

- Output PAS matrix has exactly 108,497 rows (cells) and 1629 columns (pathways).
- All PAS values fall within [0, 1].
- fast_ucell scores reproduce reference UCell PAS for the same cells and pathways (scPAFA reports fast_ucell yields the same PAS as UCell).
- Runtime scales near-linearly with cores: PAS for ~1383 pathways on million-cell data should complete within ~30 minutes.
- Pathway dictionary keys match the scored pathway set (no pathways silently dropped from the input dictionary).

## Limitations

- Rank-based PAS (like AUCell) is insensitive to dataset composition but pseudobulk integration of the PAS matrix leads to information loss since PAS of multiple cells is aggregated.
- Using a redundant pathway set increases result complexity, while too few or biologically misaligned pathways may cause information loss.
- Chunked ranking must bound memory; very large r_max or chunk sizes may blow up memory since the dense rank matrix must be truncated back to sparse format.

## Evidence

- [discussion] The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [abstract] scPAFA can achieve over 40-fold reductions in the runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [readme] In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’: "In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
- [intro] the abovementioned methods exhibit tardiness and computational inefficacy in large-scale scRNA-seq datasets, especially in situations where a large number of pathways (>103) is required: "the abovementioned methods exhibit tardiness and computational inefficacy in large-scale scRNA-seq datasets, especially in situations where a large number of pathways (>103) is required to be"
