---
name: cpp-branch-and-bound-threshold-modification
description: Use when mining significant genetic meta-markers with HOGImine and the
  |T|·δtar Tarone bound is too conservative — replace the threshold update with a
  Westfall–Young FWER estimate from pre-computed label permutations.
when_to_use_negative:
- 'Running the default HOGImine as-is (-p 0): the Tarone-style |T|·δtar bound is the
  intended behavior and no Westfall–Young estimator is computed without modifications
  and a positive -p value'
- Downstream analysis tasks on already-produced output files (filtering or interpreting
  significant meta-markers) — this skill is about the C++ search/threshold code path,
  not post-processing
- 'Binary interval search with FastCMH or network interval mining with SiNIMin: their
  Westfall–Young implementations live in separate codebases (SiNIMin-WY) and cannot
  reuse the HOGImine branch-and-bound modification directly'
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HOGImine (hogimine_additive)
  role: C++ codebase whose branch-and-bound threshold update is modified to swap the
    |T|·δtar bound for the FWER^wy Westfall–Young estimator
  repo: https://github.com/BorgwardtLab/HOGImine
- name: make
  role: Builds the modified HOGImine source in the src directory
- name: gcc-12
  role: Compiler used to compile the modified HOGImine C++ code
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:84fe4deaecb329c06bc20a5a92a21aa95a1e0baafe11876b9b0c7c2e0dbb4cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  - gcc-12
  - make
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cpp-branch-and-bound-threshold-modification

## Summary

Modifies HOGImine's C++ branch-and-bound significance-threshold update from the Tarone-style |T|·δtar FWER upper bound to a Westfall–Young permutation-based FWER estimator, computed from the running minimum CMH p-values over K label permutations.

## When to use

Use when mining significant genetic meta-markers with HOGImine and the |T|·δtar Tarone bound is too conservative — replace the threshold update with a Westfall–Young FWER estimate from pre-computed label permutations.

## When NOT to use

- Running the default HOGImine as-is (-p 0): the Tarone-style |T|·δtar bound is the intended behavior and no Westfall–Young estimator is computed without modifications and a positive -p value
- Downstream analysis tasks on already-produced output files (filtering or interpreting significant meta-markers) — this skill is about the C++ search/threshold code path, not post-processing
- Binary interval search with FastCMH or network interval mining with SiNIMin: their Westfall–Young implementations live in separate codebases (SiNIMin-WY) and cannot reuse the HOGImine branch-and-bound modification directly

## Inputs

- Marker file (-i): SNPs x samples matrix with binary or additive encoding (e.g. avrRpm1_X.txt, or data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt)
- Binary phenotype labels file (-l), e.g. avrRpm1_Y.txt or BMC.Mode_y_bin.txt
- Optional categorical covariate file (-c), e.g. avrRpm1_covar_n2.txt
- SNP ID file (-s) and SNP-to-gene mapping file (-m), e.g. avrRpm1_snp_map.txt
- Network/priors file (-e) such as genes+edges+3cliques+complexes.txt or all_int_compl.txt
- Target FWER (-f, e.g. 0.05) and number of permutations (-p > 0)

## Outputs

- Verbose output file (-O, e.g. out_athaliana) with one row per significant pattern containing the CMH p-value, interacting genes, and SNP positions, all passing the Westfall–Young-estimated significance threshold at level 0.05
- Updated in-memory per-permutation minimum p-value statistics (FWER^wy) used for branch-and-bound pruning

## How to apply

Compile the HOGImine C++ source with make (gcc-12) in the src directory. Before the search starts, pre-compute K label permutations once (controlled by the -p option, default 0 meaning permutation testing is off). During the branch-and-bound mining loop, compute each candidate pattern's CMH p-value on the true labels and update, for every permutation k, the running minimum p-value of that permutation over all processed patterns. Replace the significance-threshold-decreasing step so that instead of lowering δtar to keep the |T|·δtar FWER bound under the target level α (e.g. -f 0.05), the cutoff is decreased using the Westfall–Young FWER estimator FWER^wy derived from the per-permutation minima. Run the modified binary on a dataset such as data/athaliana/interactome_0kb/avrRpm1 with the -f 0.05 FWER level, a positive -p value, and -O for verbose output.

## Related tools

- **HOGImine (hogimine_additive)** (C++ codebase whose branch-and-bound threshold update is modified to swap the |T|·δtar bound for the FWER^wy Westfall–Young estimator) — https://github.com/BorgwardtLab/HOGImine
- **make** (Builds the modified HOGImine source in the src directory)
- **gcc-12** (Compiler used to compile the modified HOGImine C++ code)

## Examples

```
cd src && make && ./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -c avrRpm1_covar_n2.txt -f 0.05 -p 100 -O out_athaliana
```

## Evaluation signals

- The run terminates normally and out_athaliana (verbose -O format) contains one row per significant pattern with CMH p-value, interacting genes, and SNP positions
- Every reported pattern's CMH p-value passes the FWER-estimated significance threshold at level -f 0.05
- With -p > 0 the effective significance cutoff is decreased based on the per-permutation minimum p-values, not the |T|·δtar bound — verified by comparing reported patterns against the unmodified (-p 0) Tarone-thresholded run
- The binary compiles cleanly with make under gcc-12 after the threshold-update code is changed
- The number of significant patterns returned should be ≥ the unmodified Tarone-bound version if FWER^wy is tighter, reflecting HOGImine's claimed higher statistical power

## Limitations

- Permutation testing requires a positive -p value (default 0 disables it); the number of permutations K trades FWER^wy accuracy against runtime, since min-p statistics must be updated for every permutation for every candidate pattern
- The modification assumes CMH p-values conditioned on the categorical covariate file (-c) and connected gene subgraphs from the network priors file (-e); without network priors the search space is not restricted
- Compilation is documented only for gcc-12 via make; other compilers 'might work as well' but are untested

## Evidence

- [methods] Compilation uses make with gcc-12: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [methods] -p option triggers permutation testing: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [intro] HOGImine uses CMH testing and connected gene subgraphs as priors: "our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways and protein complexes, to restrict its search space"
- [methods] Output format controlled by -o/-O: "output file name, with -o in compressed format and with -O in verbose format."
- [abstract] HOGImine has higher statistical power than previous methods: "Our experimental evaluation shows that the algorithm has a substantially higher statistical power compared to previous methods"
- [methods] Article run command on athaliana avrRpm1 data: "-i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt"
