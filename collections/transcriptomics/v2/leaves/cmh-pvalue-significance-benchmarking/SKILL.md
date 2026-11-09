---
name: cmh-pvalue-significance-benchmarking
description: Use when comparing the statistical power of significant-pattern mining
  methods (e.g. HOGImine vs SiNIMin vs FastCMH) on synthetic binary genotype data,
  where the goal is to count significant patterns or extract minimum CMH p-values
  per method under a fixed FWER such as 0.05.
when_to_use_negative:
- The goal is discovering associations in real biological datasets with known covariate
  structure (e.g. FastCMH's categorical covariate conditioning) rather than benchmarking
  statistical power on synthetic data.
- The markers are additive-encoded rather than binary, and the question is not a power
  comparison of binary-encoded synthetic benchmarks.
- Only a single method needs to be run for discovery, with no comparison against SiNIMin
  or FastCMH baselines required.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: HOGImine
  role: Primary miner run on synthetic datasets with -e priors, -f 0.05 FWER, -O verbose
    and -v all-testable-patterns flags, varying interaction order k from 2 upward
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: Compiled executable variant of HOGImine produced by make in the src directory
  repo: https://github.com/BorgwardtLab/HOGImine
- name: SiNIMin
  role: State-of-the-art baseline run on identical synthetic datasets under the same
    FWER 0.05 criterion to compare significant-pattern counts
  repo: https://github.com/BorgwardtLab/SiNIMin
- name: FastCMH
  role: State-of-the-art baseline run on identical synthetic datasets under the same
    FWER 0.05 criterion; its C build uses make and outputs allpval.txt plus significant
    intervals
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: make
  role: Compiles HOGImine (cd src; make) and the FastCMH C code (cd C; make)
- name: gcc-12
  role: Compiler toolchain used to build HOGImine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:61266678d18affa2036cbc1399787a20eab561f6653b3705194ca99adf543031
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - FastCMH
  - HOGImine
  - HOGImine (hogimine_additive)
  - SiNIMin
  - gcc-12
  - make
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cmh-pvalue-significance-benchmarking

## Summary

Benchmarks statistical power of genetic meta-marker discovery tools (HOGImine, SiNIMin, FastCMH) by counting significant patterns under FWER control at alpha=0.05 on synthetic binary-encoded genotype/label datasets, comparing how higher-order gene interactions (k>=2) yield more discoveries than pairwise-baseline miners.

## When to use

Use when comparing the statistical power of significant-pattern mining methods (e.g. HOGImine vs SiNIMin vs FastCMH) on synthetic binary genotype data, where the goal is to count significant patterns or extract minimum CMH p-values per method under a fixed FWER such as 0.05.

## When NOT to use

- The goal is discovering associations in real biological datasets with known covariate structure (e.g. FastCMH's categorical covariate conditioning) rather than benchmarking statistical power on synthetic data.
- The markers are additive-encoded rather than binary, and the question is not a power comparison of binary-encoded synthetic benchmarks.
- Only a single method needs to be run for discovery, with no comparison against SiNIMin or FastCMH baselines required.

## Inputs

- Synthetic binary-encoded genotype dataset: marker matrix of size n.SNPs x n.samples with a binary label vector, spanning a range of association strengths rho
- Gene-interaction priors file (e.g. genes+edges+3cliques+complexes.txt)
- HOGImine source compiled with make (gcc-12)
- SiNIMin and FastCMH baseline binaries (C code compiled with make)

## Outputs

- Per-method, per-rho counts of statistically significant patterns under FWER 0.05
- Smallest CMH p-value among testable patterns per method and rho (first column of pattern output rows)
- benchmark_results.csv aggregating pattern counts and minimum p-values across methods and rho values
- Power-gain curves comparing higher-order interactions (k>=2) against SiNIMin and FastCMH baselines

## How to apply

Generate synthetic binary-encoded genotype datasets (marker matrix of size n.SNPs x n.samples with a binary label vector) spanning a range of association strengths rho. Compile HOGImine with make under gcc-12, then run it on each dataset with the gene-interaction priors file (-e), target FWER -f 0.05, verbose output -O, and -v to enumerate all testable patterns, varying the interaction order from k=2 upward. Run SiNIMin and FastCMH on the identical datasets under the same FWER 0.05 criterion. For each method and rho, count significant patterns and extract the smallest CMH p-value among testable patterns from the first column of each pattern output row; note that the significance threshold decreases with larger interaction order k. Aggregate per-method, per-rho pattern counts and minimum p-values into benchmark_results.csv and compare power-gain curves across methods.

## Related tools

- **HOGImine** (Primary miner run on synthetic datasets with -e priors, -f 0.05 FWER, -O verbose and -v all-testable-patterns flags, varying interaction order k from 2 upward) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (Compiled executable variant of HOGImine produced by make in the src directory) — https://github.com/BorgwardtLab/HOGImine
- **SiNIMin** (State-of-the-art baseline run on identical synthetic datasets under the same FWER 0.05 criterion to compare significant-pattern counts) — https://github.com/BorgwardtLab/SiNIMin
- **FastCMH** (State-of-the-art baseline run on identical synthetic datasets under the same FWER 0.05 criterion; its C build uses make and outputs allpval.txt plus significant intervals) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **make** (Compiles HOGImine (cd src; make) and the FastCMH C code (cd C; make))
- **gcc-12** (Compiler toolchain used to build HOGImine)

## Examples

```
./src/hogimine_additive -i synthetic_X.txt -l synthetic_y_bin.txt -e priors.txt -f 0.05 -O -v
```

## Evaluation signals

- All methods are run on the identical synthetic datasets under the same target FWER (-f 0.05 for HOGImine; alpha=0.05 for FastCMH), so pattern counts are directly comparable.
- Per-method pattern counts and minimum CMH p-values are recorded for every association strength rho, enabling a monotone power-gain curve check (counts should rise with stronger association).
- The benchmark expectation is that enlarging the meta-marker class to higher-order interactions (k>=2) increases discovered significant patterns relative to SiNIMin and FastCMH, even though the significance threshold decreases with larger k.
- benchmark_results.csv must contain one row per method x rho combination with pattern count and smallest CMH p-value from the first column of the pattern output rows.
- HOGImine verbose (-O) output enumerates all testable patterns, allowing verification that no testable pattern was missed during counting.

## Limitations

- The significance threshold decreases with larger interaction order k, so raw pattern counts at higher k are computed under a stricter per-test criterion.
- FastCMH requires a categorical covariate file (cov.txt) with rows ordered by covariate class, and its R script makefastcmhdata only generates synthetic data for K=2 covariate classes.
- HOGImine compilation is currently only validated with gcc-12; other compilers might work but are not guaranteed.
- SiNIMin requires an Eigen-linked Makefile and network edge/mapping files, so data must follow its strict binary feature-ordering format.

## Evidence

- [results] On synthetic binary-encoded data, enlarging the class of meta-markers to higher-order interactions increases the number of discovered significant patterns relative to SiNIMin and FastCMH: [evidence span withheld — anchor status: not_found]
- [methods] Run HOGImine on each synthetic dataset with the gene-interaction priors file (-e), target FWER -f 0.05, verbose output -O, and -v to enumerate all testable patterns: [evidence span withheld — anchor status: not_found]
- [readme] Compilation uses make. Currently the code is compiled with gcc-12: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [abstract] HOGImine has substantially higher statistical power compared to previous methods: "Our experimental evaluation shows that the algorithm has a substantially higher statistical power compared to previous methods, allowing it to discover genetic mutations statistically associated with"
