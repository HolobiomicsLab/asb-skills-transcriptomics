---
name: monte-carlo-simulated-annealing-implementation
description: Use when reconstructing 3D chromatin structures from ChIA-PET anchors/clusters/singletons
  and a GPU-parallel Monte Carlo bead-fitting kernel with warp-level move selection
  must be implemented or revalidated.
when_to_use_negative:
- The target is a CPU-based or non-GPU Monte Carlo implementation; this skill's kernel
  design is specific to CUDA warps (compute capability >= 6.0) and warp-level reductions.
- The goal is only to run cudaMMC end-to-end on ChIA-PET data without modifying or
  validating the annealing kernel itself — use the standard build/run workflow instead.
- Inputs are already-finished 3D chromatin models needing only downstream analysis
  (e.g. enhancer-promoter distance analysis), not bead-position optimization.
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0797
tools:
- name: cudaMMC
  role: The GPU-enhanced Multiscale Monte Carlo software whose simulated annealing
    kernel this skill implements and validates
  repo: https://github.com/SFGLab/cudaMMC
- name: NVIDIA CUDA Toolkit
  role: Provides the CUDA toolchain (11.0 or higher) required to compile the warp-parallel
    kernel
- name: CMake
  role: Generates the makefile with the -DCUDA_ARCH flag (>= 60) before compiling
    with make
- name: 3D-GNOME
  role: Predecessor software whose Monte Carlo methodology cudaMMC extends; config
    parameters (freq_dist_power, genomic_dist_scale, etc.) carry over
  repo: https://bitbucket.org/3dome/3dgnome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:7ce3567fc1abda3eef531352e86e4900b9222d33f183aa9d953ec0eb41f7adc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - C++
  - CMake
  - NVIDIA CUDA Toolkit
  - cudaMMC
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# monte-carlo-simulated-annealing-implementation

## Summary

Implements the GPU-parallel Simulated Annealing Monte Carlo search used by cudaMMC to fit 3D chromatin bead models, assigning one 32-thread CUDA warp per bead so threads propose random displacement moves and a warp-level reduction selects the lowest-energy move without global synchronization.

## When to use

Use when reconstructing 3D chromatin structures from ChIA-PET anchors/clusters/singletons and a GPU-parallel Monte Carlo bead-fitting kernel with warp-level move selection must be implemented or revalidated.

## When NOT to use

- The target is a CPU-based or non-GPU Monte Carlo implementation; this skill's kernel design is specific to CUDA warps (compute capability >= 6.0) and warp-level reductions.
- The goal is only to run cudaMMC end-to-end on ChIA-PET data without modifying or validating the annealing kernel itself — use the standard build/run workflow instead.
- Inputs are already-finished 3D chromatin models needing only downstream analysis (e.g. enhancer-promoter distance analysis), not bead-position optimization.

## Inputs

- config.ini settings file with [cuda] section (num threads, blocks multiplier, milestone fails)
- Toy bead chain for testing (short random-walk bead chain is sufficient)
- ChIA-PET anchor, PET cluster, and singleton files (e.g. GM12878_CTCF Rep1 clusters) for full simulations

## Outputs

- energy_trace.csv recording the energy value at each milestone
- Ensembles of chromatin 3D structures from the simulated annealing search

## How to apply

Set up the build with the NVIDIA CUDA Toolkit (11.0+), a C++17 compiler, and CMake 3.13+, generating the makefile with -DCUDA_ARCH set to a value >= 60 (Pascal or newer). Implement the kernel so each 32-thread warp is assigned a single bead and its threads iteratively propose random displacement moves, then a warp-level parallel reduction selects the move with the lowest energy for that warp. Implement the global acceptance step so each warp updates using temporal energies of other warps with no global synchronization points, avoiding barrier-induced stalls. Expose the [cuda] config.ini parameters 'num threads' (CUDA threads per block), 'blocks multiplier' (blocks multiplied by bead count), and 'milestone fails' (non-improved milestone scores before stopping). Compile with make --directory ./build and validate by running the kernel on a toy random-walk bead chain, recording milestone energies to energy_trace.csv.

## Related tools

- **cudaMMC** (The GPU-enhanced Multiscale Monte Carlo software whose simulated annealing kernel this skill implements and validates) — https://github.com/SFGLab/cudaMMC
- **NVIDIA CUDA Toolkit** (Provides the CUDA toolchain (11.0 or higher) required to compile the warp-parallel kernel)
- **CMake** (Generates the makefile with the -DCUDA_ARCH flag (>= 60) before compiling with make)
- **3D-GNOME** (Predecessor software whose Monte Carlo methodology cudaMMC extends; config parameters (freq_dist_power, genomic_dist_scale, etc.) carry over) — https://bitbucket.org/3dome/3dgnome

## Examples

```
cmake -B ./build -S ./ -DCUDA_ARCH="80"; make --directory ./build; ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- The energy trace in energy_trace.csv decreases monotonically until the 'milestone fails' stop condition (non-improved milestone scores trigger termination), confirming correct kernel operation.
- The binary compiles for CUDA_ARCH >= 60 and the compiled ./build/cudaMMC binary prints its usage message when run without arguments.
- One warp (32 threads) handles exactly one bead, and no global synchronization/barrier calls appear in the acceptance step.
- The [cuda] section parameters 'num threads', 'blocks multiplier', and 'milestone fails' are read from config.ini and affect block count as blocks multiplier x bead count.
- On a toy random-walk bead chain the milestone scores improve for a number of iterations before the stop condition fires.

## Limitations

- Requires an NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0); older GPUs are unsupported.
- The number of random moves each thread attempts is an arbitrarily selected number, not a theoretically derived value.
- Avoiding global synchronization means warp updates rely on temporal energies of other warps, which trades strict determinism for performance.

## Evidence

- [results] The GPU search assigns each 32-thread warp to a single bead, whose threads iteratively try random moves of that bead: [evidence span withheld — anchor status: not_found]
- [readme] num threads - this parameter specifies the number of CUDA threads to be launched per block: "num threads - this parameter specifies the number of CUDA threads to be launched per block"
- [readme] milestone fails - this parameter determines after how many non improved milestone scores, the algorithm will stop: "milestone fails - this parameter determines after how many non improved milestone scores, the algorithm will stop"
- [readme] NVIDIA CUDA Toolkit 11.0 or higher with compatible NVIDIA driver: "NVIDIA CUDA Toolkit 11.0 or higher with compatible NVIDIA driver"
- [abstract] This study introduces the cudaMMC method, based on the Simulated Annealing Monte Carlo approach and enhanced by GPU-accelerated computing: "This study introduces the cudaMMC method, based on the Simulated Annealing Monte Carlo approach and enhanced by GPU-accelerated computing"
