---
name: statistical-power-comparison-across-methods
description: Use when you need to quantify whether enlarging the class of discoverable
  meta-markers (higher-order gene interactions, k>=2) yields more significant patterns
  than single/low-order miners under the same FWER criterion.
when_to_use_negative:
- Your goal is discovery on a single real dataset rather than a controlled power comparison
  against baselines — the benchmark design requires synthetic data with known association
  strength rho.
- 'Your marker data are additive-encoded (e.g. 0/1/2 dosages) for the FastCMH baseline:
  FastCMH expects L rows x n columns of strictly binary 0/1 features, so only the
  binary-encoded arm of the comparison is valid there.'
- You cannot guarantee an identical FWER criterion across methods — comparing raw
  pattern counts when one method uses a default other than FWER 0.05 (SiNIMin default
  0.05) invalidates the power comparison.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0203
tools:
- name: HOGImine
  role: Method under test; enumerates testable higher-order (k>=2) genetic meta-markers
    with FWER control (-f 0.05) and verbose output (-O, -v)
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: HOGImine binary variant used to run the miner on binary or additive-encoded
    marker matrices
  repo: https://github.com/BorgwardtLab/HOGImine
- name: SiNIMin
  role: State-of-the-art baseline miner run on identical synthetic datasets under
    the same FWER 0.05 criterion
  repo: BorgwardtLab/SiNIMin
- name: FastCMH
  role: State-of-the-art significant-interval baseline (C binary or R package fastcmh)
    run on identical synthetic data at alpha=0.05
  repo: BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: make
  role: Builds HOGImine, SiNIMin and FastCMH C implementations prior to benchmarking
- name: gcc-12
  role: Default compiler toolchain for compiling HOGImine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:4b71222731bb9b555a8778f1cdb262207c8bf049997c7317721f33632f0419ab
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# statistical-power-comparison-across-methods

## Summary

Benchmarks the statistical power of HOGImine (k>=2 higher-order genetic interaction mining) against SiNIMin and FastCMH by counting significant genetic meta-markers under FWER control at alpha=0.05 across a range of association strengths rho on synthetic binary-encoded genotype data.

## When to use

Use when you need to quantify whether enlarging the class of discoverable meta-markers (higher-order gene interactions, k>=2) yields more significant patterns than state-of-the-art single/low-order miners under the same FWER criterion.

## When NOT to use

- Your goal is discovery on a single real dataset rather than a controlled power comparison against baselines — the benchmark design requires synthetic data with known association strength rho.
- Your marker data are additive-encoded (e.g. 0/1/2 dosages) for the FastCMH baseline: FastCMH expects L rows x n columns of strictly binary 0/1 features, so only the binary-encoded arm of the comparison is valid there.
- You cannot guarantee an identical FWER criterion across methods — comparing raw pattern counts when one method uses a default other than FWER 0.05 (SiNIMin default 0.05) invalidates the power comparison.

## Inputs

- Synthetic binary-encoded marker matrix (n.SNPs x n.samples, each column a sample)
- Binary phenotype label vector/matrix (n.samples rows or columns)
- Gene-interaction priors file (e.g. genes+edges+3cliques+complexes.txt, all_int_compl.txt) for HOGImine -e
- Target FWER value (alpha=0.05) applied identically to HOGImine, SiNIMin and FastCMH
- Optionally for FastCMH: covariate file (cov.txt) with K class counts; for SiNIMin: edge and feature-to-node mapping files

## Outputs

- Per-method, per-rho counts of statistically significant patterns
- Smallest CMH p-value among testable patterns per method and rho (first column of each pattern output row)
- benchmark_results.csv aggregating pattern counts and minimum p-values
- Power-gain curves (as in Figure 2) showing HOGImine vs SiNIMin and FastCMH

## How to apply

Compile HOGImine from source with make (gcc-12 default toolchain), and build the SiNIMin (make in SiNIMin/C, linking Eigen) and FastCMH (make in C folder) baselines from their repositories. Generate synthetic binary-encoded datasets — marker matrices of size n.SNPs x n.samples with a binary label vector — spanning a range of association strengths rho. Run HOGImine on each dataset with the gene-interaction priors file (-e), target FWER -f 0.05, verbose output -O, and -v to enumerate all testable patterns, varying interaction order from k=2 upward; run SiNIMin and FastCMH on the identical datasets under the same FWER 0.05 criterion. For each method and rho, count significant patterns and extract the smallest CMH p-value among testable patterns from the first column of each pattern output row, then aggregate per-method, per-rho counts and minimum p-values into benchmark_results.csv to plot power-gain curves. Correct application is judged by HOGImine discovering more significant patterns than both baselines at matched rho, even though the significance threshold becomes more stringent as k increases.

## Related tools

- **HOGImine** (Method under test; enumerates testable higher-order (k>=2) genetic meta-markers with FWER control (-f 0.05) and verbose output (-O, -v)) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (HOGImine binary variant used to run the miner on binary or additive-encoded marker matrices) — https://github.com/BorgwardtLab/HOGImine
- **SiNIMin** (State-of-the-art baseline miner run on identical synthetic datasets under the same FWER 0.05 criterion) — BorgwardtLab/SiNIMin
- **FastCMH** (State-of-the-art significant-interval baseline (C binary or R package fastcmh) run on identical synthetic data at alpha=0.05) — BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **make** (Builds HOGImine, SiNIMin and FastCMH C implementations prior to benchmarking)
- **gcc-12** (Default compiler toolchain for compiling HOGImine)

## Examples

```
./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt -e data/mmusculus/BMC.Mode/all_int_compl.txt -f 0.05 -O -v
```

## Evaluation signals

- benchmark_results.csv exists with one row per (method, rho) containing pattern counts and minimum CMH p-values extracted from the first column of pattern output rows.
- For matched rho and FWER 0.05, HOGImine's significant-pattern count is higher than both SiNIMin's and FastCMH's, reproducing the power-gain trend of Figure 2.
- All three methods were run with the same target FWER (alpha=0.05); any count comparison made under differing alpha should be flagged.
- The minimum reported CMH p-value per method decreases with increasing association strength rho on synthetic data.
- Pattern counts remain non-degenerate even as interaction order k increases, despite the more stringent significance threshold at larger k.

## Limitations

- The significance threshold decreases with larger interaction order k, so counts at higher k reflect a harsher null distribution rather than a uniform test.
- FastCMH operates on strictly binary-encoded intervals and requires covariate-ordered data files (data.txt, label.txt, cov.txt with rows ordered by covariate class), so its comparison arm is limited to binary encodings and K=2 for the provided synthetic-data generator.
- Compilation is compiler-sensitive: HOGImine is tested with gcc-12 (others 'might work'), and SiNIMin depends on linking the Eigen library, so build failures may require adjusting Makefile compiler settings.

## Evidence

- [abstract] HOGImine's higher statistical power vs previous methods: "Our experimental evaluation shows that the algorithm has a substantially higher statistical power compared to previous methods, allowing it to discover genetic"
- [methods] HOGImine binary-data run command: "./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt"
- [methods] HOGImine compilation toolchain: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [readme] SiNIMin FWER default matches benchmark alpha: [evidence span withheld — anchor status: not_found]
- [readme] FastCMH baseline invocation via R package: [evidence span withheld — anchor status: external_doc]
