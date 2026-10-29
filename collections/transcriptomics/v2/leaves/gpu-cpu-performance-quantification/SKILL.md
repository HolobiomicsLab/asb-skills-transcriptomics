---
name: gpu-cpu-performance-quantification
description: Use when benchmarking cudaMMC against 3D-GNOME timing on the same workstation
  and you have per-run execution times for ≥10 repeated runs per chromosome-method
  pair, and need a reproducible stability metric (CoV) rather than a single wall-clock
  time.
when_to_use_negative:
- You only have a single timing run per configuration — no variance can be estimated,
  so a CoV-based stability comparison is meaningless; report absolute run time instead.
- The goal is comparing structural quality of the generated chromatin models (e.g.
  enhancer–promoter distances), not execution-time stability.
- Timing runs were executed on different workstations or under differing GPU/CUDA
  configurations, so cudaMMC and 3D-GNOME timings are not comparable on the same hardware
  baseline.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: cudaMMC
  role: GPU-enhanced Multiscale Monte Carlo 3D chromatin modelling engine whose repeated
    run times are measured and whose CoV stability is quantified
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: CPU-based predecessor method used as the run-time stability baseline against
    which cudaMMC timings are compared
  repo: https://bitbucket.org/3dome/3dgnome
- name: pandas
  role: used with numpy to compute mean, standard deviation and CoV of the 10 execution
    times per chromosome-method pair
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:3cd2003fc7b4f47962f83d5e8677343fb60736cf213133c598d674c538df03d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - cudaMMC
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gpu-cpu-performance-quantification

## Summary

Quantifies and compares run-time stability of the GPU-accelerated cudaMMC engine against the CPU-based 3D-GNOME method by computing coefficients of variation (CoV = std/mean × 100) over repeated modelling runs per chromosome, as reported for chr1, chr14 and chr21 in the cudaMMC paper.

## When to use

Use when benchmarking cudaMMC against 3D-GNOME timing on the same workstation and you have per-run execution times for ≥10 repeated runs per chromosome-method pair, and need a reproducible stability metric (CoV) rather than a single wall-clock time.

## When NOT to use

- You only have a single timing run per configuration — no variance can be estimated, so a CoV-based stability comparison is meaningless; report absolute run time instead.
- The goal is comparing structural quality of the generated chromatin models (e.g. enhancer–promoter distances), not execution-time stability.
- Timing runs were executed on different workstations or under differing GPU/CUDA configurations, so cudaMMC and 3D-GNOME timings are not comparable on the same hardware baseline.

## Inputs

- Benchmark timing logs with execution times for 10 repeated cudaMMC modelling runs per chromosome (chr1, chr14, chr21)
- Benchmark timing logs with execution times for 10 repeated 3D-GNOME modelling runs per chromosome (chr1, chr14, chr21)
- Chromosome identifiers for the modelled regions (chr1, chr14, chr21)

## Outputs

- cov_execution_times.csv: 6-row table (chr1, chr14, chr21 × 3D-GNOME, cudaMMC) of coefficient-of-variation percentages
- Per chromosome-method mean and standard deviation of the 10 execution times

## How to apply

Load the per-run execution times recorded for the 10 repeated modelling runs of cudaMMC and 3D-GNOME on chr1, chr14 and chr21 from the benchmark timing logs. Using pandas/numpy, compute the mean and standard deviation of the 10 execution times for each chromosome-method pair, then derive CoV = (std / mean) × 100. Assemble the resulting 6-row table (2 methods × 3 chromosomes) into cov_execution_times.csv. Validate the computed CoVs against the reported stability results — cudaMMC 2.48% (chr1), 1.24% (chr14), 2.01% (chr21) versus 3D-GNOME 32.05% (chr1), 19.79% (chr14), 6.45% (chr21), corresponding to Supplementary Fig. S2 — within a small numeric tolerance. The rationale is that a GPU-enhanced Multiscale Monte Carlo implementation should show both faster and more consistent run times than the original 3D-GNOME when run on identical hardware.

## Related tools

- **cudaMMC** (GPU-enhanced Multiscale Monte Carlo 3D chromatin modelling engine whose repeated run times are measured and whose CoV stability is quantified) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (CPU-based predecessor method used as the run-time stability baseline against which cudaMMC timings are compared) — https://bitbucket.org/3dome/3dgnome
- **pandas** (used with numpy to compute mean, standard deviation and CoV of the 10 execution times per chromosome-method pair)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('timing_logs.csv'); print(df.groupby(['chromosome','method'])['time'].agg(lambda s: s.std()/s.mean()*100))"
```

## Evaluation signals

- Computed cudaMMC CoVs fall at ~2.48% (chr1), ~1.24% (chr14) and ~2.01% (chr21) within a small tolerance
- Computed 3D-GNOME CoVs fall at ~32.05% (chr1), ~19.79% (chr14) and ~6.45% (chr21) within a small tolerance
- cov_execution_times.csv contains exactly 6 rows (3 chromosomes × 2 methods) with a numeric CoV column in percent
- Every CoV is derived from exactly 10 execution-time observations (n = 10 per chromosome-method pair)
- Direction check: for every chromosome, cudaMMC CoV is an order of magnitude lower than 3D-GNOME CoV, consistent with 'faster performance with better stability'

## Limitations

- CoV stability reflects run-time consistency on a single workstation and specific NVIDIA GPU architecture (compute capability >= 6.0, CUDA Toolkit 11.0); results do not transfer to other hardware without re-benchmarking.
- Only chromosomes chr1, chr14 and chr21 were benchmarked in the reported comparison; other chromosomes or regions (e.g. region strings like chr14:1:2500000) may behave differently.
- Reported values correspond to the published Supplementary Fig. S2 comparison; re-derived CoVs can only match within a tolerance, not exactly, since the underlying raw timing logs depend on the original run environment.

## Evidence

- [abstract] cudaMMC demonstrates better run-time stability than 3D-GNOME: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [intro] cudaMMC is a GPU-enhanced version of 3D-GNOME: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [readme] cudaMMC is run from the command line per chromosome for benchmarking: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] Workstation/hardware requirements constrain the benchmark environment: "NVIDIA CUDA Toolkit 11.0 or higher with compatible NVIDIA driver"
