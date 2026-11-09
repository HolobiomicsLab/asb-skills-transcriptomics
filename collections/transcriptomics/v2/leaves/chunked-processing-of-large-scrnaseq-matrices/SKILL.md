---
name: chunked-processing-of-large-scrnaseq-matrices
description: Use when a per-cell computation (e.g. ranking-based PAS scoring over
  >1000 pathways) must run on a cell-gene expression matrix too large to hold as a
  dense matrix in memory.
when_to_use_negative:
- The dataset is small enough to rank and score in one pass without chunking overhead
- The score requires cross-cell or dataset-level statistics that cannot be computed
  independently per cell chunk
- The study has fewer than 15 samples, so the downstream MOFA-based module step of
  scPAFA cannot be applied and PAS computation alone does not justify the pipeline
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scPAFA
  role: Provides fast_ucell, the chunked, multi-core Python implementation of ranking-based
    pathway activity scoring
  repo: github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Reference R method whose PAS output fast_ucell reproduces; defines the rank-based
    statistic and max rank threshold
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:5f957885898307dd4edc9610d61c7c8d9b818c75a2844ed6bf543f5d82c14daf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chunked-processing-of-large-scrnaseq-matrices

## Summary

Bounded-memory strategy from scPAFA's fast_ucell: process million-cell-scale gene expression matrices in fixed cell chunks (e.g. 100,000), rank and score within each chunk, then truncate the dense rank matrix back to sparse format before assembling the full cell-pathway PAS matrix.

## When to use

Use when a per-cell computation (e.g. ranking-based PAS scoring over >1000 pathways) must run on a cell-gene expression matrix too large to hold as a dense matrix in memory.

## When NOT to use

- The dataset is small enough to rank and score in one pass without chunking overhead
- The score requires cross-cell or dataset-level statistics that cannot be computed independently per cell chunk
- The study has fewer than 15 samples, so the downstream MOFA-based module step of scPAFA cannot be applied and PAS computation alone does not justify the pipeline

## Inputs

- Sparse cell-gene expression matrix (scRNA-seq, e.g. 108,497 CRC tumor epithelial cells)
- Pathway dictionary mapping pathway names to gene lists (e.g. 1629 pathways from NCATS BioPlanet + 3CA signature set)
- Chunk size (e.g. 100,000 cells) and maximum rank threshold r_max (e.g. 2000)

## Outputs

- Full cell-by-pathway PAS matrix (cells x pathways, values in [0, 1])
- CSV export of the PAS matrix (e.g. pas_matrix.csv)

## How to apply

Load the sparse single-cell gene expression matrix and the pathway dictionary, then partition cells into chunks (scPAFA's fast_ucell uses 100,000 cells per chunk) to bound memory. Within each chunk, compute within-cell expression rankings with a maximum rank threshold (r_max=2000, per UCell), calculate Mann-Whitney U-based pathway scores for all gene sets, and immediately truncate the dense rank matrix back to sparse format before moving to the next chunk. Partition the pathway dictionary (e.g. 1629 pathways from NCATS BioPlanet and 3CA signatures) across multiple CPU cores so scoring proceeds in parallel within each chunk. Finally, concatenate chunk results into the full cell-by-pathway PAS matrix (e.g. 108,497 cells x 1629 pathways) and write it out (e.g. pas_matrix.csv). The rationale is that ranking a chunk requires a dense intermediate, so chunking converts an intractable dense matrix into a bounded-size temporary while vectorized, multi-core computation preserves speed.

## Related tools

- **scPAFA** (Provides fast_ucell, the chunked, multi-core Python implementation of ranking-based pathway activity scoring) — github.com/ZhuoliHuang/scPAFA
- **UCell** (Reference R method whose PAS output fast_ucell reproduces; defines the rank-based statistic and max rank threshold)

## Examples

```
import scPAFA; pas = scPAFA.fast_ucell(expr_matrix, pathways, chunk_size=100000, r_max=2000); pas.to_csv('pas_matrix.csv')
```

## Evaluation signals

- Output PAS matrix has exactly the expected number of rows and columns (e.g. 108,497 cells x 1629 pathways)
- All PAS values fall within [0, 1]
- fast_ucell scores reproduce reference UCell PAS for the same cells and pathways (scPAFA reports fast_ucell yields the same PAS as UCell)
- Memory footprint stays bounded: the dense rank matrix exists only within each 100,000-cell chunk and is truncated back to sparse afterward
- Runtime scales acceptably — scPAFA claims PAS for 1383 pathways on million-cell data within 30 minutes, a >40-fold speedup over available tools

## Limitations

- Rank-based PAS (like AUCell) is insensitive to dataset composition, which is a property to be aware of when comparing across datasets
- Pseudobulk aggregation of the PAS matrix leads to information loss since PAS of multiple cells is aggregated
- Using a redundant pathway set increases result complexity, while too few or biologically misaligned pathways cause information loss

## Evidence

- [results] The fast_ucell function reimplements the R package UCell in Python, using vectorized computations with efficient chunking and a concurrent computation process that fully utilizes multi-core CPUs: [evidence span withheld — anchor status: not_found]
- [results] Process cells in chunks of 100,000 to bound memory, truncating the dense within-cell rank matrix back to sparse format after ranking each chunk: [evidence span withheld — anchor status: not_found]
- [discussion] The 'fast_ucell' function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [abstract] scPAFA can achieve over 40-fold reductions in the runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [readme] These functions are more computationally efficient implementation of UCell and AddModuleScore: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
