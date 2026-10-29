---
name: gpu-bead-chain-energy-minimization
description: Use when reconstructing 3D chromatin structures from ChIA-PET anchors,
  PET clusters, and singletons on an NVIDIA GPU (Pascal or newer, CUDA_ARCH >= 60)
  and you need warp-parallel per-bead Monte Carlo energy minimization.
when_to_use_negative:
- Running on GPUs older than Pascal (compute capability < 6.0) — CMake requires CUDA_ARCH
  >= 60
- No GPU available and container-free CPU execution is required — cudaMMC is GPU-enhanced;
  the CPU-based 3D-GNOME (bitbucket.org/3dome/3dgnome) is the alternative
- Interpreting finished models rather than generating them — use the 3D-GNOME 3.0
  web server analysis features instead of re-running the MC kernel
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3168
tools:
- name: cudaMMC
  role: GPU-enhanced Multiscale Monte Carlo software whose kernel assigns one warp
    per bead for simulated annealing fitting of chromatin beads
  repo: https://github.com/SFGLab/cudaMMC
- name: NVIDIA CUDA Toolkit 11.0
  role: Provides the CUDA runtime/toolchain required to compile and run the warp-parallel
    kernel (11.0 or higher with compatible driver)
- name: CMake 3.13
  role: Generates the makefile with the required -DCUDA_ARCH (>= 60) setting before
    compilation
- name: 3D-GNOME
  role: CPU predecessor whose settings and Multiscale Monte Carlo approach cudaMMC
    extends; the [cuda] parameters are additive to 3D-GNOME config options
  repo: https://bitbucket.org/3dome/3dgnome
- name: Docker
  role: Alternative installation path via 'docker build -t cudammc' then 'docker run
    --gpus all'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:14bbcb53fe3e0534360218d9d3893208b24184a4d054dc92ed461099e3b5fffd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - C++
  - CMake
  - CMake 3.13
  - Docker
  - NVIDIA CUDA Toolkit 11.0
  - cudaMMC
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gpu-bead-chain-energy-minimization

## Summary

Implements cudaMMC's warp-level parallel Simulated Annealing Monte Carlo search, where each 32-thread warp optimizes one chromatin bead by iteratively proposing random displacement moves and reducing to the lowest-energy move without global synchronization. This is the GPU kernel strategy that makes cudaMMC substantially faster and more stable than CPU-based 3D-GNOME for large chromatin model ensembles.

## When to use

Use when reconstructing 3D chromatin structures from ChIA-PET anchors, PET clusters, and singletons on an NVIDIA GPU (Pascal or newer, CUDA_ARCH >= 60) and you need warp-parallel per-bead Monte Carlo energy minimization.

## When NOT to use

- Running on GPUs older than Pascal (compute capability < 6.0) — CMake requires CUDA_ARCH >= 60
- No GPU available and container-free CPU execution is required — cudaMMC is GPU-enhanced; the CPU-based 3D-GNOME (bitbucket.org/3dome/3dgnome) is the alternative
- Interpreting finished models rather than generating them — use the 3D-GNOME 3.0 web server analysis features instead of re-running the MC kernel

## Inputs

- ChIA-PET anchor file (chromosome_id anchor_start anchor_end [orientation L/R/N])
- PET cluster files (chromosome_id_1 anchor_1_start anchor_1_end chromosome_id_2 anchor_2_start anchor_2_end PET_count)
- Singleton files (intrachromosomal 'singletons' and interchromosomal 'singletons_inter') in cluster-file format
- config.ini settings file with [cuda], [main], [data], and [distance] sections
- Optional BED files for centromeres and segment split definitions

## Outputs

- Ensembles of chromatin 3D structures (labeled output files via -n)
- Milestone energy trace (energy_trace.csv) for validation runs
- Optionally anchor and subanchor heatmaps refining chromatin loop modeling

## How to apply

Build with CMake specifying -DCUDA_ARCH (>= 60, semicolon-separated for multi-architecture binaries like "60;70;80") and a C++17 compiler with NVIDIA CUDA Toolkit 11.0+, then compile with make. In the kernel, assign one 32-thread warp per bead: each thread proposes random displacement moves for that bead, and a warp-level parallel reduction selects the move with the lowest energy. Implement the global acceptance step so each warp updates using the temporal energies of other warps with no global synchronization points, avoiding barrier-induced stalls. Expose the [cuda] config.ini parameters 'num threads' (CUDA threads per block), 'blocks multiplier' (blocks multiplied by bead count), and 'milestone fails' (number of non-improved milestone scores before stopping), then run on a toy random-walk bead chain and record milestone energies to energy_trace.csv.

## Related tools

- **cudaMMC** (GPU-enhanced Multiscale Monte Carlo software whose kernel assigns one warp per bead for simulated annealing fitting of chromatin beads) — https://github.com/SFGLab/cudaMMC
- **NVIDIA CUDA Toolkit 11.0** (Provides the CUDA runtime/toolchain required to compile and run the warp-parallel kernel (11.0 or higher with compatible driver))
- **CMake 3.13** (Generates the makefile with the required -DCUDA_ARCH (>= 60) setting before compilation)
- **3D-GNOME** (CPU predecessor whose settings and Multiscale Monte Carlo approach cudaMMC extends; the [cuda] parameters are additive to 3D-GNOME config options) — https://bitbucket.org/3dome/3dgnome
- **Docker** (Alternative installation path via 'docker build -t cudammc' then 'docker run --gpus all')

## Examples

```
cmake -B ./build -S ./ -DCUDA_ARCH="80"; make --directory ./build; ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- The energy_trace.csv milestone energies decrease monotonically until the 'milestone fails' stop condition (the configured number of non-improved milestone scores) terminates the run
- Running the compiled binary with no arguments prints the cudaMMC usage message, confirming the build completed properly
- The kernel contains no global synchronization points — warps update from other warps' temporal energies without barrier-induced stalls
- Block geometry is consistent with config: threads per block equals 'num threads' and block count equals 'blocks multiplier' times the bead count
- On the same workstation, runtime and stability are better than the CPU-based previous method for equivalent large chromatin model ensembles

## Limitations

- Requires an NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0) and CUDA Toolkit 11.0+ with a compatible driver
- 'milestone fails' termination means the search stops after a set number of non-improved milestone scores, so convergence depends on an arbitrarily selected number of per-thread random move attempts
- Interchromosomal singleton files can be large; when reconstructing a single chromosome, skip reading them and split intrachromosomal singletons per chromosome (split_singleton_files_by_chr) before subanchor heatmap generation

## Evidence

- [methods] warp-per-bead parallelization: [evidence span withheld — anchor status: not_found]
- [methods] no global sync rationale: [evidence span withheld — anchor status: not_found]
- [abstract] performance claim: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [readme] build requirement: "NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0)"
- [readme] cuda config parameters: "milestone fails - this parameter determines after how many non improved milestone scores, the algorithm will stop"
