---
name: single-cell-pathway-activity-scoring
description: Use when you must compute PAS for a large pathway collection (>10^3 gene
  sets) across a large scRNA-seq dataset (e.g., 100k+ cells, million-cell atlases)
  and existing scorers like UCell, AUCell, or AddModuleScore are too slow or memory-hungry.
when_to_use_negative:
- When the goal is cross-condition differential pathway testing where SCPA's downsampling
  (500/1000/2000 cells) design is acceptable and PAS per cell is not needed — SCPA
  may suffice, though scPAFA was 1.3x–103.9x faster.
- When you already have a cell-by-pathway score matrix (e.g., from a prior fast_Ucell
  run) — re-running scoring wastes computation; proceed directly to pseudobulk processing
  and MOFA training.
- When the expression matrix already contains PAS columns or when pathway sets are
  highly redundant — pathway selection is a supervised process and a redundant set
  increases result complexity.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPAFA
  role: Primary library providing fast_Ucell and fast_score_genes functions that compute
    the PAS matrix with chunking and multi-core parallelization
  repo: github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Reference R implementation whose PAS fast_Ucell reproduces exactly, using
    rank-based Mann-Whitney U statistics with max rank threshold r_max=2000
- name: AUCell
  role: 'Reference method used for validation: fast_ucell PAS is closely related to
    AUCell PAS when the same max rank threshold is set (cosine similarity 0.97 CRC
    / 0.94 lupus)'
- name: AddModuleScore
  role: Alternative per-cell scoring method whose behavior fast_score_genes reimplements
    efficiently (known as 'score_genes' in Scanpy)
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:9c868f08b06ac6e36470d7681f1fc5543c30587dd0d0982a8dda1bbe20cb3cc6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - AddModuleScore
  - Python
  - SCPA
  - UCell
  - mofapy2
  - scPAFA
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-pathway-activity-scoring

## Summary

Computes cell-level pathway activity scores (PAS) for large-scale scRNA-seq data using scPAFA's fast_Ucell (a Python reimplementation of UCell) or fast_score_genes (AddModuleScore/Scanpy score_genes equivalent), converting a cell-gene expression matrix into a cell-pathway PAS matrix at million-cell scale.

## When to use

Use when you must compute PAS for a large pathway collection (>10^3 gene sets) across a large scRNA-seq dataset (e.g., 100k+ cells, million-cell atlases) and existing scorers like UCell, AUCell, or AddModuleScore are too slow or memory-hungry.

## When NOT to use

- When the goal is cross-condition differential pathway testing where SCPA's downsampling (500/1000/2000 cells) design is acceptable and PAS per cell is not needed — SCPA may suffice, though scPAFA was 1.3x–103.9x faster.
- When you already have a cell-by-pathway score matrix (e.g., from a prior fast_Ucell run) — re-running scoring wastes computation; proceed directly to pseudobulk processing and MOFA training.
- When the expression matrix already contains PAS columns or when pathway sets are highly redundant — pathway selection is a supervised process and a redundant set increases result complexity.

## Inputs

- Single-cell gene expression matrix (e.g., dense/sparse cell x gene matrix from an .h5ad-style scRNA-seq dataset; CRC tumor epithelial cells: 108,497 cells; lupus atlas training subset: 941,542 cells after filtering RBC-contaminated cluster 0)
- Pathway dictionary: Python dict mapping pathway name to list of genes, from MsigDB (JSON bundle), NCATS BioPlanet (pathway.csv), or custom collections (e.g., 1629 CRC pathways from NCATS BioPlanet + 3CA signatures; 1383 NCATS BioPlanet pathways for lupus)
- Cell-level metadata (sample/donor, cell type, technical batch) if downstream pseudobulk/MOFA use is planned

## Outputs

- Cell-by-pathway PAS matrix (e.g., 108,497 cells x 1629 pathways for CRC; written to pas_matrix.csv)
- PAS values bounded in [0, 1], consistent with UCell rank-based scoring

## How to apply

Build a Python pathway dictionary (pathway name -> gene list) from MsigDB (JSON bundle) or NCATS BioPlanet (pathway.csv), e.g., 1629 BioPlanet+3CA pathways for CRC or 1383 BioPlanet pathways for lupus. Load the single-cell gene expression matrix and run scPAFA's fast_Ucell, which computes per-cell Mann-Whitney U statistics on within-cell expression rankings with a maximum rank threshold of r_max=2000; alternatively run fast_score_genes for AddModuleScore-style scoring. Process cells in chunks (e.g., 100,000 cells per chunk) to bound memory, truncating the dense within-cell rank matrix back to sparse format after ranking each chunk, and partition pathways across multiple CPU cores (e.g., 10 cores) so pathway scoring runs in parallel within each chunk. Assemble chunk results into the full cell-by-pathway PAS matrix (e.g., 108,497 cells x 1629 pathways for CRC tumor epithelial cells) and write it out (e.g., pas_matrix.csv). Rationale: fast_Ucell yields the same PAS as UCell, and because scoring is rank-based within each cell it is insensitive to dataset composition; with the same max rank threshold it closely matches AUCell (mean per-cell cosine similarity 0.97 CRC / 0.94 lupus on 100k-cell subsets).

## Related tools

- **scPAFA** (Primary library providing fast_Ucell and fast_score_genes functions that compute the PAS matrix with chunking and multi-core parallelization) — github.com/ZhuoliHuang/scPAFA
- **UCell** (Reference R implementation whose PAS fast_Ucell reproduces exactly, using rank-based Mann-Whitney U statistics with max rank threshold r_max=2000)
- **AUCell** (Reference method used for validation: fast_ucell PAS is closely related to AUCell PAS when the same max rank threshold is set (cosine similarity 0.97 CRC / 0.94 lupus))
- **AddModuleScore** (Alternative per-cell scoring method whose behavior fast_score_genes reimplements efficiently (known as 'score_genes' in Scanpy))

## Examples

```
from scpafa import fast_ucell; pas = fast_ucell(expr_matrix, pathway_dict, max_rank=2000)  # per README tutorial/step1_generate_PAS_UCell.ipynb
```

## Evaluation signals

- Output PAS matrix has the expected shape, e.g., exactly 108,497 rows (cells) and 1629 columns (pathways) for the CRC tumor epithelial dataset.
- All PAS values fall within [0, 1].
- fast_ucell scores reproduce reference UCell PAS for the same cells and pathways (scPAFA reports fast_ucell yields the same PAS as UCell).
- Per-cell cosine similarity between fast_ucell and AUCell PAS vectors (same max rank threshold) averages ~0.97 on the CRC 100k-cell subset and ~0.94 on the lupus 100k-cell subset.
- Runtime check: PAS for 1383 pathways on million-cell-level scRNA-seq data completes within ~30 minutes (scPAFA reports >40-fold runtime reduction vs. existing tools).

## Limitations

- Rank-based PAS is insensitive to the dataset in which cells are located — desirable for comparability, but it means dataset-specific expression composition effects are not captured.
- Downstream pseudobulk aggregation of the PAS matrix loses information because PAS of multiple cells is aggregated.
- Pathway selection is a supervised process: a redundant pathway set increases result complexity, while too few or biologically misaligned pathways cause information loss.
- scPAFA's multicellular module identification (MOFA-based) requires at least 15 samples (not cells), limiting smaller studies.

## Evidence

- [intro] Existing single-cell pathway activity scoring methods, such as AddModuleScore7,8, UCell9, and AUCell10 can generate pathway activity scores (PAS) for individual cells: "Existing single-cell pathway activity scoring methods, such as AddModuleScore7,8, UCell9, and AUCell10 can generate pathway activity scores (PAS) for individual cells"
- [intro] which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [discussion] The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [readme] In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’: "In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
- [readme] These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy): "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
