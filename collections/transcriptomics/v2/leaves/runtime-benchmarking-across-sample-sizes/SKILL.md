---
name: runtime-benchmarking-across-sample-sizes
description: Use when comparing the wall-clock runtime of significant genetic meta-marker
  miners (HOGImine, SiNIMin, FastCMH) across subsampled biobank sizes, or when validating
  that a faster miner outputs the same patterns as a baseline.
when_to_use_negative:
- The goal is statistical power comparison rather than runtime — use power/association
  evaluations instead.
- HOGImine is run with additive-encoded markers or p>0 permutations, which adds permutation-testing
  overhead absent from the pure runtime benchmark.
- Only one (n, l) configuration is needed; the benchmark's value comes from systematically
  varying sample count and SNPs-per-gene.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: HOGImine
  role: Tool being benchmarked; run with binary encoding at k=1 and k=2 on subsampled
    data, timing each run
  repo: https://github.com/BorgwardtLab/HOGImine
- name: SiNIMin
  role: Baseline significant network interval miner run on identical inputs for the
    same-pattern-set runtime comparison
  repo: https://github.com/BorgwardtLab/SiNIMin
- name: FastCMH
  role: Baseline significant interval search tool run on identical inputs as an additional
    runtime reference
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: gcc-12
  role: Compiler used to build HOGImine before benchmarking
- name: make
  role: Build system for compiling HOGImine, SiNIMin, and FastCMH
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:bb646d8077dc870e680d38bf964303824a54e7c861f38e76871cf4430e45967f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - FastCMH
  - HOGImine
  - SiNIMin
  - gcc-12
  - make
  - unzip
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# runtime-benchmarking-across-sample-sizes

## Summary

Benchmark the running time of HOGImine (binary encoding, k=1 and k=2) against SiNIMin and FastCMH on the same SNP datasets while varying the number of samples n and SNPs per gene l, to quantify HOGImine's one-to-two order-of-magnitude speedup over SiNIMin for the identical pattern set.

## When to use

Use when comparing the wall-clock runtime of significant genetic meta-marker miners (HOGImine, SiNIMin, FastCMH) across subsampled biobank sizes, or when validating that a faster miner outputs the same patterns as a baseline.

## When NOT to use

- The goal is statistical power comparison rather than runtime — use power/association evaluations instead.
- HOGImine is run with additive-encoded markers or p>0 permutations, which adds permutation-testing overhead absent from the pure runtime benchmark.
- Only one (n, l) configuration is needed; the benchmark's value comes from systematically varying sample count and SNPs-per-gene.

## Inputs

- Binary-encoded SNP marker matrix (n.SNPs x n.samples) for HOGImine, e.g. data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt
- Binary phenotype labels file, e.g. avrRpm1_Y.txt
- Data/label/covariate files for FastCMH (data.txt L rows x n columns, label.txt n rows, cov.txt K rows)
- Data, label, feature, edge, and mapping files required by SiNIMin
- SiNIMin and FastCMH baseline tools compiled from the BorgwardtLab repositories

## Outputs

- runtimes.csv with columns tool, encoding, k, n, l
- Per-configuration runtime measurements (seconds to hours) for HOGImine k=1, k=2, SiNIMin, and FastCMH
- Runtime ratio for HOGImine k=2 vs SiNIMin (expected 10x-100x speedup)
- Pattern lists (CMH p-value, genes, SNP positions) from HOGImine k=2 and SiNIMin used for the identity check

## How to apply

Compile HOGImine from source with make under gcc-12 (cd src && make), and build the baseline tools SiNIMin and FastCMH from their BorgwardtLab repositories. Run HOGImine with binary-encoded markers (e.g. on the athaliana avrRpm1 dataset, -f 0.05, default p=0 permutations) at k=1 and k=2, subsampling the data to vary sample count n and SNPs-per-gene l, timing each configuration. Run SiNIMin and FastCMH on the identical inputs for each (n, l) configuration and time each run. Collect all measured runtimes into runtimes.csv with columns tool, encoding, k, n, l. Finally, verify that HOGImine k=2 and SiNIMin produce identical pattern lists (CMH p-value, genes, SNP positions per row) before computing the runtime ratio, which should confirm a one-to-two order-of-magnitude speedup for HOGImine k=2 over SiNIMin thanks to BFS exploration and bitwise support updates.

## Related tools

- **HOGImine** (Tool being benchmarked; run with binary encoding at k=1 and k=2 on subsampled data, timing each run) — https://github.com/BorgwardtLab/HOGImine
- **SiNIMin** (Baseline significant network interval miner run on identical inputs for the same-pattern-set runtime comparison) — https://github.com/BorgwardtLab/SiNIMin
- **FastCMH** (Baseline significant interval search tool run on identical inputs as an additional runtime reference) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **gcc-12** (Compiler used to build HOGImine before benchmarking)
- **make** (Build system for compiling HOGImine, SiNIMin, and FastCMH)

## Examples

```
cd src && make && ./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -f 0.05
```

## Evaluation signals

- HOGImine k=2 and SiNIMin return identical pattern lists (same CMH p-value, genes, and SNP positions per row) on every benchmarked configuration.
- The computed runtime ratio for HOGImine k=2 over SiNIMin falls in the 10x-100x range (one to two orders of magnitude).
- runtimes.csv is complete: one row per tool/encoding/k/n/l combination with no missing timing entries.
- HOGImine completes large-biobank configurations in tens of seconds while SiNIMin requires hours.
- All binaries were built successfully: HOGImine with gcc-12/make, FastCMH via its C make target, SiNIMin via its Makefile (Eigen-linked).

## Limitations

- The identical pattern set is only guaranteed for the HOGImine k=2 vs SiNIMin comparison (k=2 recovers the same pattern set); k=1 and FastCMH mine different pattern classes.
- Subsampling is required to vary n and l, so runtimes reflect subsampled configurations rather than the full biobank scale for small settings.
- SiNIMin compilation may require adjusting the Makefile's Eigen and OpenMP linkage; FastCMH's C code additionally needs Python and numpy for result post-processing.

## Evidence

- [results] HOGImine with k=2 achieves roughly one to two orders of magnitude speedup over SiNIMin (same pattern set) thanks to BFS exploration and bitwise support updates, mining large biobanks in tens of seconds instead of hours.: [evidence span withheld — anchor status: not_found]
- [methods] Compilation uses make. Currently the code is compiled with gcc-12, but other compilers might work as well.: "Compilation uses make. Currently the code is compiled with gcc-12, but other compilers might work as well."
- [abstract] We also develop a more efficient search strategy and support computation to make our approach applicable in practice, leading to substantial runtime improvements compared to state-of-the-art methods.: "we also develop a more efficient search strategy and support computation to make our approach applicable in practice, leading to substantial runtime improvements compared to state-of-the-art methods"
- [readme] To compile the C version of the code, enter the C folder and run make.: "To compile the C version of the code, enter the `C` folder and run `make`:


cd C
make
"
- [readme] SiNIMin package relies on the Eigen-library and OpenMP; its Makefile may have to be adjusted for compilation.: [evidence span withheld — anchor status: external_doc]
