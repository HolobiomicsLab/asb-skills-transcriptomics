---
name: chromosome-level-3d-genome-reconstruction
description: Use when you have ChIA-PET anchor/cluster/singleton files (e.g. GM12878
  CTCF Rep1) and need per-chromosome 3D chromatin structure ensembles faster than
  3D-GNOME, with a CUDA GPU (compute capability >= 6.0) available.
when_to_use_negative:
- Only the previous 3D-GNOME workflow with its exact legacy settings must be reproduced
  bit-for-bit — cudaMMC is a GPU-enhanced reimplementation, not the original engine
  (original is at https://bitbucket.org/3dome/3dgnome).
- No NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0) is available
  and the Docker/CUDA build path cannot be used.
- Interchromosomal (genome-wide) structure reconstruction is the goal and per-chromosome
  splitting is not desired — although interchromosomal singletons can still be provided
  via singletons_inter.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: cudaMMC
  role: 'Primary engine: GPU-accelerated Multiscale Monte Carlo (Simulated Annealing)
    reconstruction of chromosome-level 3D structures from ChIA-PET data'
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: Previous CPU-based Multiscale Monte Carlo method and baseline; cudaMMC is
    its GPU-enhanced version
  repo: https://bitbucket.org/3dome/3dgnome
- name: Docker
  role: Containerized installation path for cudaMMC ('sudo docker build -t cudammc')
- name: CMake
  role: Build system to generate the makefile with -DCUDA_ARCH set to the GPU compute
    capability (>= 60)
- name: NVIDIA CUDA Toolkit
  role: Runtime dependency (11.0 or higher with compatible driver) enabling GPU-accelerated
    Monte Carlo simulation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:5483873e78938996ede3c761fa73380b41042f295ee268d5f14af4ce6433d186
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - C++
  - CMake
  - Docker
  - NVIDIA CUDA Toolkit
  - cudaMMC
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# chromosome-level-3d-genome-reconstruction

## Summary

Reconstruct ensembles of 3D chromatin structures for individual chromosomes from ChIA-PET data (anchors, PET clusters, singletons) using cudaMMC, the GPU-enhanced Multiscale Monte Carlo successor of 3D-GNOME, which enables fast ensemble generation for large chromosomes on GRCh37/human datasets.

## When to use

Use when you have ChIA-PET anchor/cluster/singleton files (e.g. GM12878 CTCF Rep1) and need per-chromosome 3D chromatin structure ensembles faster than 3D-GNOME, with a CUDA GPU (compute capability >= 6.0) available.

## When NOT to use

- Only the previous 3D-GNOME workflow with its exact legacy settings must be reproduced bit-for-bit — cudaMMC is a GPU-enhanced reimplementation, not the original engine (original is at https://bitbucket.org/3dome/3dgnome).
- No NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0) is available and the Docker/CUDA build path cannot be used.
- Interchromosomal (genome-wide) structure reconstruction is the goal and per-chromosome splitting is not desired — although interchromosomal singletons can still be provided via singletons_inter.

## Inputs

- ChIA-PET anchors file (chromosome, anchor start/end, optional CTCF motif orientation L/R/N)
- ChIA-PET PET cluster files per transcription factor (chromosome, anchor coordinates, PET_count)
- Intrachromosomal singleton files (read-pair coordinates + PET_count), optionally split per chromosome
- Interchromosomal singleton file (singletons_inter), skipped when modelling a single chromosome
- Optional BED files: segment split definition and centromere locations
- config.ini settings file with [main], [data], [distance] and [cuda] sections

## Outputs

- Ensemble of 3D chromatin structure models for the requested chromosome(s), labelled with the -n flag and written to the -o output directory
- Optionally, anchor and subanchor heatmaps used to refine loop and bead placement

## How to apply

Prepare the ChIA-PET inputs so filenames follow the 'GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt' convention, and if subanchor heatmaps will be generated set split_singleton_files_by_chr=yes in the config.ini [data] section and split intrachromosomal singletons per chromosome (via extract_singletons.sh or awk filtering on the chromosome column). Build cudaMMC either with Docker ('sudo docker build -t cudammc') or by generating a makefile with CMake (-DCUDA_ARCH set to the GPU compute capability, >= 60, e.g. '80' for Ampere; semicolon-separate for multi-architecture) and compiling with make, verifying the binary prints its usage message. Configure the [data], [main] and [distance] sections (e.g. max_pet_length, loop_density, use_motif_orientation, use_subanchor_heatmap, genomic_dist_power/scale, freq_dist_power/scale) and run per chromosome, e.g. './build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test'. The multiscale Monte Carlo simulation proceeds on the chromosome level (lvl1), segments (lvl2), anchors (arcs) and subanchors (smooth), with step counts and noise levels controlled in [main]. Optionally tune the [cuda] parameters (num threads, blocks multiplier, milestone fails) to trade off throughput and termination behavior.

## Related tools

- **cudaMMC** (Primary engine: GPU-accelerated Multiscale Monte Carlo (Simulated Annealing) reconstruction of chromosome-level 3D structures from ChIA-PET data) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (Previous CPU-based Multiscale Monte Carlo method and baseline; cudaMMC is its GPU-enhanced version) — https://bitbucket.org/3dome/3dgnome
- **Docker** (Containerized installation path for cudaMMC ('sudo docker build -t cudammc'))
- **CMake** (Build system to generate the makefile with -DCUDA_ARCH set to the GPU compute capability (>= 60))
- **NVIDIA CUDA Toolkit** (Runtime dependency (11.0 or higher with compatible driver) enabling GPU-accelerated Monte Carlo simulation)

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- The compiled binary prints its usage message when executed with no arguments, confirming a correct build.
- Timing benchmarks on chr1, chr14 and chr21 GM12878 CTCF ChIA-PET data should show cudaMMC outperforming 3D-GNOME on the same workstation (published speed-up of 3x–25x, dependent on chromosome size).
- Output files are written to the -o directory with names carrying the -n label (e.g. chr14_test).
- When split_singleton_files_by_chr=yes, the simulation consumes per-chromosome intrachromosomal singleton files rather than the aggregated file, and the interchromosomal singleton file is skipped for single-chromosome runs.
- Filenames of PET clusters match the required pattern 'GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt'.

## Limitations

- Requires an NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0), CUDA Toolkit 11.0+, CMake 3.13+, and a C++17 compliant compiler.
- The speed-up magnitude over 3D-GNOME is chromosome-size dependent (3x–25x), so small chromosomes benefit less.
- The -c option must be given per chromosome (default is 'chr1-chr22,chrX'); output directory must exist with a trailing '/'.
- Optionally providing centromere and segment-split BED files is needed for segment definitions; without them the segmentation relies on defaults.
- The [cuda] 'milestone fails' parameter causes the algorithm to stop after a set number of non-improved milestone scores, which can truncate refinement if set too low.

## Evidence

- [abstract] This study introduces the cudaMMC method, based on the Simulated Annealing Monte Carlo approach and enhanced by GPU-accelerated computing, to efficiently generate ensembles of chromatin 3D structures.: "This study introduces the cudaMMC method, based on the Simulated Annealing Monte Carlo approach and enhanced by GPU-accelerated computing, to efficiently generate ensembles of chromatin 3D structures."
- [readme] PET clusters should have filenames of the form 'GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt', where <M> corresponds to the flag -m: "PET clusters should have filenames of the form 'GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt'"
- [readme] When the subanchor heatmaps are to be generated it is beneficial to create the intrachromosomal singletons files for every chromosome separately: "When the subanchor heatmaps are to be generated it is beneficial to create the intrachromosomal singletons files for every chromosome separately"
- [methods] ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] cmake -B ./build -S ./ -DCUDA_ARCH="80": "cmake -B ./build -S ./ -DCUDA_ARCH="80""
