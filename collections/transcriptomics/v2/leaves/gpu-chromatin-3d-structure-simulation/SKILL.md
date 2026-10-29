---
name: gpu-chromatin-3d-structure-simulation
description: Use when reconstructing 3D chromatin models from ChIA-PET anchors/PET
  clusters/singletons (e.g. GM12878 CTCF GRCh37/GRCh38) for single chromosomes or
  100-model ensembles on an NVIDIA GPU (compute capability >= 6.0), especially when
  CPU 3D-GNOME runtimes (hours to days) are prohibitive.
when_to_use_negative:
- No NVIDIA GPU with Pascal-or-newer architecture (compute capability >= 6.0) is available
  — the CPU-only 3D-GNOME from the Bitbucket repo should be used instead
- The task is interactive web-based modelling or promoter-enhancer contact analysis
  under structural variation — the 3D-GNOME 3.0 web server (https://3dgnome.mini.pw.edu.pl/)
  is the integrated interface for cudaMMC
- Input data is not ChIA-PET-style anchors/clusters/singletons (e.g. raw Hi-C contact
  matrices), which the 3D-GNOME input format does not cover
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0622
tools:
- name: cudaMMC
  role: GPU-enhanced multiscale Monte Carlo engine that runs the Simulated Annealing
    simulation per chromosome to produce 3D chromatin model ensembles
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: CPU-only baseline whose multiscale Monte Carlo settings are replicated for
    speed-up and model-quality comparison; previous version hosted on Bitbucket
  repo: https://bitbucket.org/3dome/3dgnome
- name: Docker
  role: Containerized build/launch of cudaMMC with GPU access ('sudo docker build
    -t cudammc' then 'docker run --gpus all')
- name: CMake
  role: Generates the makefile with the -DCUDA_ARCH flag matching the GPU compute
    capability (>= 60) for native compilation
- name: NVIDIA CUDA Toolkit 11.0
  role: Required toolchain (11.0 or higher with compatible driver) to compile and
    run the CUDA kernels
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:15861ad6abeb792200d3041c1e5fe2b79776320713cbcb89d9877c1fc463c329
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - 3D-GNOME 3.0
  - C++
  - CMake
  - Docker
  - NVIDIA CUDA Toolkit 11.0
  - cudaMMC
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gpu-chromatin-3d-structure-simulation

## Summary

Generate ensembles of chromatin 3D structures from CTCF ChIA-PET data using cudaMMC, a GPU-enhanced multiscale Monte Carlo (Simulated Annealing) reimplementation of 3D-GNOME that yields 3x-25x speed-ups and order-of-magnitude runtime reductions for large chromosomes while preserving within-ensemble model quality.

## When to use

Use when reconstructing 3D chromatin models from ChIA-PET anchors/PET clusters/singletons (e.g. GM12878 CTCF GRCh37/GRCh38) for single chromosomes or 100-model ensembles on an NVIDIA GPU (compute capability >= 6.0), especially when CPU 3D-GNOME runtimes (hours to days) are prohibitive.

## When NOT to use

- No NVIDIA GPU with Pascal-or-newer architecture (compute capability >= 6.0) is available — the CPU-only 3D-GNOME from the Bitbucket repo should be used instead
- The task is interactive web-based modelling or promoter-enhancer contact analysis under structural variation — the 3D-GNOME 3.0 web server (https://3dgnome.mini.pw.edu.pl/) is the integrated interface for cudaMMC
- Input data is not ChIA-PET-style anchors/clusters/singletons (e.g. raw Hi-C contact matrices), which the 3D-GNOME input format does not cover

## Inputs

- ChIA-PET anchors file (chromosome_id anchor_start anchor_end [orientation L/R/N])
- PET cluster files per transcription factor (chromosome_1 anchor_1_start anchor_1_end chromosome_2 anchor_2_start anchor_2_end PET_count), e.g. GM12878 CTCF Rep1 PET2+/PET3+ merged anchors
- Intrachromosomal singleton files, split per chromosome (split_singleton_files_by_chr = yes)
- Optional: interchromosomal singleton file, centromere locations BED, segment split definition BED
- config.ini settings file with [data], [cuda], [distance], [main] sections

## Outputs

- Ensembles of chromatin 3D structural models per chromosome (e.g. 100-model ensembles for chr1, chr14, chr21) under the -o output directory, labelled via -n
- Per-chromosome wall-clock timings (e.g. execution_times.csv / timing_comparison.csv) for cudaMMC vs 3D-GNOME
- Within-ensemble pairwise structural-similarity matrices and density plots of model similarity scores

## How to apply

Prepare the ChIA-PET inputs in 3D-GNOME format — an anchors file (chromosome, anchor_start, anchor_end, optional CTCF motif orientation L/R/N), PET cluster files named per the 'GM12878_CTCF_Rep1.withLinker.clusters.intra.PET<N>+.merged_anchors.simple.txt' convention, and intrachromosomal singleton files split per chromosome (via extract_singletons.sh or awk, setting split_singleton_files_by_chr = yes) — and wire their paths in the [data] section of config.ini. Build cudaMMC from https://github.com/SFGLab/cudaMMC either with Docker ('sudo docker build -t cudammc' and 'docker run --gpus all') or with CMake 3.13+ / CUDA Toolkit 11.0+, setting -DCUDA_ARCH to your GPU compute capability (>= 60, e.g. "80" for Ampere) and compiling with make. Configure the [cuda] parameters (num_threads, blocks_multiplier, milestone_fails) plus the 3D-GNOME-inherited distance/springs/simulation_* sections (freq_dist_power, genomic_dist_scale, use_motif_orientation, use_subanchor_heatmap, steps_lvl1/lvl2/arcs/smooth). Launch one run per chromosome with './build/cudaMMC -s config.ini -c chr14 -o <outdir>/ -n <label>' (-c defaults to 'chr1-chr22,chrX' and accepts regions like 'chr14:1:2500000'). Rationale: the GPU parallelization of the multiscale Monte Carlo levels (chromosome, segments, anchor, subanchor) reduces chr21 100-model ensemble time from ~85 to ~10 min, chr14 from ~9 h to 35 min, and chr1 from ~3 days to 1.5 h, with model quality matching 3D-GNOME (within-ensemble similarity medians differ by <1%).

## Related tools

- **cudaMMC** (GPU-enhanced multiscale Monte Carlo engine that runs the Simulated Annealing simulation per chromosome to produce 3D chromatin model ensembles) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (CPU-only baseline whose multiscale Monte Carlo settings are replicated for speed-up and model-quality comparison; previous version hosted on Bitbucket) — https://bitbucket.org/3dome/3dgnome
- **Docker** (Containerized build/launch of cudaMMC with GPU access ('sudo docker build -t cudammc' then 'docker run --gpus all'))
- **CMake** (Generates the makefile with the -DCUDA_ARCH flag matching the GPU compute capability (>= 60) for native compilation)
- **NVIDIA CUDA Toolkit 11.0** (Required toolchain (11.0 or higher with compatible driver) to compile and run the CUDA kernels)

## Examples

```
cmake -B ./build -S ./ -DCUDA_ARCH="80" ; make --directory ./build ; ./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- The compiled binary prints its usage message when executed without arguments (./build/cudaMMC)
- Per-chromosome wall-clock runtimes follow the expected ordering chr1 > chr14 > chr21, with reference timings ~50 min (chr1), ~17 min (chr14), ~4.5 min (chr21) on in situ GRCh38 data
- Speed-up over 3D-GNOME falls within the published 3x-25x range for single chromosomes, and 100-model ensembles show order-of-magnitude reductions (chr21 ~85->~10 min, chr14 ~9 h->35 min, chr1 ~3 days->1.5 h)
- Within-ensemble similarity distributions for cudaMMC and 3D-GNOME (100 chr21 models each) overlap visually and their medians differ by <1% (reproducing Supplementary Fig. S3)
- Timing tables contain the expected row counts (e.g. 3 chromosomes x 3 runs for the in situ GRCh38 benchmark)

## Limitations

- GPU speed-up magnitude is chromosome-size dependent (3x-25x for single chromosomes), so small chromosomes show the smallest gains
- Requires an NVIDIA GPU of Pascal architecture or newer (compute capability >= 6.0), CUDA Toolkit 11.0+, CMake 3.13+, and a C++17-compliant compiler; -DCUDA_ARCH must match the GPU or compilation will target the wrong architecture
- Intrachromosomal singleton files should be split per chromosome (split_singleton_files_by_chr) when subanchor heatmaps are used, otherwise performance/behaviour may deviate from the benchmarked configuration

## Evidence

- [intro] GPU-enhanced multiscale Monte Carlo chromatin 3D modelling: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [abstract] speed-up and stability: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [methods] example invocation: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] singleton splitting: "split_singleton_files_by_chr - if *yes* then the chromosome-splitted intrachromosmal files are used"
- [readme] hardware requirements: "NVIDIA GPU with Pascal architecture or newer (compute capability >= 6.0)"
