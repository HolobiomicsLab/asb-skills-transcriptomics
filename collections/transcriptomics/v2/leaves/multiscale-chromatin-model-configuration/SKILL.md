---
name: multiscale-chromatin-model-configuration
description: Use when generating ensembles of chromatin 3D models from GM12878 CTCF
  ChIA-PET data (anchors, PET clusters, singletons) and CPU-based 3D-GNOME runtimes
  on large chromosomes such as chr1 are prohibitive.
when_to_use_negative:
- The task only needs a web-based interactive run or promoter-enhancer contact analysis
  via Structural Variation — use the integrated 3D-GNOME 3.0 web server instead of
  local cudaMMC builds
- Input data is not ChIA-PET-derived anchors/clusters/singletons (e.g. Hi-C matrices),
  since the Multiscale Monte Carlo pipeline expects the anchor/cluster/singleton file
  formats described in the README
- No CUDA-capable NVIDIA GPU (compute capability < 6.0) or CUDA Toolkit 11.0+ is available
  and Docker GPU passthrough cannot be configured
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3678
tools:
- name: cudaMMC
  role: GPU-accelerated Simulated Annealing Monte Carlo engine that generates the
    chromatin 3D model ensembles from ChIA-PET inputs
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: Previous CPU-only Multiscale Monte Carlo version used as the timing baseline
    (Bitbucket release) and the codebase cudaMMC enhances
  repo: https://bitbucket.org/3dome/3dgnome
- name: 3D-GNOME 3.0
  role: Web server into which cudaMMC has been integrated for interactive use
- name: CMake 3.13
  role: Generates the makefile with the -DCUDA_ARCH flag (>= 60) matched to GPU compute
    capability before compilation
- name: NVIDIA CUDA Toolkit 11.0
  role: Required toolkit and driver to compile and run the GPU kernels
- name: Docker
  role: Alternative containerized installation path (sudo docker build -t cudammc)
    avoiding local CUDA compilation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:06bebd721377a7a12572f9a54a8f3f74183d48d1182ae27c45aded53185d20fd
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
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# multiscale-chromatin-model-configuration

## Summary

Configure and run cudaMMC, a GPU-enhanced multiscale Monte Carlo (Simulated Annealing) rebuild of 3D-GNOME, to generate ensembles of chromatin 3D structures from CTCF ChIA-PET anchor, cluster and singleton files, with per-chromosome GPU acceleration that cut 100-model ensemble runtimes from ~85 min to ~10 min (chr21), ~9 h to ~35 min (chr14) and ~3 days to 1.5 h (chr1).

## When to use

Use when generating ensembles of chromatin 3D models from GM12878 CTCF ChIA-PET data (anchors, PET clusters, singletons) and CPU-based 3D-GNOME runtimes on large chromosomes such as chr1 are prohibitive.

## When NOT to use

- The task only needs a web-based interactive run or promoter-enhancer contact analysis via Structural Variation — use the integrated 3D-GNOME 3.0 web server instead of local cudaMMC builds
- Input data is not ChIA-PET-derived anchors/clusters/singletons (e.g. Hi-C matrices), since the Multiscale Monte Carlo pipeline expects the anchor/cluster/singleton file formats described in the README
- No CUDA-capable NVIDIA GPU (compute capability < 6.0) or CUDA Toolkit 11.0+ is available and Docker GPU passthrough cannot be configured

## Inputs

- ChIA-PET anchors file (chromosome, anchor_start, anchor_end, optional CTCF motif orientation L/R/N)
- PET cluster files (chromosome_1, anchor_1_start, anchor_1_end, chromosome_2, anchor_2_start, anchor_2_end, PET_count), e.g. GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt
- Intrachromosomal singleton files (read-pair mapping coordinates + PET_count), optionally split per chromosome
- Optional interchromosomal singleton files (singletons_inter)
- Optional BED files with centromere locations and chromosome segment split definitions
- config.ini settings file with [main], [data], [distance] and [cuda] sections
- NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0)

## Outputs

- Ensemble of chromatin 3D structure models per chromosome (e.g. 100-model ensemble for chr21, chr14, chr1)
- Wall-clock timing comparison (e.g. timing_comparison.csv) of cudaMMC vs 3D-GNOME per chromosome
- Optionally subanchor heatmaps refined per chromosome when split_singleton_files_by_chr is enabled

## How to apply

Prepare the ChIA-PET inputs (anchors with optional CTCF motif orientation L/R/N, PET cluster files, and intrachromosomal singleton files split per chromosome via extract_singletons.sh or the provided awk procedure, enabling split_singleton_files_by_chr=yes in the [data] section of config.ini). Build cudaMMC with CMake (cmake -B ./build -S ./ -DCUDA_ARCH="80" to match your GPU compute capability >= 6.0, requiring CUDA Toolkit 11.0+ and a C++17 compiler) and compile with make --directory ./build, or use the Docker image. Run per chromosome, e.g. ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test, tuning simulation levels (steps_lvl1/lvl2/arcs/smooth), loop_density, and distance parameters (genomic_dist_power/scale, freq_dist_power/scale, max_pet_length) as needed. For a timing benchmark, run the previous CPU-only 3D-GNOME (Bitbucket release) on the same workstation with identical inputs and settings, and tabulate wall-clock times per chromosome per tool.

## Related tools

- **cudaMMC** (GPU-accelerated Simulated Annealing Monte Carlo engine that generates the chromatin 3D model ensembles from ChIA-PET inputs) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (Previous CPU-only Multiscale Monte Carlo version used as the timing baseline (Bitbucket release) and the codebase cudaMMC enhances) — https://bitbucket.org/3dome/3dgnome
- **3D-GNOME 3.0** (Web server into which cudaMMC has been integrated for interactive use)
- **CMake 3.13** (Generates the makefile with the -DCUDA_ARCH flag (>= 60) matched to GPU compute capability before compilation)
- **NVIDIA CUDA Toolkit 11.0** (Required toolkit and driver to compile and run the GPU kernels)
- **Docker** (Alternative containerized installation path (sudo docker build -t cudammc) avoiding local CUDA compilation)

## Examples

```
cmake -B ./build -S ./ -DCUDA_ARCH="80"; make --directory ./build; ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- Running ./build/cudaMMC without arguments prints the usage message, confirming a correct build
- Wall-clock timing table shows cudaMMC substantially faster than 3D-GNOME on the same workstation for each chromosome (chr21 ~85 min to ~10 min, chr14 ~9 h to ~35 min, chr1 ~3 days to ~1.5 h), matching Fig. 1C / Supplementary Table ST3 speed-up pattern
- cudaMMC runs complete with better stability than the previous CPU method on the same workstation (no failed/incomplete 100-model ensembles)
- config.ini flags take effect: per-chromosome singleton files (chr/data.txt.chr$i) are consumed only when split_singleton_files_by_chr=yes; per-chromosome output files are written to the -o directory with the -n label
- CMake reports the requested CUDA_ARCH value (e.g. 80) matching the installed GPU's compute capability

## Limitations

- Requires a UNIX-based OS, C++17 compiler (gcc/clang), CUDA Toolkit 11.0+, CMake 3.13+, and an NVIDIA GPU of Pascal architecture or newer (compute capability >= 6.0)
- Very large chromosomes remain non-trivial: even GPU-accelerated, chr1 takes ~1.5 h for a 100-model ensemble
- Aggregated (non-split) singleton files slow subanchor heatmap generation; interchromosomal singleton files are read unnecessarily when reconstructing a single chromosome unless provided separately
- Output directory specified with -o must already exist and carry a trailing '/'

## Evidence

- [intro] cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling.: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [methods] cudaMMC usage and run command: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] CMake CUDA_ARCH requirement: "generate the makefile with CMake by executing the command below from the root directory of the package with your desired `path_to_build_folder` and `cuda_arch` (>= 60) values"
- [readme] Hardware requirements: "NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0)"
- [readme] Singleton splitting rationale: "When the subanchor heatmaps are to be generated it is beneficial to create the intrachromosomal singletons files for every chromosome separately"
