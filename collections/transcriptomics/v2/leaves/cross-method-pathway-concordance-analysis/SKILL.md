---
name: cross-method-pathway-concordance-analysis
description: Use when a large-scale scRNA-seq study requires validating that a fast
  pathway-activity method (scPAFA/fast_Ucell) detects the same disease-related differential
  pathways as a slower reference (SCPA), while quantifying compute cost.
when_to_use_negative:
- When the study has fewer than 15 samples (not cells) — scPAFA's MOFA-based multicellular
  module identification requires at least 15 samples, so the comparison context differs
- When the goal is cross-condition comparison per cell type only and compute is not
  a constraint — a full-information method like SCPA without downsampling may suffice
- When the pathway set is highly redundant or misaligned with the biological context
  — pathway selection is supervised, and a redundant set increases result complexity
  for both methods
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scPAFA
  role: Fast PAS computation via fast_Ucell (10 cores) on large subsets; benchmarked
    method
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: SCPA
  role: Reference differential-pathway method run with compare_pathways at 500/1000/2000-cell
    downsampling budgets
- name: UCell
  role: Underlying scoring method that fast_Ucell reproduces (PAS closely related
    to AUCell at the same max rank threshold)
- name: Python
  role: Implementation environment for scPAFA and benchmark table assembly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:84175090a6cbe0e2997b0f0800dc948f79d70b6d0267c4341eaf8827d154eea8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - Python
  - SCPA
  - UCell
  - scPAFA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-method-pathway-concordance-analysis

## Summary

Benchmarks scPAFA against alternative single-cell pathway differential testing methods (SCPA) on the same scRNA-seq subsets, recording runtime and memory at multiple downsampling budgets and checking that both methods surface the same disease-related differential pathways.

## When to use

Use when a large-scale scRNA-seq study requires validating that a fast pathway-activity method (scPAFA/fast_Ucell) detects the same disease-related differential pathways as a slower reference (SCPA), while quantifying compute cost.

## When NOT to use

- When the study has fewer than 15 samples (not cells) — scPAFA's MOFA-based multicellular module identification requires at least 15 samples, so the comparison context differs
- When the goal is cross-condition comparison per cell type only and compute is not a constraint — a full-information method like SCPA without downsampling may suffice
- When the pathway set is highly redundant or misaligned with the biological context — pathway selection is supervised, and a redundant set increases result complexity for both methods

## Inputs

- scRNA-seq expression matrix subset (e.g., CRC 108,497-cell subset; lupus 664,214-cell training subset)
- Pathway dictionary (Python dict of pathway name -> gene list, e.g., 1629 pathways for CRC; 1383 pathways for lupus; NCATS BioPlanet ~1658 pathways)
- Cell-type and condition metadata for per-cell-type differential testing
- SCPA tool with compare_pathways and configurable downsampling cell budgets
- scPAFA Python library (fast_Ucell / fast_score_genes) with multi-core setting (10 cores)

## Outputs

- benchmark_results.csv with runtime and peak-memory entries for every method/dataset/downsampling combination (scPAFA 10 cores vs SCPA at 500/1000/2000 cells)
- Per-cell-type ranked lists of differential pathways from scPAFA and from SCPA at each downsampling level
- Concordance check confirming interferon-related pathways (METAPROGRAM_17_INTERFERON_MHC_II_1 in CRC; Interferon alpha/beta signaling in lupus) among top differential pathways, as in Fig. 7

## How to apply

Load a matched expression-matrix subset (e.g., the CRC 108,497-cell subset with a 1629-pathway dictionary, or the lupus 664,214-cell training subset with a 1383-pathway dictionary) and compute PAS on 10 cores with scPAFA's fast_Ucell, recording wall-clock runtime and peak memory. Then run SCPA compare_pathways on the same subsets with downsampling budgets of 500, 1000, and 2000 cells per condition, recording runtime and memory for each setting. Assemble all method/dataset/downsampling combinations into a single benchmark table (benchmark_results.csv). For concordance, extract per cell type the union of the top-3 differential pathways from scPAFA and from each SCPA downsampling level, and verify that the interferon-related modules — METAPROGRAM_17_INTERFERON_MHC_II_1 in CRC and Interferon alpha/beta signaling in lupus — appear among the top differential pathways. The rationale is that SCPA's downsampling strategy (default 500 cells per condition) may lose information on large datasets, so scPAFA should be judged not only by speed (reported 1.3x–103.9x faster) but by whether it avoids that information loss while highlighting the same biology.

## Related tools

- **scPAFA** (Fast PAS computation via fast_Ucell (10 cores) on large subsets; benchmarked method) — https://github.com/ZhuoliHuang/scPAFA
- **SCPA** (Reference differential-pathway method run with compare_pathways at 500/1000/2000-cell downsampling budgets)
- **UCell** (Underlying scoring method that fast_Ucell reproduces (PAS closely related to AUCell at the same max rank threshold))
- **Python** (Implementation environment for scPAFA and benchmark table assembly)

## Examples

```
conda create -n scPAFA_env python=3.10; conda activate scPAFA_env; pip install scPAFA  # then run fast_Ucell PAS with n_cpus=10 vs SCPA compare_pathways at 500/1000/2000-cell budgets
```

## Evaluation signals

- benchmark_results.csv contains runtime and peak-memory entries for every method/dataset/downsampling combination (scPAFA and SCPA at 500, 1000, 2000 cells on both CRC and lupus subsets)
- scPAFA runtime with fast_Ucell on 10 cores is faster than SCPA at every downsampling level (expected ~1.3x at 500 cells on lupus up to ~103.9x at 2000 cells on CRC)
- METAPROGRAM_17_INTERFERON_MHC_II_1 appears among the top-3 differential pathways per cell type in the CRC comparison
- Interferon alpha/beta signaling appears among the top-3 differential pathways per cell type in the lupus comparison, matching Fig. 7
- SCPA results at larger downsampling budgets converge toward the full-data scPAFA ranking, indicating downsampling information loss has been accounted for

## Limitations

- SCPA's downsampling strategy (default 500 cells per condition) may lose information on large datasets, and increasing downsampled cells significantly reduces computational efficiency
- Pseudobulk integration of the PAS matrix leads to information loss since the PAS of multiple cells is aggregated
- Pathway selection is a supervised process; a redundant pathway set increases result complexity, while too few or biologically misaligned pathways may cause information loss
- fast_Ucell PAS, like AUCell, is ranking-based within each cell and thus insensitive to dataset composition, which may differ in sensitivity from methods using absolute expression

## Evidence

- [intro] it uses a downsampling strategy (selecting 500 cells per condition as default) that may lose information on large datasets, whereas increasing the number of downsampled cells significantly affects: "it uses a downsampling strategy (selecting 500 cells per condition as default) that may lose information on large datasets, whereas increasing the number of downsampled cells significantly affects"
- [discussion] The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [methods] compute PAS by ‘fast_Ucell’: "compute PAS by ‘fast_Ucell’"
- [abstract] scPAFA can achieve over 40-fold reductions in the runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [readme] In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’: "In step1, single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
