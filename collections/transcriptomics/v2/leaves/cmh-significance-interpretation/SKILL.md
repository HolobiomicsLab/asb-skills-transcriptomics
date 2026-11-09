---
name: cmh-significance-interpretation
description: Use when comparing significant meta-marker/interval outputs from CMH-based
  miners (HOGImine, SiNIMin, FastCMH) on the same binary-encoded biobank, or benchmarking
  their runtimes across sample counts n and SNPs-per-gene l.
when_to_use_negative:
- The goal is additive-encoded or permutation-tested (SiNIMin-WY / p > 0) results
  rather than binary-encoding runtime and pattern-set comparison
- The analysis requires conditioning on a categorical covariate beyond what CMH handles,
  or FastCMH's synthetic-data generation (makefastcmhdata only supports K=2 covariate
  classes)
- You need to interpret FastCMH's significance intervals in isolation without a comparable
  HOGImine/SiNIMin run, since the skill centers on cross-tool pattern-set equivalence
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: HOGImine
  role: Candidate miner under test; run with binary encoding at k=1 and k=2, timed
    across subsampled (n, l) configurations
  repo: https://github.com/BorgwardtLab/HOGImine
- name: SiNIMin
  role: Baseline miner whose pattern set must be identical to HOGImine k=2; used to
    compute the runtime speedup ratio
  repo: https://github.com/BorgwardtLab/SiNIMin
- name: FastCMH
  role: Additional baseline for runtime comparison on identical binary-encoded inputs
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: gcc-12
  role: Compiler used to build HOGImine (and adaptable for SiNIMin's Makefile) before
    benchmarking
- name: make
  role: Build driver for HOGImine (cd src && make) and FastCMH C code (cd C && make)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:c32cd413ec8ffb47d6848b174ca969648dc24d97cad974e7cfc4dd4d940ad6fd
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cmh-significance-interpretation

## Summary

Interprets and cross-validates CMH-based significant-interval / genetic meta-marker outputs from HOGImine, SiNIMin, and FastCMH, including verifying that HOGImine k=2 and SiNIMin return identical pattern sets and quantifying HOGImine's one-to-two order-of-magnitude runtime speedup.

## When to use

Use when comparing significant meta-marker/interval outputs from CMH-based miners (HOGImine, SiNIMin, FastCMH) on the same binary-encoded biobank, or benchmarking their runtimes across sample counts n and SNPs-per-gene l.

## When NOT to use

- The goal is additive-encoded or permutation-tested (SiNIMin-WY / p > 0) results rather than binary-encoding runtime and pattern-set comparison
- The analysis requires conditioning on a categorical covariate beyond what CMH handles, or FastCMH's synthetic-data generation (makefastcmhdata only supports K=2 covariate classes)
- You need to interpret FastCMH's significance intervals in isolation without a comparable HOGImine/SiNIMin run, since the skill centers on cross-tool pattern-set equivalence

## Inputs

- HOGImine source repository (compiled with make/gcc-12)
- Binary-encoded marker matrix (n. SNPs x n. samples) with labels, e.g. data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt and avrRpm1_Y.txt
- SiNIMin and FastCMH baseline tool repositories with their required data/label/feature/edge/mapping files
- Subsampled dataset configurations varying n (samples) and l (SNPs per gene)

## Outputs

- runtimes.csv with columns tool, encoding, k, n, l
- Pattern lists per tool (CMH p-value, genes, SNP positions per row), in compressed (-o) or verbose (-O) format
- Runtime ratio between HOGImine k=2 and SiNIMin (expected 1-2 orders of magnitude)

## How to apply

Compile HOGImine from source with make under gcc-12 (cd src && make) and prepare the SiNIMin (Eigen-linked Makefile or cmake) and FastCMH baselines from the BorgwardtLab repositories. Run HOGImine with binary-encoded markers on the athaliana avrRpm1 dataset (e.g. -f 0.05, default p=0 permutations) at k=1 and k=2, subsampling the data to vary n and l, and run SiNIMin and FastCMH on the identical (n, l) configurations. Collect all runtimes into runtimes.csv with columns tool, encoding, k, n, l. Because HOGImine k=2 and SiNIMin discover the same class of patterns, diff their output pattern lists row by row (CMH p-value, genes, SNP positions per row) to confirm identical pattern sets; output patterns are captured with -o (compressed) or -O (verbose). The expected result is that HOGImine k=2 matches SiNIMin's pattern set while being roughly one to two orders of magnitude faster (tens of seconds vs hours on large biobanks), thanks to BFS exploration and bitwise support updates.

## Related tools

- **HOGImine** (Candidate miner under test; run with binary encoding at k=1 and k=2, timed across subsampled (n, l) configurations) — https://github.com/BorgwardtLab/HOGImine
- **SiNIMin** (Baseline miner whose pattern set must be identical to HOGImine k=2; used to compute the runtime speedup ratio) — https://github.com/BorgwardtLab/SiNIMin
- **FastCMH** (Additional baseline for runtime comparison on identical binary-encoded inputs) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **gcc-12** (Compiler used to build HOGImine (and adaptable for SiNIMin's Makefile) before benchmarking)
- **make** (Build driver for HOGImine (cd src && make) and FastCMH C code (cd C && make))

## Examples

```
./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -f 0.05
```

## Evaluation signals

- Pattern lists from HOGImine k=2 and SiNIMin are identical row-by-row in CMH p-value, genes, and SNP positions
- Runtime ratio of HOGImine k=2 versus SiNIMin is between 10x and 100x (one to two orders of magnitude), with HOGImine mining large biobanks in tens of seconds
- runtimes.csv exists with the required schema: tool, encoding, k, n, l
- HOGImine was compiled under gcc-12 via make without error, and both tools were run on identical (n, l) subsampled inputs
- FastCMH significant intervals (df$sig in R) are obtained with the same alpha (e.g. 0.05) convention on the shared inputs

## Limitations

- SiNIMin's compilation depends on linking the Eigen library and may require adjusting the Makefile or using cmake; OpenMP is used for parallelizing permutation testing
- The C version of FastCMH needs Python and numpy installed for postprocessing/filtering of final results
- FastCMH requires data.txt/label.txt rows ordered by covariate class, and makefastcmhdata only generates synthetic data for K=2 covariate classes

## Evidence

- [abstract] we also develop a more efficient search strategy and support computation to make our approach applicable in practice: "we also develop a more efficient search strategy and support computation to make our approach applicable in practice"
- [readme] HOGImine is a pattern-mining-based algorithm for finding genetic meta-markers: "is a pattern-mining-based algorithm for finding genetic meta-markers, i.e. combinations of genetic markers, that show a statistical association with a phenotype"
- [methods] HOGImine output options -o compressed and -O verbose: "output file name, with -o in compressed format and with -O in verbose format."
- [methods] HOGImine permutation testing option p (default 0): "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [readme] SiNIMin repository description: [evidence span withheld — anchor status: external_doc]
