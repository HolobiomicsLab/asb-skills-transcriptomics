---
name: downsampling-based-method-benchmarking
description: Use when evaluating whether a tool that downsamples cells per condition
  (e.g. SCPA's default 500 cells) loses information or gains speed relative to a full-dataset
  method on million-cell-scale scRNA-seq data with >1000 pathways.
when_to_use_negative:
- 'Datasets with fewer than ~15 samples: scPAFA''s MOFA-based multicellular module
  identification requires at least 15 samples (not cells), so the module-agreement
  validation step is not applicable to small studies'
- Analyses where the target method is not downsampling-based and consumes all cells
  anyway — the 500/1000/2000-cell budget sweep only characterizes information loss
  from downsampling strategies
- Pathway sets that are redundant or biologically misaligned with the context — the
  article notes redundant pathway sets increase result complexity and misaligned sets
  cause information loss, confounding the differential-pathway comparison
tools:
- name: scPAFA
  role: 'Full-data baseline: fast_Ucell function computes UCell-equivalent PAS for
    all cells on 10 cores and records runtime/peak memory'
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: SCPA
  role: 'Downsampling-based comparator: compare_pathways run at 500/1000/2000 cells
    per condition, with runtime and peak memory recorded per setting'
- name: UCell
  role: Reference scoring method whose PAS fast_Ucell reproduces; AUCell-related when
    the same max rank threshold is set
- name: Python
  role: Implementation language for scPAFA fast_Ucell and the benchmark table assembly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:3858dc0c282f97dca13cd652d1858b1667cce75b86a027c7018f9cf699660ae1
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# downsampling-based-method-benchmarking

## Summary

Benchmarking protocol for comparing a full-data method (scPAFA's fast_Ucell PAS computation) against a downsampling-based competitor (SCPA) across 500/1000/2000-cell budgets per condition, measuring wall-clock runtime, peak memory, and differential-pathway agreement on CRC and lupus scRNA-seq subsets.

## When to use

Use when evaluating whether a tool that downsamples cells per condition (e.g. SCPA's default 500 cells) loses information or gains speed relative to a full-dataset method on million-cell-scale scRNA-seq data with >1000 pathways.

## When NOT to use

- Datasets with fewer than ~15 samples: scPAFA's MOFA-based multicellular module identification requires at least 15 samples (not cells), so the module-agreement validation step is not applicable to small studies
- Analyses where the target method is not downsampling-based and consumes all cells anyway — the 500/1000/2000-cell budget sweep only characterizes information loss from downsampling strategies
- Pathway sets that are redundant or biologically misaligned with the context — the article notes redundant pathway sets increase result complexity and misaligned sets cause information loss, confounding the differential-pathway comparison

## Inputs

- CRC scRNA-seq expression matrix (108,497-cell subset) with a 1629-pathway dictionary
- Lupus PBMC atlas training subset (664,214 cells) with a 1383-pathway dictionary
- Pathway dictionary as Python dict {pathway_name: [gene list]}, e.g. from MsigDB or NCATS BioPlanet
- Cell-type annotations and condition labels (disease vs healthy) for per-cell-type differential comparison

## Outputs

- benchmark_results.csv with runtime and peak-memory entries for every method/dataset/downsampling combination (scPAFA 10 cores vs SCPA at 500/1000/2000 cells)
- Per-cell-type union of top-3 differential pathways from scPAFA and each SCPA downsampling setting
- Speed/memory ratios (e.g. scPAFA 1.3x–103.9x faster than SCPA) and qualitative agreement on interferon-related pathways

## How to apply

Run the full-data baseline first: compute pathway activity scores (PAS) with scPAFA's fast_Ucell on 10 cores for the CRC 108,497-cell subset with its 1629-pathway dictionary and the lupus 664,214-cell training subset with its 1383-pathway dictionary, recording wall-clock runtime and peak memory. Then run SCPA compare_pathways on both subsets with downsampling budgets of 500, 1000, and 2000 cells per condition, recording runtime and peak memory for each level. Assemble all method/dataset/downsampling combinations into a single benchmark table (benchmark_results.csv). Finally, for each cell type extract the union of the top-3 differential pathways from scPAFA and from each SCPA downsampling setting, and check that disease-relevant pathways (METAPROGRAM_17_INTERFERON_MHC_II_1 in CRC; Interferon alpha/beta signaling in lupus) are consistently highlighted. The rationale is that SCPA's default of 500 cells per condition may lose information on large datasets, whereas scPAFA computes PAS for all cells, so agreement on top differential pathways plus the speed ratio (reported as 1.3x to 103.9x in scPAFA's favor) quantifies the trade-off.

## Related tools

- **scPAFA** (Full-data baseline: fast_Ucell function computes UCell-equivalent PAS for all cells on 10 cores and records runtime/peak memory) — https://github.com/ZhuoliHuang/scPAFA
- **SCPA** (Downsampling-based comparator: compare_pathways run at 500/1000/2000 cells per condition, with runtime and peak memory recorded per setting)
- **UCell** (Reference scoring method whose PAS fast_Ucell reproduces; AUCell-related when the same max rank threshold is set)
- **Python** (Implementation language for scPAFA fast_Ucell and the benchmark table assembly)

## Examples

```
conda create -n scPAFA_env python=3.10 && conda activate scPAFA_env && pip install scPAFA
```

## Evaluation signals

- benchmark_results.csv contains runtime and peak-memory entries for every method/dataset/downsampling combination (scPAFA and SCPA x {CRC, lupus} x {500, 1000, 2000})
- Speed ratio falls within the reported range: scPAFA 1.3x (500 cells, lupus) to 103.9x (2000 cells, CRC) faster than SCPA
- METAPROGRAM_17_INTERFERON_MHC_II_1 appears among top differential pathways per CRC cell type, and Interferon alpha/beta signaling among lupus cell types, as in Fig. 7
- Top-3 differential pathway unions from scPAFA and SCPA settings show the interferon-related pathways are not lost by downsampling-level choice
- scPAFA fast_Ucell PAS matches UCell PAS exactly (fast_Ucell is described as providing the same PAS of UCell)

## Limitations

- SCPA's downsampling strategy (default 500 cells per condition) may lose information on large datasets, and increasing the number of downsampled cells significantly affects computational efficiency
- scPAFA requires studies to include at least 15 samples (not cells) for MOFA-based multicellular pathway module identification
- Pseudobulk integration of the PAS matrix leads to information loss because PAS of multiple cells is aggregated

## Evidence

- [abstract] scPAFA runtime advantage: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [intro] SCPA downsampling trade-off: "it uses a downsampling strategy (selecting 500 cells per condition as default) that may lose information on large datasets"
- [readme] fast_Ucell input/output contract: "single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
- [discussion] fast_Ucell equals UCell: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [intro] million-cell-scale feasibility: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
