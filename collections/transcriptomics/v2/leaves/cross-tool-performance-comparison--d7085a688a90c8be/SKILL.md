---
name: cross-tool-performance-comparison
description: Use when comparing computation time of a new GPU-accelerated implementation
  against its CPU predecessor on the same workstation, inputs, and settings (e.g.
  cudaMMC vs 3D-GNOME on GM12878 CTCF ChIA-PET data).
when_to_use_negative:
- Comparing tools that produce different outputs or operate on different input formats
  — only same-task, same-input benchmarks support a fair timing comparison
- The target chromosomes or data are not available for both tools (e.g. the legacy
  3D-GNOME Bitbucket release cannot be run on the workstation)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: cudaMMC
  role: GPU-accelerated tool whose runtime is measured on 100-model ensembles per
    chromosome
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: CPU-only baseline whose runtime is measured on identical inputs/workstation;
    previous version hosted on Bitbucket
  repo: https://bitbucket.org/3dome/3dgnome
- name: Docker
  role: Optional containerized installation path for cudaMMC to standardize the benchmark
    environment
- name: CMake 3.13
  role: Generates the makefile for building cudaMMC with the GPU-appropriate CUDA_ARCH
- name: NVIDIA CUDA Toolkit 11.0
  role: Required CUDA toolchain for compiling and running cudaMMC on the benchmark
    GPU
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:7c252438dae8c707e48ae4ed44e8a416384de31b64b20de63ac1a7aae1758fd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - 3D-GNOME 3.0
  - CMake
  - CMake 3.13
  - Docker
  - NVIDIA CUDA Toolkit 11.0
  - cudaMMC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# cross-tool-performance-comparison

## Summary

Benchmarks the wall-clock runtime of two functionally equivalent tools (here, GPU-based cudaMMC vs CPU-based 3D-GNOME) on identical input data and hardware to quantify speed-up for generating 100-model chromatin ensembles.

## When to use

Use when comparing computation time of a new GPU-accelerated implementation against its CPU predecessor on the same workstation, inputs, and settings (e.g. cudaMMC vs 3D-GNOME on GM12878 CTCF ChIA-PET data).

## When NOT to use

- Comparing tools that produce different outputs or operate on different input formats — only same-task, same-input benchmarks support a fair timing comparison
- The target chromosomes or data are not available for both tools (e.g. the legacy 3D-GNOME Bitbucket release cannot be run on the workstation)

## Inputs

- CTCFC ChIA-PET anchors file (e.g. GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt)
- PET cluster files and intrachromosomal singleton files, split per chromosome (chr21, chr14, chr1)
- cudaMMC config.ini with [data] and [cuda] sections
- Same workstation hardware (NVIDIA GPU with compute capability >= 6.0) for both tools

## Outputs

- 100-model 3D chromatin ensembles per chromosome from each tool
- Per-chromosome, per-tool wall-clock timing table (timing_comparison.csv)
- Speed-up ratios (cudaMMC vs 3D-GNOME) per chromosome

## How to apply

Prepare identical inputs for both tools — CTCF ChIA-PET anchors, PET clusters, and singletons for GRCh37 GM12878, with intrachromosomal singleton files split per chromosome (extract_singletons.sh or awk) and split_singleton_files_by_chr enabled in config.ini. Build cudaMMC from source with CMake matching the GPU compute capability (e.g. -DCUDA_ARCH="80", requiring CUDA Toolkit 11.0+ and a Pascal-or-newer NVIDIA GPU) or use the Docker image, then run it per chromosome (chr21, chr14, chr1) to produce 100-model ensembles while recording wall-clock time. Run the previous CPU-only 3D-GNOME (Bitbucket release) on the same workstation with identical settings and data, recording its wall-clock times. Tabulate per-chromosome, per-tool times into a CSV (e.g. timing_comparison.csv) and compute the speed-up ratios per chromosome, referencing the published reductions (Fig. 1C, Supplementary Table ST3).

## Related tools

- **cudaMMC** (GPU-accelerated tool whose runtime is measured on 100-model ensembles per chromosome) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (CPU-only baseline whose runtime is measured on identical inputs/workstation; previous version hosted on Bitbucket) — https://bitbucket.org/3dome/3dgnome
- **Docker** (Optional containerized installation path for cudaMMC to standardize the benchmark environment)
- **CMake 3.13** (Generates the makefile for building cudaMMC with the GPU-appropriate CUDA_ARCH)
- **NVIDIA CUDA Toolkit 11.0** (Required CUDA toolchain for compiling and running cudaMMC on the benchmark GPU)

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- Timing table reproduces the reported pattern: cudaMMC ~10 min vs 3D-GNOME ~85 min for chr21; ~35 min vs ~9 h for chr14; ~1.5 h vs ~3 days for chr1 (Fig. 1C, Supplementary Table ST3)
- Speed-up is order-of-magnitude for each tested chromosome, with the largest absolute savings on the longest chromosome (chr1)
- Both tools were run on the same workstation with identical input data and settings — verify identical config.ini and file inputs were used
- cudaMMC binary produces valid output (usage message on bare invocation; 100 models per run as configured)

## Limitations

- Timing is workstation- and GPU-dependent; CUDA_ARCH in CMake must match the GPU compute capability (>= 60) or the build/run may fail
- The reported speed-ups are for GRCh37 GM12878 CTCF ChIA-PET data on specific chromosomes; other data or chromosome sizes may scale differently

## Evidence

- [abstract] Reported wall-clock reductions for 100-model ensembles: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [methods] Benchmark execution command and per-chromosome runs: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] Build configuration matched to GPU architecture: "Example for GPU with compute capability >= 8.0:
cmake -B ./build -S ./ -DCUDA_ARCH="80""
- [readme] Hardware/software prerequisites for the GPU benchmark: "NVIDIA CUDA Toolkit 11.0 or higher with compatible NVIDIA driver"
- [readme] Input splitting required to make per-chromosome runs comparable: "When the subanchor heatmaps are to be generated it is beneficial to create the intrachromosomal singletons files for every chromosome separately"
