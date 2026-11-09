---
name: coefficient-of-variation-computation
description: Use when you have repeated (e.g., 10×) timing measurements per method
  per chromosome and need a dimensionless stability metric to compare run-time consistency
  across methods or configurations.
when_to_use_negative:
- Only a single run time per method exists — std is undefined/meaningless and no stability
  assessment is possible
- The goal is absolute speed comparison (total computation time) rather than run-time
  consistency; use a timing/performance benchmark instead
- Input measurements are quality metrics (e.g., structure similarity) rather than
  repeated run times of the same workload
edam_operation: http://edamontology.org/operation_2239
tools:
- name: cudaMMC
  role: GPU-enhanced Multiscale Monte Carlo method whose repeated run times are measured
    and whose CoV is compared against 3D-GNOME
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: Reference method whose per-run execution times are benchmarked to obtain the
    baseline CoV stability values
  repo: https://bitbucket.org/3dome/3dgnome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:47a4b7a1cf8e021110316ab9b1e97b9a7d3d8fc99526006df358c3ea36b3a119
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - cudaMMC
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# coefficient-of-variation-computation

## Summary

Computes the coefficient of variation (CoV = std/mean × 100) of per-run execution times to quantify run-to-run stability of a computational method, as used in the cudaMMC benchmark to show its GPU Monte Carlo runs are far more stable than 3D-GNOME.

## When to use

Use when you have repeated (e.g., 10×) timing measurements per method per chromosome and need a dimensionless stability metric to compare run-time consistency across methods or configurations.

## When NOT to use

- Only a single run time per method exists — std is undefined/meaningless and no stability assessment is possible
- The goal is absolute speed comparison (total computation time) rather than run-time consistency; use a timing/performance benchmark instead
- Input measurements are quality metrics (e.g., structure similarity) rather than repeated run times of the same workload

## Inputs

- Benchmark timing log with per-run execution times for each repeated modelling run (e.g., 10 runs) per method per chromosome
- Chromosome/method labels (chr1, chr14, chr21 × cudaMMC, 3D-GNOME) for grouping

## Outputs

- cov_execution_times.csv: 6-row table of CoV (%) per chromosome–method pair
- Mean and standard deviation of execution times per pair (intermediate values)

## How to apply

Load the per-run execution times recorded for the repeated modelling runs of each method (e.g., cudaMMC and 3D-GNOME on chr1, chr14, chr21) from the benchmark timing logs into pandas. For each chromosome–method pair compute the mean and standard deviation of the run times with pandas/numpy, then form CoV = (std / mean) × 100. Assemble one row per chromosome–method pair (6 rows for two methods × three chromosomes) and write it to a CSV such as cov_execution_times.csv. Validate by checking the computed CoV against published benchmark values within a small tolerance (cudaMMC: 2.48%/1.24%/2.01% for chr1/chr14/chr21; 3D-GNOME: 32.05%/19.79%/6.45%). The rationale is that a low CoV indicates deterministic-like GPU-accelerated behaviour, supporting the article's claim that cudaMMC has better stability than the CPU-based 3D-GNOME on the same workstation.

## Related tools

- **cudaMMC** (GPU-enhanced Multiscale Monte Carlo method whose repeated run times are measured and whose CoV is compared against 3D-GNOME) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (Reference method whose per-run execution times are benchmarked to obtain the baseline CoV stability values) — https://bitbucket.org/3dome/3dgnome

## Examples

```
import pandas as pd; df = pd.read_csv('timing_logs.csv'); cov = df.groupby(['method','chromosome'])['time'].agg(lambda x: x.std()/x.mean()*100); cov.to_csv('cov_execution_times.csv')
```

## Evaluation signals

- Computed CoV for cudaMMC equals 2.48% (chr1), 1.24% (chr14), 2.01% (chr21) within a small tolerance
- Computed CoV for 3D-GNOME equals 32.05% (chr1), 19.79% (chr14), 6.45% (chr21) within a small tolerance
- Output table contains exactly 6 rows (3 chromosomes × 2 methods) and CoV values are non-negative percentages
- Cross-check: mean and std per pair recomputed independently (numpy) match the pandas-derived values used in the CoV formula

## Limitations

- CoV is scale-dependent on the mean run time — methods with systematically different absolute run times (cudaMMC is far faster) can still be compared because CoV is dimensionless, but very small means can inflate the metric
- Requires the same number of repeated runs per chromosome–method pair (10 runs in the benchmark) for a fair comparison
- Stability conclusions are workstation-specific: the article claims better stability 'on the same workstation', so values may not transfer across hardware

## Evidence

- [abstract] cudaMMC stability claim: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [intro] cudaMMC relationship to 3D-GNOME: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [supplementary] reported CoV benchmark values: [evidence span withheld — anchor status: not_found]
