---
name: rank-based-gene-signature-scoring
description: Use when you need rapid per-cell pathway activity scores on large-scale
  scRNA-seq data (e.g., >100k cells, >10^3 pathways) and existing tools like UCell,
  AUCell, or AddModuleScore are too slow.
when_to_use_negative:
- Input is already aggregated (pseudobulk or bulk expression) rather than per-cell
  scRNA-seq — PAS computation on single cells is the premise of this skill; pseudobulk
  integration happens later and loses information
- The downstream goal is disease-related multicellular pathway module discovery with
  fewer than 15 samples — this skill only produces the PAS matrix (Step 1); MOFA-based
  module identification requires ≥15 samples
- Expression data has already been scored — re-scoring an existing PAS matrix with
  fast_ucell is unnecessary
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPAFA
  role: Provides fast_ucell and fast_score_genes functions that convert the cell-gene
    expression matrix into a cell-pathway PAS matrix via vectorized, chunked, multi-core
    computation
  repo: github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Reference R method whose PAS fast_ucell reproduces exactly; same within-cell
    ranking scheme with max rank threshold
- name: AUCell
  role: Reference method whose PAS is closely related to fast_ucell PAS when the same
    max rank threshold is set; used as cross-validation target via per-cell cosine
    similarity
- name: scikit-learn
  role: Used to compute per-cell cosine similarity between fast_ucell and AUCell PAS
    vectors for validation
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:95737fc44719c1e1eebf149d8d9bd82ecc5feebb5f8a7e710bdb245ac124e5f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - Python
  - UCell
  - scPAFA
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rank-based-gene-signature-scoring

## Summary

Compute per-cell pathway activity scores (PAS) from a single-cell gene expression matrix using rank-based gene-set scoring, via scPAFA's fast_ucell (UCell reimplementation) or fast_score_genes (AddModuleScore/Scanpy score_genes reimplementation), designed for million-cell-scale scRNA-seq with >10^3 pathways.

## When to use

Use when you need rapid per-cell pathway activity scores on large-scale scRNA-seq data (e.g., >100k cells, >10^3 pathways) and existing tools like UCell, AUCell, or AddModuleScore are too slow.

## When NOT to use

- Input is already aggregated (pseudobulk or bulk expression) rather than per-cell scRNA-seq — PAS computation on single cells is the premise of this skill; pseudobulk integration happens later and loses information
- The downstream goal is disease-related multicellular pathway module discovery with fewer than 15 samples — this skill only produces the PAS matrix (Step 1); MOFA-based module identification requires ≥15 samples
- Expression data has already been scored — re-scoring an existing PAS matrix with fast_ucell is unnecessary

## Inputs

- Single-cell gene expression matrix (cell × gene) for the target dataset, e.g., CRC tumor epithelial cells (108,497 cells)
- Pathway dictionary: Python dict of pathway name -> gene list, e.g., 1629 pathways assembled from NCATS BioPlanet and 3CA signatures, or 1383 pathways from MsigDB/BioPlanet
- Max rank threshold parameter (r_max, e.g., 2000) controlling the fraction of top-ranked genes considered

## Outputs

- Cell × pathway PAS matrix (e.g., 108,497 × 1629) with values in [0, 1], written to pas_matrix.csv
- Optionally, per-cell cosine-similarity comparison table (pas_cosine_similarity_comparison.csv) when cross-validating against AUCell

## How to apply

First generate a pathway dictionary (Python dict mapping pathway name to a list of genes) from MsigDB, NCATS BioPlanet, or a custom collection using the generate_pathway_input.ipynb tutorial. Then run scPAFA's fast_ucell, which computes per-cell Mann-Whitney U statistics for each pathway gene set on within-cell expression rankings with a max rank threshold (r_max=2000 in the CRC workflow); cells are processed in chunks of 100,000 to bound memory, truncating the dense within-cell rank matrix back to sparse format after ranking each chunk, and the ~1629 pathways are partitioned across CPU cores in parallel. Alternatively use fast_score_genes for the AddModuleScore ('score_genes' in Scanpy) style of scoring. Assemble chunk results into a cells-by-pathways PAS matrix (e.g., 108,497 cells × 1629 pathways) and write it to pas_matrix.csv. Rationale: rank-based scoring is insensitive to dataset composition, and the same max rank threshold makes fast_ucell PAS closely related to AUCell PAS, while speed comes from vectorized chunked, multi-core computation rather than an approximation of the score.

## Related tools

- **scPAFA** (Provides fast_ucell and fast_score_genes functions that convert the cell-gene expression matrix into a cell-pathway PAS matrix via vectorized, chunked, multi-core computation) — github.com/ZhuoliHuang/scPAFA
- **UCell** (Reference R method whose PAS fast_ucell reproduces exactly; same within-cell ranking scheme with max rank threshold)
- **AUCell** (Reference method whose PAS is closely related to fast_ucell PAS when the same max rank threshold is set; used as cross-validation target via per-cell cosine similarity)
- **scikit-learn** (Used to compute per-cell cosine similarity between fast_ucell and AUCell PAS vectors for validation)

## Examples

```
jupyter nbconvert --to notebook --execute tutorial/step1_generate_PAS_UCell.ipynb  # after: conda create -n scPAFA_env python=3.10 && pip install scPAFA
```

## Evaluation signals

- Output PAS matrix has the expected dimensions, e.g., exactly 108,497 rows (cells) and 1629 columns (pathways) for the CRC tumor epithelial example
- All PAS values fall within [0, 1]
- fast_ucell scores reproduce reference UCell PAS for the same cells and pathways (scPAFA reports fast_ucell yields the same PAS as UCell)
- Average per-cell cosine similarity between fast_ucell and AUCell PAS vectors matches reported values of 0.97 (CRC) and 0.94 (lupus) within a small tolerance
- Runtime shows the reported speedup (over 40-fold reduction vs. available tools; 1383 pathways in million-cell-level data within 30 min)

## Limitations

- MOFA-based module identification downstream requires at least 15 samples (not cells), limiting smaller studies
- Pathway selection is a supervised process: a redundant pathway set increases result complexity, while too few or biologically misaligned pathways cause information loss
- For very large datasets, existing PAS tools are slow when >10^3 pathways are required — hence the need for this implementation, but memory must be bounded via chunking (100,000 cells per chunk in the CRC workflow)

## Evidence

- [readme] single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’: "In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
- [readme] These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy): "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
- [discussion] The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [abstract] scPAFA can achieve over 40-fold reductions in the runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [intro] which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
