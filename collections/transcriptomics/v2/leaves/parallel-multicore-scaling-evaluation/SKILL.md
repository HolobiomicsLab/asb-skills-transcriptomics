---
name: parallel-multicore-scaling-evaluation
description: Use when comparing the wall-clock runtime and memory footprint of a fast
  re-implementation versus original scoring tools on large single-cell datasets, or
  when testing scaling in cell number and pathway number.
when_to_use_negative:
- The goal is biological interpretation of pathway activity (multicellular module
  discovery), not computational performance — use the scPAFA PAS+MOFA workflow instead
- Datasets are small (far below 10k cells or <10^3 pathways) where runtime differences
  between implementations are negligible and benchmarking adds no value
- You need new PAS values for downstream analysis rather than a runtime/memory comparison
  of scoring methods
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: scPAFA
  role: Supplies the fast parallel implementations fast_ucell and fast_score_genes
    whose runtime and memory scaling is benchmarked
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Baseline PAS method (10 cores) compared against fast_ucell for runtime and
    peak memory
- name: AUCell
  role: Baseline PAS method (10 cores) compared against fast_ucell and fast_score_genes;
    fast_ucell gives closely related PAS by setting the same max rank threshold
- name: Scanpy score_genes / AddModuleScore
  role: Baseline PAS method (1 core) compared against fast_score_genes, which is an
    efficient re-implementation of AddModuleScore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:7b893ea6958df4617bf517555104d6c4eb8701ff2dc239e4246f85907659431d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - AUCell
  - AddModuleScore
  - Python
  - Scanpy
  - Scanpy score_genes / AddModuleScore
  - UCell
  - scPAFA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# parallel-multicore-scaling-evaluation

## Summary

Benchmark runtime and peak-memory scaling of fast parallel PAS implementations (scPAFA's fast_ucell and fast_score_genes) against their originals (UCell, AddModuleScore/score_genes) and AUCell across cell counts from 10k to 1.26M cells and pathway counts from 1383 to 14,300, using fixed core allocations.

## When to use

Use when comparing the wall-clock runtime and memory footprint of a fast re-implementation versus original scoring tools on large single-cell datasets, or when testing scaling in cell number and pathway number.

## When NOT to use

- The goal is biological interpretation of pathway activity (multicellular module discovery), not computational performance — use the scPAFA PAS+MOFA workflow instead
- Datasets are small (far below 10k cells or <10^3 pathways) where runtime differences between implementations are negligible and benchmarking adds no value
- You need new PAS values for downstream analysis rather than a runtime/memory comparison of scoring methods

## Inputs

- Single-cell gene expression matrix (CRC dataset and large-scale lupus atlas, up to 1,263,676 cells)
- Pathway dictionaries (Python dict: pathway name -> gene list) from MsigDB (C5, 14,300 pathways) and NCATS BioPlanet (1629 CRC / 1383 lupus pathways)
- Downsampled expression matrices at cell counts from 10,000 to 1,263,676
- Core-count settings per method (scPAFA: 10 or 20 cores; UCell/AUCell: 10 cores; score_genes: 1 core)

## Outputs

- Runtime and peak-memory measurements for each method x dataset x cell-count combination
- Aggregated benchmark table written to runtime_memory_benchmark.csv

## How to apply

Load the single-cell expression matrices (CRC with 1629 pathways, lupus with 1383 pathways built from MsigDB and NCATS BioPlanet) plus pathway dictionaries, then generate downsampled datasets spanning 10,000 to 1,263,676 cells. Compute PAS with scPAFA's fast_ucell and fast_score_genes using 10 cores, and with baselines UCell (10 cores), AUCell (10 cores), and Scanpy score_genes (1 core) on each downsampled dataset. To test pathway-number scaling, run fast_ucell and fast_score_genes with 20 cores on the expanded 14,300-pathway C5 (MsigDB) collection. Record elapsed runtime and peak memory for every method-dataset-cell-count combination and aggregate into one benchmark table (runtime_memory_benchmark.csv). Success is judged by the expected speedup profile: e.g., on the full lupus dataset fast_ucell took 27.05 min and fast_score_genes 29.9 min versus UCell 21.4 h, score_genes 9.3 h, and AUCell 5.1 h (~3.8–47.4x faster than originals, ~4.4–11.4x faster than AUCell), confirming the claimed ≥30-min completion for 1383 pathways on million-cell data.

## Related tools

- **scPAFA** (Supplies the fast parallel implementations fast_ucell and fast_score_genes whose runtime and memory scaling is benchmarked) — https://github.com/ZhuoliHuang/scPAFA
- **UCell** (Baseline PAS method (10 cores) compared against fast_ucell for runtime and peak memory)
- **AUCell** (Baseline PAS method (10 cores) compared against fast_ucell and fast_score_genes; fast_ucell gives closely related PAS by setting the same max rank threshold)
- **Scanpy score_genes / AddModuleScore** (Baseline PAS method (1 core) compared against fast_score_genes, which is an efficient re-implementation of AddModuleScore)

## Examples

```
conda create -n scPAFA_env python=3.10; conda activate scPAFA_env; pip install scPAFA
```

## Evaluation signals

- Full lupus run (1,263,676 cells, 1383 pathways): fast_ucell ≈ 27 min and fast_score_genes ≈ 30 min, versus UCell ≈ 21.4 h, score_genes ≈ 9.3 h, AUCell ≈ 5.1 h — speedup ratios ~3.8–47.4x over originals
- fast_ucell PAS values match original UCell PAS (and are closely related to AUCell when the same max rank threshold is set) — correctness in addition to speed
- Scaling test on 14,300-pathway C5 collection with 20 cores completes within the same order of magnitude, confirming pathway-number scaling
- Every method-dataset-cell-count combination has a recorded runtime and peak-memory value with no missing cells in the benchmark table
- Aggregated table written successfully to runtime_memory_benchmark.csv

## Limitations

- Baseline methods need substantially more time (UCell 21.4 h, score_genes 9.3 h, AUCell 5.1 h on the full lupus dataset), so end-to-end benchmarking at million-cell scale is expensive
- Core allocations differ between methods (scPAFA/UCell/AUCell 10 cores vs score_genes 1 core in the paper), so speedup comparisons are not purely per-core normalized
- Runtime/memory results depend on pathway set size — an expanded 14,300-pathway C5 collection changes scaling behavior and requires a separate measurement

## Evidence

- [abstract] Over-40x speedup claim: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [intro] Million-cell runtime target: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [methods] Fast functions used in the pipeline: [evidence span withheld — anchor status: not_found]
- [readme] Relationship of fast functions to originals: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
- [discussion] Equivalence check for correctness: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
