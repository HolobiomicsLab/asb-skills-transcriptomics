---
name: runtime-and-memory-profiling
description: Use when you need to quantify runtime and peak-memory costs of PAS computation
  (fast_ucell, fast_score_genes) vs UCell, AUCell, score_genes, or SCPA on datasets
  scaling from 10,000 up to 1,263,676 cells with 1383–14,300 pathways.
when_to_use_negative:
- The goal is biological interpretation (differential pathways, multicellular modules)
  rather than measuring computational efficiency — use scPAFA's PAS + MOFA workflow
  instead
- Input is not a single-cell gene expression matrix with a pathway dictionary; the
  benchmark targets PAS computation, not other omics tasks
- You need a strictly parallelism-controlled comparison — the benchmark mixes core
  counts (scPAFA/UCell/AUCell 10–20 cores vs score_genes 1 core), so speedup factors
  conflate implementation and parallelization
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: scPAFA
  role: Benchmarked implementation whose fast_ucell and fast_score_genes functions
    are timed and memory-profiled with 10/20 cores
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Baseline PAS method (10 cores) against which fast_ucell speedup is measured
- name: AUCell
  role: Baseline PAS method (10 cores) whose runtime is compared to fast_ucell
- name: Scanpy
  role: Provides score_genes (equivalent to AddModuleScore), run on 1 core as the
    slowest baseline
- name: SCPA
  role: Compared at 500/1000/2000-cell downsampling budgets for runtime and differential-pathway
    recall against scPAFA on full cells
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:e09d70db951745987de5a29ec9fb6333bdbdcd2e2ecedfa3578daa290cf46f2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - AUCell
  - AddModuleScore
  - Python
  - SCPA
  - Scanpy
  - UCell
  - scPAFA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# runtime-and-memory-profiling

## Summary

Benchmarks the elapsed runtime and peak memory of scPAFA's fast_ucell and fast_score_genes against UCell, AUCell, Scanpy score_genes and SCPA when computing pathway activity scores (PAS) on large scRNA-seq data with >10^3 pathways, producing a consolidated runtime/memory benchmark table.

## When to use

Use when you need to quantify runtime and peak-memory costs of PAS computation (fast_ucell, fast_score_genes) vs UCell, AUCell, score_genes, or SCPA on datasets scaling from 10,000 up to 1,263,676 cells with 1383–14,300 pathways.

## When NOT to use

- The goal is biological interpretation (differential pathways, multicellular modules) rather than measuring computational efficiency — use scPAFA's PAS + MOFA workflow instead
- Input is not a single-cell gene expression matrix with a pathway dictionary; the benchmark targets PAS computation, not other omics tasks
- You need a strictly parallelism-controlled comparison — the benchmark mixes core counts (scPAFA/UCell/AUCell 10–20 cores vs score_genes 1 core), so speedup factors conflate implementation and parallelization

## Inputs

- Single-cell gene expression matrices for CRC (108,497-cell subset) and lupus (664,214-cell training subset; 1,263,676-cell full atlas)
- Pathway dictionaries mapping pathway name to gene list: 1629 pathways (CRC), 1383 pathways (lupus), 14,300 pathways (MsigDB C5)
- Downsampled cell subsets spanning 10,000 to 1,263,676 cells
- SCPA downsampling budgets of 500, 1000, and 2000 cells per condition

## Outputs

- runtime_memory_benchmark.csv aggregating elapsed runtime and peak memory for every method/dataset/cell-count/pathway-count combination
- benchmark_results.csv for the scPAFA vs SCPA comparison at each downsampling level
- Speedup factors per method pair (e.g., fast_ucell 27.05 min vs UCell 21.4 h on the full lupus atlas)

## How to apply

Load the single-cell gene expression matrices together with the pathway dictionaries (1629 pathways for CRC from MsigDB/BioPlanet, 1383 for lupus), then generate downsampled versions spanning 10k to 1.26M cells. For each method–dataset–cell-count combination, run scPAFA's fast_ucell and fast_score_genes with 10 cores, UCell and AUCell with 10 cores, and Scanpy's score_genes with 1 core, recording elapsed runtime and peak memory each time. Run fast_ucell and fast_score_genes with 20 cores on the expanded 14,300-pathway C5 (MsigDB) collection to measure how runtime scales with pathway number. For the SCPA comparison, run SCPA compare_pathways at 500/1000/2000-cell downsampling budgets per condition and compare against scPAFA fast_Ucell on the full (non-downsampled) cells. Aggregate every measurement into a single table (e.g., runtime_memory_benchmark.csv) so speedup factors can be computed per method pair.

## Related tools

- **scPAFA** (Benchmarked implementation whose fast_ucell and fast_score_genes functions are timed and memory-profiled with 10/20 cores) — https://github.com/ZhuoliHuang/scPAFA
- **UCell** (Baseline PAS method (10 cores) against which fast_ucell speedup is measured)
- **AUCell** (Baseline PAS method (10 cores) whose runtime is compared to fast_ucell)
- **Scanpy** (Provides score_genes (equivalent to AddModuleScore), run on 1 core as the slowest baseline)
- **SCPA** (Compared at 500/1000/2000-cell downsampling budgets for runtime and differential-pathway recall against scPAFA on full cells)

## Examples

```
conda create -n scPAFA_env python=3.10; conda activate scPAFA_env; pip install scPAFA
```

## Evaluation signals

- The benchmark table contains a runtime and peak-memory entry for every method/dataset/cell-count combination (every combination populated, e.g., runtime_memory_benchmark.csv)
- Absolute anchors reproduce: fast_ucell 27.05 min and fast_score_genes 29.9 min on the full lupus atlas (1,263,676 cells, 1383 pathways) — i.e., PAS for 1383 pathways in million-cell-level data within ~30 min
- Speedup ratios fall in expected ranges: ~3.8–47.4x vs UCell/score_genes, ~4.4–11.4x vs AUCell, and 1.3x–103.9x vs SCPA depending on downsampling budget
- Pathway-scaling check: fast_ucell/fast_score_genes complete the 14,300-pathway C5 collection with 20 cores, confirming runtime scales with pathway number
- Correctness precondition: fast_ucell must return the same PAS as UCell before timing numbers are trusted

## Limitations

- Core counts differ across methods in the benchmark (scPAFA fast_ucell/fast_score_genes at 10 or 20 cores, UCell/AUCell at 10 cores, score_genes at 1 core), so comparisons partially reflect parallelization rather than algorithm alone
- fast_ucell PAS is ranking-based and insensitive to dataset composition — identical PAS must be verified against UCell before benchmark numbers are interpretable
- SCPA comparisons trade information for speed: SCPA downsamples (default 500 cells per condition), while scPAFA computes PAS for all cells, so the comparison is not a like-for-like cell coverage

## Evidence

- [intro] Claimed performance: scPAFA can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 minutes: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [abstract] Abstract claim of runtime reduction: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [readme] README describes the benchmarked functions: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
- [intro] SCPA downsampling trade-off motivating the runtime comparison: "it uses a downsampling strategy (selecting 500 cells per condition as default) that may lose information on large datasets"
