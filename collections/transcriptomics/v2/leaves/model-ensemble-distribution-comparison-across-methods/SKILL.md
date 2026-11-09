---
name: model-ensemble-distribution-comparison-across-methods
description: Use when porting or re-implementing a stochastic 3D chromatin modelling
  tool and you need to demonstrate the new version produces statistically equivalent
  ensembles, by comparing within-ensemble similarity distributions.
when_to_use_negative:
- Input is a single 3D model per method — this skill requires an ensemble of ~100
  models per method to form a within-ensemble similarity distribution
- The goal is a runtime/performance benchmark rather than model-quality equivalence
  — use timing benchmarks instead, since cudaMMC's headline advantage is speed
- Comparing models from different chromosomes or non-equivalent settings — the ensembles
  must be chr21 (or the same locus) with matched simulation parameters
tools:
- name: cudaMMC
  role: Generates the candidate ensemble of 100 chr21 chromatin models whose within-ensemble
    similarity distribution is compared
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: Reference method whose 100-model chr21 ensemble provides the baseline similarity
    distribution for equivalence testing
  repo: https://bitbucket.org/3dome/3dgnome
- name: Docker
  role: Containerized build/run environment for cudaMMC ('sudo docker build -t cudammc')
    so both method runs are reproducible
- name: CMake
  role: Generates the makefile for native compilation of cudaMMC with -DCUDA_ARCH
    set to the GPU compute capability (>=60)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:1a41ceb46992578895d6a1902a24efe2ef5926ed2273b538b817b8e1eb3358eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - C++
  - CMake
  - Docker
  - cudaMMC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# model-ensemble-distribution-comparison-across-methods

## Summary

Validates that a rewritten or ported structure-modelling engine (e.g. cudaMMC vs. its predecessor 3D-GNOME) preserves model quality by comparing within-ensemble similarity distributions of two ensembles of 100 chromatin models for the same locus (chr21). It quantifies method equivalence as a difference in distribution medians of less than 1%.

## When to use

Use when porting or re-implementing a stochastic 3D chromatin modelling tool and you need to demonstrate the new version produces statistically equivalent ensembles, by comparing within-ensemble similarity distributions.

## When NOT to use

- Input is a single 3D model per method — this skill requires an ensemble of ~100 models per method to form a within-ensemble similarity distribution
- The goal is a runtime/performance benchmark rather than model-quality equivalence — use timing benchmarks instead, since cudaMMC's headline advantage is speed
- Comparing models from different chromosomes or non-equivalent settings — the ensembles must be chr21 (or the same locus) with matched simulation parameters

## Inputs

- cudaMMC settings file (config.ini) with ChIA-PET data inputs (anchors, PET clusters, singletons, e.g. GM12878 CTCF files)
- Ensemble of 100 chr21 models produced by cudaMMC
- Ensemble of 100 chr21 models produced by the reference 3D-GNOME method with equivalent settings
- Pairwise structural-similarity comparison scripts from the package benchmark directory

## Outputs

- Pairwise structural-similarity matrix over 100 models for each method's ensemble
- Overlaid density plots of within-ensemble similarity-score distributions (cudaMMC vs 3D-GNOME), reproducing Supplementary Fig. S3
- Summary table of distribution statistics (median, spread) per method, with median difference <1%

## How to apply

Build the new engine (cudaMMC via 'sudo docker build -t cudammc --progress=plain .' or CMake with -DCUDA_ARCH>=60 and 'make --directory ./build') and launch 100 independent simulations for chr21 with './build/cudaMMC -s config.ini -c chr21 -o <outdir>/ -n <label>'; in parallel, generate 100 chr21 models with the reference method (3D-GNOME, from its Bitbucket repo) using equivalent settings. Compute a pairwise structural-similarity matrix across the 100 models within each ensemble using the comparison scripts in the package's benchmark directory. Overlay the two within-ensemble similarity-score distributions as density plots (matplotlib/ggplot2) and compute per-ensemble medians and spread. Method equivalence is judged by visual overlap of the density curves (reproducing Supplementary Fig. S3) and a median difference below 1%; assemble a summary table of median and spread per method.

## Related tools

- **cudaMMC** (Generates the candidate ensemble of 100 chr21 chromatin models whose within-ensemble similarity distribution is compared) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (Reference method whose 100-model chr21 ensemble provides the baseline similarity distribution for equivalence testing) — https://bitbucket.org/3dome/3dgnome
- **Docker** (Containerized build/run environment for cudaMMC ('sudo docker build -t cudammc') so both method runs are reproducible)
- **CMake** (Generates the makefile for native compilation of cudaMMC with -DCUDA_ARCH set to the GPU compute capability (>=60))

## Examples

```
./build/cudaMMC -s config.ini -c chr21 -o ~/chiapet/chr21/ -n chr21_cudammc
```

## Evaluation signals

- The two within-ensemble density distributions visually overlap, reproducing Supplementary Fig. S3
- Medians of the cudaMMC and 3D-GNOME similarity distributions differ by less than 1%
- Each ensemble contains 100 models built with equivalent settings and the same target (chr21)
- Similarity matrices are complete 100x100 pairwise comparisons within each ensemble
- The cudaMMC binary runs correctly (prints its usage message after compilation), confirming the environment was built properly

## Limitations

- Equivalence is demonstrated for chr21 with GM12878 CTCF ChIA-PET inputs; other loci or factors require re-running the full 100-vs-100 ensemble comparison
- Similarity of distributions does not certify per-model accuracy, only that the stochastic output of the new method matches the reference at ensemble level
- Comparison requires matching settings between cudaMMC and 3D-GNOME runs; divergent parameters (e.g. freq_dist_power, use_motif_orientation) would confound the distributions

## Evidence

- [abstract] GPU-enhanced successor preserves ensemble-level model quality: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [intro] cudaMMC is the GPU-enhanced version of 3D-GNOME: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [methods] Concrete simulation invocation used to generate each ensemble member: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [readme] Compilation sanity check before generating ensembles: "You should see the usage message printed in the terminal."
