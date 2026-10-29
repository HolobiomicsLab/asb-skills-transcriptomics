---
name: cross-algorithm-pseudotime-comparison-table-construction
description: Use when you have pseudotime orderings from two or more trajectory-inference
  algorithms run on the same single-cell RNA-seq dataset and need a quantitative smoothness
  benchmark (correlated-gene counts and lag-1 autocorrelation of lineage markers)
  to compare them.
when_to_use_negative:
- The dataset contains only a single algorithm's pseudotime ordering — with no competing
  trajectories there is nothing to tabulate against.
- The question is about CellRouter's own trajectory reconstruction or regulatory-network
  analysis rather than benchmarking it against Monocle 2, DPT, Wishbone, or Waterfall
  smoothness.
- The input data lacks lineage-specific branch markers (e.g., Klf1/Car2/Cited4 or
  Mpo/Prtn3/Ctsg orthologs), since lag-1 autocorrelation of these markers is a core
  metric.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Reconstructs the multi-state trajectories being benchmarked, via its flow-network
    algorithm on the subpopulation-aware graph; also run in a variant using Paul et
    al. cluster labels as subpopulation annotations
  repo: https://github.com/edroaldo/cellrouter
- name: Monocle 2
  role: Competing trajectory algorithm run on the same cells to generate pseudotime
    for comparison
- name: Diffusion Pseudotime (DPT)
  role: Competing trajectory algorithm providing diffusion-map pseudotime ordering
    for the comparison
- name: Wishbone
  role: Competing trajectory algorithm whose pseudotime is included in the benchmark
    table
- name: Waterfall
  role: Competing linear trajectory algorithm whose pseudotime is included in the
    benchmark table
- name: R
  role: Environment for running all algorithms, computing per-gene correlation and
    lag-1 autocorrelation metrics, and writing algorithm_comparison.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:33940633feb6919b9164b5d770901873522eec4cb7d602f9dedeb3a32d20ee5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Diffusion Pseudotime (DPT)
  - Monocle 2
  - R
  - Waterfall
  - Wishbone
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Cross-algorithm pseudotime comparison table construction

## Summary

Benchmark CellRouter's subpopulation-aware trajectories against competing trajectory-inference algorithms (Monocle 2, DPT, Wishbone, Waterfall) on the same scRNA-seq dataset (Paul et al. GSE72857 myeloid progenitors) by computing, per algorithm, the number of genes significantly correlated with pseudotime and the lag-1 autocorrelation of branch markers, then assembling a single comparison table (algorithm_comparison.csv).

## When to use

Use when you have pseudotime orderings from two or more trajectory-inference algorithms run on the same single-cell RNA-seq dataset and need a quantitative smoothness benchmark (correlated-gene counts and lag-1 autocorrelation of lineage markers) to compare them.

## When NOT to use

- The dataset contains only a single algorithm's pseudotime ordering — with no competing trajectories there is nothing to tabulate against.
- The question is about CellRouter's own trajectory reconstruction or regulatory-network analysis rather than benchmarking it against Monocle 2, DPT, Wishbone, or Waterfall smoothness.
- The input data lacks lineage-specific branch markers (e.g., Klf1/Car2/Cited4 or Mpo/Prtn3/Ctsg orthologs), since lag-1 autocorrelation of these markers is a core metric.

## Inputs

- Preprocessed GSE72857 mouse myeloid progenitor scRNA-seq dataset (Paul et al., Cell 2015)
- Pseudotime orderings from CellRouter (flow-network on subpopulation-aware graph, CMP subpopulation 20 → erythroid subpopulation 13 and GMP subpopulation 9)
- Pseudotime orderings from Monocle 2, Diffusion Pseudotime (DPT), Wishbone, and Waterfall run on the same cells
- Lineage branch marker gene sets: Klf1, Car2, Cited4 (erythroid); Mpo, Prtn3, Ctsg (myeloid)

## Outputs

- algorithm_comparison.csv — one row per algorithm variant (six total, including CellRouter-with-Paul-labels) with significantly correlated gene counts and lag-1 autocorrelations

## How to apply

Run each trajectory algorithm on the identical preprocessed cell set: CellRouter's flow-network algorithm on its subpopulation-aware graph (e.g., from CMP subpopulation 20 to erythroid subpopulation 13 and GMP subpopulation 9, plus a CellRouter variant using the Paul et al. cluster labels directly as subpopulation annotations), and the competing algorithms Monocle 2, Diffusion Pseudotime (DPT), Wishbone, and Waterfall on the same cells. For each algorithm's pseudotime ordering, compute two smoothness metrics: (1) the number of genes whose expression is significantly correlated with pseudotime, and (2) the lag-1 autocorrelation of lineage branch markers — Klf1, Car2, Cited4 for the erythroid branch and Mpo, Prtn3, Ctsg for the myeloid branch. Assemble one row per algorithm variant (six rows: five algorithms plus CellRouter-with-Paul-labels) with columns for significantly correlated gene counts and lag-1 autocorrelations, and write to algorithm_comparison.csv. The rationale, per the article, is that CellRouter relaxes the requirement of identifying branching points and introduces subpopulation-awareness, making it more robust for reconstructing trajectories from early states towards intermediate or transitory cell states — a property the smoothness metrics are designed to expose.

## Related tools

- **CellRouter** (Reconstructs the multi-state trajectories being benchmarked, via its flow-network algorithm on the subpopulation-aware graph; also run in a variant using Paul et al. cluster labels as subpopulation annotations) — https://github.com/edroaldo/cellrouter
- **Monocle 2** (Competing trajectory algorithm run on the same cells to generate pseudotime for comparison)
- **Diffusion Pseudotime (DPT)** (Competing trajectory algorithm providing diffusion-map pseudotime ordering for the comparison)
- **Wishbone** (Competing trajectory algorithm whose pseudotime is included in the benchmark table)
- **Waterfall** (Competing linear trajectory algorithm whose pseudotime is included in the benchmark table)
- **R** (Environment for running all algorithms, computing per-gene correlation and lag-1 autocorrelation metrics, and writing algorithm_comparison.csv)

## Examples

```
Rscript run_trajectory_benchmark.R && verify: read.csv('algorithm_comparison.csv') has 6 rows with non-empty correlated_gene_counts and lag1_autocorrelation columns
```

## Evaluation signals

- algorithm_comparison.csv exists and contains exactly six algorithm-variant rows (CellRouter, CellRouter-with-Paul-labels, Monocle 2, DPT, Wishbone, Waterfall).
- Both metric columns — significantly correlated gene counts and lag-1 autocorrelations — are populated (non-empty, numeric) for every row.
- CellRouter rows show more significantly correlated genes and higher lag-1 autocorrelation of branch markers than the competing algorithms, reproducing the paper's reported pattern.
- All algorithms were run on the same preprocessed cell set, so differences in metrics reflect trajectory smoothness rather than differing cell populations.
- Per-algorithm pseudotime orderings were verified to cover the same start (CMP subpopulation 20) and target (erythroid subpopulation 13, GMP subpopulation 9) states before metric computation.

## Limitations

- Monocle 2 is not designed to reconstruct trajectories between any two chosen cell states, which might include transitions from or to rare cell types, limiting its comparability on arbitrary start/end pairs.
- Existing compared algorithms are limited by reconstructing linear trajectories (Waterfall, Monocle 1), identifying only a single branch point (Wishbone), or requiring a priori knowledge of the number of branches (DPT).
- Current single-cell trajectory algorithms are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states, so benchmark results may depend on which subpopulations are chosen as trajectory endpoints.
- CellRouter was tested in Unix-based systems or Mac and was not tested in Windows (per README).

## Evidence

- [intro] Monocle 2 comparability limit: "Monocle 2 addresses many of these challenges but is not designed to reconstruct trajectories between any two chosen cell states, which might include transitions from or to rare cell types"
- [intro] Existing algorithm limitations: "Limitations include reconstructing linear trajectories (Waterfall, Monocle 1), identifying only a single branch point (Wishbone), or requiring a priori knowledge of the number of branches (Diffusion"
- [intro] Robustness rationale: "they are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states"
- [methods] Multi-state trajectories between any subpopulations: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [readme] Dataset used in benchmark: "Analysis of mouse myeloid progenitor single-cell RNA-seq data generated by Paul et al.(Cell 2015)"
- [abstract] Flow-network method: "identifies complex cell-state transition trajectories by using flow networks to explore the subpopulation structure of multi-dimensional, single-cell omics data"
