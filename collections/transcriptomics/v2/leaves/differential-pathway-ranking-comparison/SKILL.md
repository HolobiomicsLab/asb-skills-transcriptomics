---
name: differential-pathway-ranking-comparison
description: Use when comparing the computational efficiency (runtime, memory) and
  sensitivity of pathway activity scoring / differential pathway methods (scPAFA vs
  SCPA) on million-cell-scale scRNA-seq data with >1000 pathways, or when assessing
  whether downsampling-based tools lose disease-relevant pathway.
when_to_use_negative:
- The goal is multicellular pathway module discovery via factor analysis rather than
  per-cell-type differential pathway ranking — use scPAFA's MOFA-based steps 2–4 instead
- Either condition has fewer cells than the SCPA downsampling budget (e.g., <500 cells
  per condition), making the downsampling comparison uninformative
- Input is already a pseudobulk/aggregated sample-level matrix rather than a cell-level
  expression matrix
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPAFA
  role: Computes PAS via 'fast_Ucell' on 10 cores and provides differential pathway
    rankings; the speed- and memory-efficient method under benchmark
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: SCPA
  role: Comparator method run with compare_pathways at 500/1000/2000-cell downsampling
    budgets to measure runtime/memory tradeoff and sensitivity loss
- name: UCell
  role: Reference scoring method whose PAS output scPAFA's fast_Ucell is designed
    to reproduce exactly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:3bad3f1cc4b4fea2c7159aba7cedee6bca26c12a77a8490f58446f6573d60354
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - Python
  - SCPA
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# differential-pathway-ranking-comparison

## Summary

Benchmarks scPAFA against SCPA for pathway activity scoring (PAS) and per-cell-type differential pathway detection on large single-cell datasets (CRC 108,497-cell subset; lupus 664,214-cell subset), comparing wall-clock runtime and peak memory across SCPA downsampling budgets (500/1000/2000 cells per condition) and validating that interferon-related pathways are consistently ranked as top differential pathways.

## When to use

Use when comparing the computational efficiency (runtime, memory) and sensitivity of pathway activity scoring / differential pathway methods (scPAFA vs SCPA) on million-cell-scale scRNA-seq data with >1000 pathways, or when assessing whether downsampling-based tools lose disease-relevant pathway signals.

## When NOT to use

- The goal is multicellular pathway module discovery via factor analysis rather than per-cell-type differential pathway ranking — use scPAFA's MOFA-based steps 2–4 instead
- Either condition has fewer cells than the SCPA downsampling budget (e.g., <500 cells per condition), making the downsampling comparison uninformative
- Input is already a pseudobulk/aggregated sample-level matrix rather than a cell-level expression matrix

## Inputs

- Single-cell gene expression matrix for each dataset (e.g., CRC 108,497-cell subset; lupus 664,214-cell training subset from a >1.2M-cell SLE PBMC atlas)
- Pathway dictionary (Python dict: pathway name -> gene list; 1629 pathways for CRC, 1383 for lupus), sourced from NCATS BioPlanet or MSigDB
- Condition labels (e.g., healthy control vs CRC / SLE) and cell type annotations per cell

## Outputs

- Pathway activity score (PAS) matrix computed via scPAFA fast_Ucell
- benchmark_results.csv: runtime and peak memory for every method (scPAFA 10 cores, SCPA at 500/1000/2000-cell downsampling) × dataset (CRC, lupus) combination
- Per-cell-type ranked differential pathway lists (top-3 per method) with union-comparison across scPAFA and SCPA settings

## How to apply

Compute PAS for each dataset using scPAFA's 'fast_Ucell' on 10 cores, recording wall-clock runtime and peak memory; scPAFA's fast_Ucell reproduces UCell PAS and is a rank-based (AUCell-like) scoring scheme that needs no downsampling. Run SCPA's compare_pathways on the same subsets with downsampling budgets of 500, 1000, and 2000 cells per condition, recording runtime and memory per setting — SCPA's default of 500 cells per condition may lose information, and efficiency degrades as more cells are sampled. Assemble all method/dataset/downsampling combinations into a benchmark table (benchmark_results.csv) and quantify the speedup (reported as 1.3x on 500-cell lupus up to 103.9x on 2000-cell CRC). For sensitivity, extract per cell type the union of top-3 differential pathways from scPAFA and each SCPA setting, and check that interferon-related pathways (METAPROGRAM_17_INTERFERON_MHC_II_1 in CRC; Interferon alpha/beta signaling in lupus) are highlighted as in Fig. 7. Convergence of these interferon signals across methods, despite scPAFA avoiding downsampling, demonstrates that scPAFA matches SCPA's sensitivity without its computational cost.

## Related tools

- **scPAFA** (Computes PAS via 'fast_Ucell' on 10 cores and provides differential pathway rankings; the speed- and memory-efficient method under benchmark) — https://github.com/ZhuoliHuang/scPAFA
- **SCPA** (Comparator method run with compare_pathways at 500/1000/2000-cell downsampling budgets to measure runtime/memory tradeoff and sensitivity loss)
- **UCell** (Reference scoring method whose PAS output scPAFA's fast_Ucell is designed to reproduce exactly)

## Examples

```
conda activate scPAFA_env && pip install scPAFA  # then run tutorial/step1_generate_PAS_UCell.ipynb: compute PAS with fast_Ucell on 10 cores
```

## Evaluation signals

- benchmark_results.csv contains runtime and peak-memory entries for every method/dataset/downsampling combination (scPAFA 10 cores × SCPA 500/1000/2000 cells × CRC/lupus)
- Reported speedup falls within the published envelope: scPAFA 1.3x (500 cells, lupus) to 103.9x (2000 cells, CRC) faster than SCPA
- METAPROGRAM_17_INTERFERON_MHC_II_1 (CRC) and Interferon alpha/beta signaling (lupus) appear among top-3 differential pathways per cell type, as in Fig. 7
- fast_Ucell PAS matches the PAS of the original UCell implementation
- Throughput sanity check: scPAFA computes PAS for 1383 pathways on million-cell-level data within 30 min

## Limitations

- SCPA's downsampling strategy (default 500 cells per condition) may lose information on large datasets, and increasing the number of downsampled cells significantly affects efficiency — so SCPA results are budget-dependent
- fast_Ucell PAS is rank-based within each cell and is insensitive to the dataset in which cells are located, which may miss composition-dependent effects
- Pathway selection is a supervised process: a redundant pathway set increases result complexity, while too few or biologically misaligned pathways cause information loss

## Evidence

- [abstract] scPAFA can achieve over 40-fold reductions in the runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [intro] SCPA downsampling strategy and its cost: "it uses a downsampling strategy (selecting 500 cells per condition as default) that may lose information on large datasets"
- [intro] scPAFA throughput on million-cell data: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [discussion] fast_Ucell equivalence to UCell PAS: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [readme] fast_Ucell as efficient UCell implementation: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
