---
name: same-workstation-wall-clock-benchmarking
description: Use when measuring how much a GPU-accelerated reimplementation (e.g.
  cudaMMC vs 3D-GNOME) reduces wall-clock time for generating 100-model chromatin
  3D ensembles per chromosome on one fixed workstation.
when_to_use_negative:
- The goal is assessing model accuracy or structural quality (e.g. enhancer-promoter
  distances) rather than computational runtime
- The two tools would run on different hardware (e.g. cloud CPU vs local GPU), which
  invalidates the same-workstation comparison rationale
- Data is not ChIA-PET anchors/clusters/singletons in the format cudaMMC and 3D-GNOME
  accept (e.g. Hi-C contact matrices without the required anchor/PET structure)
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: cudaMMC
  role: GPU-enhanced Simulated Annealing Monte Carlo implementation whose wall-clock
    runtime is measured for 100-model ensembles per chromosome
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME 3.0
  role: CPU-only predecessor (Bitbucket release) run with identical settings on the
    same workstation to provide the baseline wall-clock times
  repo: https://bitbucket.org/3dome/3dgnome
- name: Docker
  role: Alternative containerized installation path for building/running cudaMMC identically
    to the source build
  repo: https://github.com/SFGLab/cudaMMC
- name: CMake
  role: Generates the makefile with -DCUDA_ARCH set to the GPU compute capability
    so cudaMMC is compiled for the benchmark workstation
  repo: https://github.com/SFGLab/cudaMMC
- name: NVIDIA CUDA Toolkit 11.0
  role: Required toolkit and compatible driver enabling the GPU-accelerated Monte
    Carlo runs being timed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:b1d2e27a28dd2d261133def31da429a10fc4b9d08f06d33ecc0ab8d82028551b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME 3.0
  - CMake
  - Docker
  - NVIDIA CUDA Toolkit 11.0
  - cudaMMC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# same-workstation-wall-clock-benchmarking

## Summary

Benchmark the wall-clock runtime of the GPU-enhanced cudaMMC implementation against the CPU-only 3D-GNOME predecessor by running both tools on the identical workstation with identical GM12878 CTCF ChIA-PET inputs, to quantify computation-time reduction for 100-model chromatin 3D ensembles.

## When to use

Use when measuring how much a GPU-accelerated reimplementation (e.g. cudaMMC vs 3D-GNOME) reduces wall-clock time for generating 100-model chromatin 3D ensembles per chromosome on one fixed workstation.

## When NOT to use

- The goal is assessing model accuracy or structural quality (e.g. enhancer-promoter distances) rather than computational runtime
- The two tools would run on different hardware (e.g. cloud CPU vs local GPU), which invalidates the same-workstation comparison rationale
- Data is not ChIA-PET anchors/clusters/singletons in the format cudaMMC and 3D-GNOME accept (e.g. Hi-C contact matrices without the required anchor/PET structure)

## Inputs

- GRCh37 GM12878 CTCF ChIA-PET data: anchors file, PET cluster files (e.g. GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt), and inter-/intrachromosomal singleton files
- config.ini settings file shared by both tools
- Workstation with an NVIDIA GPU (compute capability >= 6.0), CUDA Toolkit 11.0+, CMake 3.13+, C++17 compiler (or the cudammc Docker image)
- 3D-GNOME release from the Bitbucket repository as the CPU baseline

## Outputs

- Ensemble of 100 chromatin 3D models per chromosome from cudaMMC
- Ensemble of 100 chromatin 3D models per chromosome from 3D-GNOME (baseline)
- Recorded wall-clock times per chromosome per tool
- timing_comparison.csv tabulating runtime and speed-up per chromosome

## How to apply

Install both tools on the same workstation: build cudaMMC from source with CMake (cmake -B ./build -S ./ -DCUDA_ARCH="80" matching the GPU compute capability, CUDA Toolkit 11.0+, Pascal-or-newer NVIDIA GPU, CMake 3.13+) or via Docker, and use the previous 3D-GNOME release from Bitbucket for the baseline. Run both on the same GRCh37 GM12878 CTCF ChIA-PET data (anchors, PET clusters, singletons) with identical settings in config.ini, per chromosome (chr21, chr14, chr1), launching simulations like ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test and recording the wall-clock time to produce the 100-model ensemble. Tabulate per-chromosome, per-tool times into timing_comparison.csv and compare against the published reductions (chr21 ~85 min to ~10 min, chr14 ~9 h to ~35 min, chr1 ~3 days to ~1.5 h; Fig. 1C, Supplementary Table ST3). The rationale is that GPU acceleration via Simulated Annealing Monte Carlo yields order-of-magnitude speed-ups that only hold as a fair comparison when hardware, inputs, and settings are held constant.

## Related tools

- **cudaMMC** (GPU-enhanced Simulated Annealing Monte Carlo implementation whose wall-clock runtime is measured for 100-model ensembles per chromosome) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME 3.0** (CPU-only predecessor (Bitbucket release) run with identical settings on the same workstation to provide the baseline wall-clock times) — https://bitbucket.org/3dome/3dgnome
- **Docker** (Alternative containerized installation path for building/running cudaMMC identically to the source build) — https://github.com/SFGLab/cudaMMC
- **CMake** (Generates the makefile with -DCUDA_ARCH set to the GPU compute capability so cudaMMC is compiled for the benchmark workstation) — https://github.com/SFGLab/cudaMMC
- **NVIDIA CUDA Toolkit 11.0** (Required toolkit and compatible driver enabling the GPU-accelerated Monte Carlo runs being timed)

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- chr21 timing reproduces the reported reduction from ~85 min (3D-GNOME) to ~10 min (cudaMMC) on the same workstation
- chr14 timing reproduces the reported reduction from ~9 h to ~35 min
- chr1 timing reproduces the reported reduction from ~3 days to ~1.5 h
- Both tools were executed with identical config.ini settings and identical GRCh37 GM12878 CTCF ChIA-PET input files, differing only in the executable
- The tabulated speed-ups match the order-of-magnitude pattern of Fig. 1C and Supplementary Table ST3

## Limitations

- Reported runtimes (~85 to ~10 min for chr21, ~9 h to ~35 min for chr14, ~3 days to ~1.5 h for chr1) are specific to the authors' workstation hardware and GPU; other GPUs/CUDA_ARCH values will differ
- CPU-only 3D-GNOME runs are prohibitively slow for large chromosomes (chr1 takes days), limiting how many chromosomes can be benchmarked per workstation run
- No changelog or discussion-section detail is available in the source document to explain version-specific performance differences

## Evidence

- [abstract] Faster runtime and better stability on the same workstation: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [abstract] cudaMMC reduces computation time for large chromatin ensembles: "cudaMMC also substantially reduces the computation time required for generating ensembles of large chromatin models"
- [intro] cudaMMC is a GPU-enhanced version of 3D-GNOME: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [methods] Per-chromosome timing run command: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [results] Reported per-chromosome wall-clock reductions for 100-model ensembles: [evidence span withheld — anchor status: not_found]
- [readme] Hardware requirements for the timed GPU runs: "NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0)"
