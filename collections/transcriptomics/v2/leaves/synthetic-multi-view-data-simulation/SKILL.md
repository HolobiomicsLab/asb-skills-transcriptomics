---
name: synthetic-multi-view-data-simulation
description: 'Use when benchmarking or validating JIVE/scJIVE decompositions against
  known ground truth: you need simulated multi-view matrices with controlled joint
  rank, individual ranks, and residual error to measure runtime differences or agreement
  of joint/individual/residual variance proportions.'
when_to_use_negative:
- Input is real experimental multi-omics or single-cell data where ground-truth joint/individual
  structure is unknown (use the actual decomposition pipeline instead, e.g., on TCGA
  BRCA_data)
- The goal is batch-effect correction of scRNA-seq data rather than runtime or numerical-equivalence
  benchmarking of the JIVE algorithm
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language for simulating the two-view matrices and timing repeated decomposition
    runs
- name: r.jive
  role: Baseline JIVE implementation benchmarked against scJIVE on the simulated pair
- name: JIVE (scJIVE)
  role: Enhanced, computationally efficient JIVE implementation whose runtime and
    variance proportions are compared to r.jive
  repo: github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:5f254d141675749973cda6e7092f90ac92c892fb1705ba67c98a4b08f597ec79
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JIVE (scJIVE)
  - R
  - r.jive
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# synthetic-multi-view-data-simulation

## Summary

Generate paired multi-view simulation data (e.g., two 200x1000 matrices sharing a rank-1 joint structure, each with a unique rank-1 individual structure, plus standard-normal residual error) to benchmark JIVE decomposition implementations. Synthetic ground-truth structure enables controlled runtime and variance-proportion comparisons between the original R.JIVE and the enhanced scJIVE.

## When to use

Use when benchmarking or validating JIVE/scJIVE decompositions against known ground truth: you need simulated multi-view matrices with controlled joint rank, individual ranks, and residual error to measure runtime differences or agreement of joint/individual/residual variance proportions.

## When NOT to use

- Input is real experimental multi-omics or single-cell data where ground-truth joint/individual structure is unknown (use the actual decomposition pipeline instead, e.g., on TCGA BRCA_data)
- The goal is batch-effect correction of scRNA-seq data rather than runtime or numerical-equivalence benchmarking of the JIVE algorithm

## Inputs

- Simulation parameters: matrix dimensions (200x1000 per view), number of views (2), joint rank (1), per-view individual rank (1), residual noise distribution (standard normal)
- Reference implementations to benchmark: r.jive::jive and scJIVE
- Number of repetitions for timing (e.g., 20)

## Outputs

- runtime_benchmark.csv: summary table of per-implementation mean runtimes over 20 runs and the runtime ratio
- variance_proportions.csv: comparison table of joint, individual, and residual proportion of variation explained under each implementation
- SimData: the simulated two-view matrix pair with known rank structure

## How to apply

Generate two 200x1000 matrices that share a common rank-1 joint structure, give each matrix a unique rank-1 individual structure, and add standard-normal residual error so the true decomposition is known a priori. Run the baseline implementation (r.jive::jive) and the enhanced implementation (scJIVE from github.com/oconnell-statistics-lab/scJIVE) on the identical simulated pair, repeating each run (e.g., 20 repetitions) and recording per-run wall-clock times. Summarize mean runtimes and their ratio, and compute the proportion of variation explained by joint, individual, and residual components under each implementation. Accept the benchmark only if the runtime ratio falls in the ~8-10x range (~35.8 s for r.jive vs ~4.1 s for scJIVE on this data size) and the variance proportions are near-identical between implementations.

## Related tools

- **R** (Language for simulating the two-view matrices and timing repeated decomposition runs)
- **r.jive** (Baseline JIVE implementation benchmarked against scJIVE on the simulated pair)
- **JIVE (scJIVE)** (Enhanced, computationally efficient JIVE implementation whose runtime and variance proportions are compared to r.jive) — github.com/oconnell-statistics-lab/scJIVE

## Examples

```
set.seed(1); X1 <- U %*% t(V) + A1 %*% t(B1) + matrix(rnorm(200*1000), 200, 1000); X2 <- U %*% t(V) + A2 %*% t(B2) + matrix(rnorm(200*1000), 200, 1000)
```

## Evaluation signals

- Mean runtime of r.jive::jive on the 200x1000 pair is ~35.8 s while scJIVE is ~4.1 s, giving a ratio in the ~8-10x range
- Joint, individual, and residual variance proportions from scJIVE are near-identical to those from r.jive on the same SimData
- Both implementations recover the planted rank-1 joint structure and rank-1 per-view individual structures rather than over-decomposing
- Per-run wall-clock times recorded across 20 repetitions show consistent (low-variance) timings for each implementation
- Output CSVs (runtime_benchmark.csv, variance_proportions.csv) contain one summary row per implementation with the required columns

## Limitations

- The original JIVE implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets, so the benchmark data size (200x1000) must stay modest for r.jive to finish
- The simulated ground truth (rank-1 joint, rank-1 individual) is simpler than real multi-omics structure such as the BRCA data where JIVE estimated a final joint rank of 2 and individual ranks of 27/26/25
- Rank-1/1 simulation does not exercise rank-selection steps (permutation-based or BIC-motivated), so agreement of estimated ranks on real data is not validated by this skill

## Evidence

- [other] simulated ground-truth structure: [evidence span withheld — anchor status: not_found]
- [other] benchmark protocol: [evidence span withheld — anchor status: not_found]
- [other] acceptance thresholds: [evidence span withheld — anchor status: not_found]
- [readme] motivation for the efficient implementation: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [abstract] original implementation infeasibility: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [methods] variance proportion reporting: "The most simple function is `showVarExplained`, which displays a barchart of the amount of variation explained"
