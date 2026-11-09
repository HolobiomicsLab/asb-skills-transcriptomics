---
name: snp-dataset-subsampling
description: Use when the goal is to reproduce or extend runtime benchmarks of SNP
  pattern miners — i.e. measuring how running time scales with number of samples n
  and SNPs per gene l across HOGImine (k=1, k=2), SiNIMin, and FastCMH.
when_to_use_negative:
- The goal is biological discovery rather than runtime benchmarking — then run HOGImine
  on the full dataset with permutations (p > 0) instead of timing subsampled runs
- The pattern output of the two tools is the deliverable; subsampling only controls
  (n, l) for scaling experiments and does not by itself validate statistical power
- Data uses additive encoding or requires covariate conditioning — subsample construction
  must then respect the FastCMH covariate file format (cov.txt) and the SiNIMin mapping/edge
  files
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: HOGImine
  role: Tool under benchmark; run at k=1 and k=2 with binary encoding on subsampled
    data, timing each run
  repo: https://github.com/BorgwardtLab/HOGImine
- name: SiNIMin
  role: Baseline miner run on identical subsampled inputs; HOGImine k=2 must return
    the same pattern set
  repo: https://github.com/BorgwardtLab/SiNIMin
- name: FastCMH
  role: Second baseline (significant interval search with CMH test) timed on the same
    (n, l) configurations
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: gcc-12
  role: Compiler used to build HOGImine before benchmarking
- name: make
  role: Build entry point for HOGImine (cd src && make) and for the FastCMH/SiNIMin
    C code
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:9757b4df0f88edf24d786ca7256b9e658f155edda6a63f2015c370e4711e26d4
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# snp-dataset-subsampling

## Summary

Runtime benchmarking skill for SNP meta-marker mining: subsample a biobank (e.g. the athaliana avrRpm1 dataset) to vary sample count n and SNPs-per-gene l, then time HOGImine, SiNIMin, and FastCMH on identical inputs to reproduce the Figure 4 speedup comparison.

## When to use

Use when the goal is to reproduce or extend runtime benchmarks of SNP pattern miners — i.e. measuring how running time scales with number of samples n and SNPs per gene l across HOGImine (k=1, k=2), SiNIMin, and FastCMH.

## When NOT to use

- The goal is biological discovery rather than runtime benchmarking — then run HOGImine on the full dataset with permutations (p > 0) instead of timing subsampled runs
- The pattern output of the two tools is the deliverable; subsampling only controls (n, l) for scaling experiments and does not by itself validate statistical power
- Data uses additive encoding or requires covariate conditioning — subsample construction must then respect the FastCMH covariate file format (cov.txt) and the SiNIMin mapping/edge files

## Inputs

- SNP marker matrix (SNPs x samples, binary encoding), e.g. data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt
- Binary phenotype labels file, e.g. avrRpm1_Y.txt
- Baseline tools: SiNIMin and FastCMH clones from BorgwardtLab repositories
- Subsampling grid of (n, l) configurations derived from the original dataset

## Outputs

- runtimes.csv with columns tool, encoding, k, n, l holding measured wall-clock times
- Pattern lists (CMH p-value, genes, SNP positions per row) from HOGImine k=2 and SiNIMin on identical inputs
- Computed runtime ratio HOGImine k=2 / SiNIMin per (n, l) configuration

## How to apply

Compile HOGImine from source (cd src && make, gcc-12) and prepare the baselines SiNIMin and FastCMH from their BorgwardtLab repositories. Subsample the SNP dataset (e.g. data/athaliana/interactome_0kb/avrRpm1) along two axes: number of samples n and number of SNPs per gene l, creating one dataset per (n, l) configuration. Run HOGImine with binary-encoded markers (e.g. with -f 0.05 and default p=0 permutations) at k=1 and k=2, and run SiNIMin and FastCMH on the identical subsampled inputs, timing every run. Write all measured runtimes into runtimes.csv with columns tool, encoding, k, n, l. As the fairness check, verify that HOGImine at k=2 and SiNIMin return the identical pattern set (same CMH p-values, genes, and SNP positions per row), then compute the runtime ratio: the expected result is a one-to-two order-of-magnitude speedup for HOGImine k=2 over SiNIMin, with large biobanks mined in tens of seconds instead of hours.

## Related tools

- **HOGImine** (Tool under benchmark; run at k=1 and k=2 with binary encoding on subsampled data, timing each run) — https://github.com/BorgwardtLab/HOGImine
- **SiNIMin** (Baseline miner run on identical subsampled inputs; HOGImine k=2 must return the same pattern set) — https://github.com/BorgwardtLab/SiNIMin
- **FastCMH** (Second baseline (significant interval search with CMH test) timed on the same (n, l) configurations) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **gcc-12** (Compiler used to build HOGImine before benchmarking)
- **make** (Build entry point for HOGImine (cd src && make) and for the FastCMH/SiNIMin C code)

## Examples

```
cd src && make && ./hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -f 0.05
```

## Evaluation signals

- runtimes.csv exists and its rows follow the schema (tool, encoding, k, n, l)
- Pattern lists produced by HOGImine k=2 and SiNIMin on the identical input are byte-equivalent: same CMH p-values, same genes, same SNP positions per row
- Computed runtime ratio HOGImine k=2 vs SiNIMin falls in the one-to-two orders-of-magnitude range (roughly 10x-100x)
- Runtime trend is monotone in n and l, and the same (n, l) configs are covered by all three tools
- All three tools run at matching FWER/threshold settings (e.g. -f 0.05, default p=0 permutations) so timing is apples-to-apples

## Limitations

- The speedup claim holds for the identical pattern set at k=2 vs SiNIMin; HOGImine at other k values explores different pattern spaces and the ratio is not directly comparable
- FastCMH's C code requires Python and numpy for postprocessing/filtering of final results, which must be installed before benchmark runs
- SiNIMin's Makefile may need compiler adjustments (Eigen, OpenMP) — a failed compile invalidates the benchmark until resolved

## Evidence

- [methods] Compile HOGImine before benchmarking: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [abstract] Runtime improvement is the paper's headline benchmark result: "leading to substantial runtime improvements compared to state-of-the-art methods."
- [readme] HOGImine input format for the binary-encoded benchmark runs: "- ```-i file```: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$"
- [methods] Example runtime run on the athaliana avrRpm1 dataset: "./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt"
- [readme] Data decompression before running: "unzip data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt.zip -d data/mmusculus/BMC.Mode/"
