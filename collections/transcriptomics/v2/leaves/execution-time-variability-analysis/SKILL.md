---
name: execution-time-variability-analysis
description: Use when comparing the consistency (not just the speed) of execution
  times between two methods or implementations across repeated benchmark runs, e.g.
  GPU-accelerated vs CPU-based pipelines on the same workstation.
when_to_use_negative:
- The goal is to compare mean/absolute runtime speed of the methods rather than run-to-run
  consistency — a simple mean execution-time comparison suffices and CoV adds no information.
- Only a single execution per condition is available, making the standard deviation
  (and hence CoV) undefined or unstable.
- The inputs are modelling outputs (3D structures) or biological results rather than
  benchmark timing logs — timing variability analysis does not apply.
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0102
tools:
- name: cudaMMC
  role: GPU-enhanced Multiscale Monte Carlo tool whose 10 repeated per-chromosome
    execution times are measured as the stable comparator; run as ./build/cudaMMC
    -s config.ini -c <chr> -o <outdir> -n <label>
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: Baseline CPU-based predecessor whose 10 repeated per-chromosome execution
    times are measured as the less-stable comparator in the CoV benchmark
  repo: https://bitbucket.org/3dome/3dgnome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:d3789bf7a34c2a14ec0f923e22f2cb6f1d5f5b1d1a96b33f1867b942d3d6771e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - cudaMMC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# execution-time-variability-analysis

## Summary

Quantifies run-time stability of cudaMMC versus 3D-GNOME by computing the coefficient of variation (CoV = std/mean × 100) of execution times over 10 repeated modelling runs per chromosome (chr1, chr14, chr21) from benchmark timing logs.

## When to use

Use when comparing the consistency (not just the speed) of execution times between two methods or implementations across repeated benchmark runs, e.g. GPU-accelerated vs CPU-based pipelines on the same workstation.

## When NOT to use

- The goal is to compare mean/absolute runtime speed of the methods rather than run-to-run consistency — a simple mean execution-time comparison suffices and CoV adds no information.
- Only a single execution per condition is available, making the standard deviation (and hence CoV) undefined or unstable.
- The inputs are modelling outputs (3D structures) or biological results rather than benchmark timing logs — timing variability analysis does not apply.

## Inputs

- Benchmark timing logs containing per-run execution times for 10 repeated cudaMMC modelling runs on chr1, chr14, chr21
- Benchmark timing logs containing per-run execution times for 10 repeated 3D-GNOME modelling runs on chr1, chr14, chr21
- GM12878 CTCF ChIA-PET input data (anchors, PET clusters, singletons) used by both methods for each chromosome

## Outputs

- cov_execution_times.csv: 6-row table of chromosome-method pairs with mean, standard deviation, and CoV (%) of execution times
- Validation report comparing computed CoV values to the published stability results (cudaMMC 2.48%/1.24%/2.01%; 3D-GNOME 32.05%/19.79%/6.45%)

## How to apply

Load the per-run execution times recorded for the 10 repeated modelling runs of cudaMMC and 3D-GNOME on chr1, chr14 and chr21 from the benchmark timing logs. For each chromosome-method pair, compute the mean and standard deviation of the 10 execution times using pandas/numpy, then compute the coefficient of variation as CoV = (std / mean) × 100. Assemble the resulting 6-row table (chr1, chr14, chr21 for each of 3D-GNOME and cudaMMC) into cov_execution_times.csv. Validate the computed values against the reported benchmark stability results — cudaMMC: 2.48% (chr1), 1.24% (chr14), 2.01% (chr21); 3D-GNOME: 32.05% (chr1), 19.79% (chr14), 6.45% (chr21), as visualized in Supplementary Fig. S2 — within a small tolerance. The rationale is that run-to-run variance arising from stochastic Monte Carlo steps and GPU scheduling is as operationally important as mean speed when judging reproducibility of chromatin 3D structure reconstruction.

## Related tools

- **cudaMMC** (GPU-enhanced Multiscale Monte Carlo tool whose 10 repeated per-chromosome execution times are measured as the stable comparator; run as ./build/cudaMMC -s config.ini -c <chr> -o <outdir> -n <label>) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (Baseline CPU-based predecessor whose 10 repeated per-chromosome execution times are measured as the less-stable comparator in the CoV benchmark) — https://bitbucket.org/3dome/3dgnome

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('times.csv'); cov=df.groupby(['chromosome','method'])['time'].agg(lambda x: x.std()/x.mean()*100); cov.to_csv('cov_execution_times.csv'); print(cov)"
```

## Evaluation signals

- Computed CoV for cudaMMC matches 2.48% (chr1), 1.24% (chr14), and 2.01% (chr21) within a small tolerance
- Computed CoV for 3D-GNOME matches 32.05% (chr1), 19.79% (chr14), and 6.45% (chr21) within a small tolerance
- cov_execution_times.csv contains exactly 6 rows: chr1, chr14, chr21 for each of 3D-GNOME and cudaMMC
- Each chromosome-method CoV is derived from exactly 10 repeated run times (n=10 per cell of the table)
- All cudaMMC CoV values fall well below all 3D-GNOME CoV values, reproducing the reported 'better stability' ordering

## Limitations

- The published CoV figures are specific to the authors' workstation and hardware (CUDA-capable NVIDIA GPU, compute capability >= 6.0); absolute times and variability will differ on other machines.
- Both tools use stochastic Monte Carlo sampling, so per-run times include stochastic and GPU-scheduling variance that CoV captures but does not attribute.
- The comparison covers only chr1, chr14 and chr21 and 10 repeats per condition, so the stability estimate is limited to those chromosomes and sample size.

## Evidence

- [abstract] cudaMMC calculations demonstrate significantly faster performance with better stability compared to the authors' previous method on the same workstation.: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [readme] Run of cudaMMC on a single chromosome used for timing benchmarking.: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] Build and hardware prerequisites for reproducing the cudaMMC timing benchmark on the same workstation.: "NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0)"
- [intro] cudaMMC is the GPU-accelerated counterpart of 3D-GNOME that the stability comparison evaluates.: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
