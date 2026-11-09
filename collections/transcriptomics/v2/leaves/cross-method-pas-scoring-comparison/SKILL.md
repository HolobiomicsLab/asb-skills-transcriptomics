---
name: cross-method-pas-scoring-comparison
description: Use when PAS must be computed on large-scale scRNA-seq data (>10^3 pathways,
  up to million-cell atlases) and an original vs. accelerated implementation comparison
  is needed.
when_to_use_negative:
- The PAS matrix is already computed and the task is downstream multicellular pathway
  module identification via MOFA — use the pseudobulk/MOFA workflow instead
- Datasets are small or pathway counts are far below 10^3, where scoring speed is
  not the bottleneck and the original UCell/score_genes implementations suffice
- The comparison requires methods sensitive to dataset composition (UCell/score_genes-like
  mean-expression scoring) to be equated with ranking-based AUCell-like scores without
  matching the max rank threshold
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scPAFA
  role: Accelerated PAS computation under test (fast_ucell, fast_score_genes)
  repo: github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Original scoring baseline benchmarked against fast_ucell
- name: AUCell
  role: Independent baseline (ranking-based PAS) benchmarked for runtime/memory
- name: Scanpy score_genes
  role: Original scoring baseline (AddModuleScore-equivalent) benchmarked against
    fast_score_genes
- name: AddModuleScore
  role: Seurat-style scoring method that fast_score_genes efficiently reimplements
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:990bdd9a510ae21ea4d2473ba428dd019b72a6970e47200cfb52986eb328069e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - AddModuleScore
  - Python
  - Scanpy
  - Scanpy score_genes
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# cross-method-pas-scoring-comparison

## Summary

Benchmarks scPAFA's fast_ucell and fast_score_genes against UCell, AUCell, and Scanpy score_genes for pathway activity score (PAS) computation, recording runtime and peak memory from 10,000 up to 1,263,676 cells and up to 14,300 pathways.

## When to use

Use when PAS must be computed on large-scale scRNA-seq data (>10^3 pathways, up to million-cell atlases) and an original vs. accelerated implementation comparison is needed.

## When NOT to use

- The PAS matrix is already computed and the task is downstream multicellular pathway module identification via MOFA — use the pseudobulk/MOFA workflow instead
- Datasets are small or pathway counts are far below 10^3, where scoring speed is not the bottleneck and the original UCell/score_genes implementations suffice
- The comparison requires methods sensitive to dataset composition (UCell/score_genes-like mean-expression scoring) to be equated with ranking-based AUCell-like scores without matching the max rank threshold

## Inputs

- Single-cell gene expression matrix (e.g. CRC datasets and the >1.2M-cell lupus PBMC atlas)
- Pathway dictionary (Python dict: pathway name -> gene list) from MsigDB or NCATS BioPlanet, e.g. 1383 lupus pathways or 14,300 C5 MsigDB pathways
- Downsampled cell-count series spanning 10,000 to 1,263,676 cells
- Core-count configuration (10 cores for PAS methods, 20 cores for pathway-scaling runs)

## Outputs

- runtime_memory_benchmark.csv aggregating elapsed runtime and peak memory for every method-dataset-cell-count combination
- Pathway activity score (PAS) matrix per method for each dataset/cell-count condition

## How to apply

Load the single-cell gene expression matrices for the CRC and lupus datasets together with pathway dictionaries built from MsigDB and NCATS BioPlanet (1629 pathways for CRC, 1383 for lupus). Generate downsampled versions spanning 10k to 1.26M cells, then compute PAS with scPAFA's fast_ucell and fast_score_genes using 10 cores per downsampled dataset. Run baselines under matched data: UCell (10 cores), AUCell (10 cores), and Scanpy score_genes (1 core). To measure scaling with pathway number, rerun fast_ucell and fast_score_genes on the expanded 14,300-pathway C5 (MsigDB) collection with 20 cores. Record elapsed runtime and peak memory for every method-dataset-cell-count combination and aggregate into a single runtime_memory_benchmark.csv. Correctness hinges on methodological equivalence: fast_ucell must yield the same PAS as UCell when the same max rank threshold is set, and fast_score_genes must reproduce AddModuleScore/score_genes scores.

## Related tools

- **scPAFA** (Accelerated PAS computation under test (fast_ucell, fast_score_genes)) — github.com/ZhuoliHuang/scPAFA
- **UCell** (Original scoring baseline benchmarked against fast_ucell)
- **AUCell** (Independent baseline (ranking-based PAS) benchmarked for runtime/memory)
- **Scanpy score_genes** (Original scoring baseline (AddModuleScore-equivalent) benchmarked against fast_score_genes)
- **AddModuleScore** (Seurat-style scoring method that fast_score_genes efficiently reimplements)

## Examples

```
conda activate scPAFA_env; pip install scPAFA
```

## Evaluation signals

- Equivalence check: fast_ucell PAS must equal UCell PAS when the same max rank threshold is set (fast_ucell is described as providing 'the same PAS of UCell')
- Expected runtime on the full lupus atlas (1,263,676 cells, 1383 pathways): fast_ucell ~27.05 min and fast_score_genes ~29.9 min, vs UCell 21.4 h, score_genes 9.3 h, AUCell 5.1 h
- Benchmark table must contain one row per method-dataset-cell-count combination with both elapsed runtime and peak memory before aggregation to runtime_memory_benchmark.csv
- Scaling check: fast_ucell/fast_score_genes should complete the 14,300-pathway C5 MsigDB collection within the ~30-minute envelope claimed for 1383 pathways on million-cell data
- Reported speedups should fall in the ~3.8–47.4x range vs original implementations and ~4.4–11.4x vs AUCell

## Limitations

- Baseline core counts differ (UCell/AUCell 10 cores, score_genes 1 core), so speedup ratios partly reflect parallelization rather than algorithm alone
- fast_ucell is ranking-based like AUCell and is insensitive to dataset composition, whereas UCell/score_genes-style scores are not — cross-method PAS values are not interchangeable without matched parameters
- The original implementations become tardy and computationally ineffective at >10^3 pathways on large scRNA-seq data, which is precisely the regime this skill targets

## Evidence

- [intro] Claimed throughput of fast implementations: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [discussion] Methodological equivalence of fast_ucell and UCell: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [readme] Efficient reimplementation of baseline scoring methods: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
- [methods] The two PAS computation filter functions benchmarked: [evidence span withheld — anchor status: not_found]
- [abstract] Overall speedup claim motivating the comparison: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
