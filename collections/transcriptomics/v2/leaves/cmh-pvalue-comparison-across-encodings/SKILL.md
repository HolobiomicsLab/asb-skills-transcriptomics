---
name: cmh-pvalue-comparison-across-encodings
description: Use when genotype data may have been binarized with the wrong encoding
  and you must decide whether HOGImine's additive-encoding mode better recovers the
  planted phenotype association than binary-encoding mode.
when_to_use_negative:
- The correct binarization is already known and matches the phenotype's encoding mechanism
  — just run the matching binary-encoding mode directly.
- Markers are natively additive-encoded (0/1/2 allele counts) with no binarization
  ambiguity — the additive-mode run alone is sufficient; no cross-encoding comparison
  is needed.
- The goal is significant-interval discovery with covariate conditioning in FastCMH's
  CMH framework on purely binary features rather than encoding-mode comparison in
  HOGImine.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0654
tools:
- name: HOGImine
  role: Pattern-mining algorithm run in both additive- and binary-encoding modes to
    obtain CMH p-values for genetic meta-markers under each encoding
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: The compiled additive-mode executable invoked on both the additive and the
    dominant-binarized marker matrices with -f 0.05 and -O verbose output
  repo: https://github.com/BorgwardtLab/HOGImine
- name: gcc-12
  role: Compiler used to build HOGImine from source before running the encoding comparison
- name: make
  role: Build system for compiling HOGImine (cd src; make)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:b8df149013dfe8899e30659045e06ff76985976800187cf5730c8a7b20a0acc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  - gcc-12
  - make
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cmh-pvalue-comparison-across-encodings

## Summary

A benchmarking skill for HOGImine that quantifies the loss of statistical power caused by applying the wrong binary encoding (e.g., dominant) to data whose phenotype signal follows another encoding (e.g., recessive), by comparing significant-pattern counts and smallest CMH p-values between binary-encoding and additive-encoding runs at k=1,2.

## When to use

Use when genotype data may have been binarized with the wrong encoding and you must decide whether HOGImine's additive-encoding mode better recovers the planted phenotype association than binary-encoding mode.

## When NOT to use

- The correct binarization is already known and matches the phenotype's encoding mechanism — just run the matching binary-encoding mode directly.
- Markers are natively additive-encoded (0/1/2 allele counts) with no binarization ambiguity — the additive-mode run alone is sufficient; no cross-encoding comparison is needed.
- The goal is significant-interval discovery with covariate conditioning in FastCMH's CMH framework on purely binary features rather than encoding-mode comparison in HOGImine.

## Inputs

- Synthetic additive-encoded marker file (0/1/2 allele counts, matrix of size n_SNPs x n_samples, columns are samples, HOGImine marker-file format)
- Dominant-encoding binarization of the same marker matrix (mis-specified binary input)
- Binary label/phenotype vector whose association signal follows the recessive encoding
- SNP-name file, covariate file, SNP-to-gene mapping file, and gene-interaction prior file (e.g., genes+edges+3cliques+complexes format)

## Outputs

- encoding_comparison.csv tabulating significant-pattern counts and smallest CMH p-values for binary vs additive mode at k=1 and k=2
- Verbose HOGImine output rows per run (CMH p-value, genes with encoding annotation, SNP positions)

## How to apply

Generate synthetic additive-encoded genotype data (0/1/2 allele counts) in HOGImine's n_SNPs x n_samples marker format where the binary label vector's association signal follows the recessive encoding, then binarize a copy using the dominant encoding to create the mis-specified input. Compile HOGImine from source (cd src; make, gcc-12 toolchain). Run ./src/hogimine_additive on both the dominant-binarized matrix and the original additive matrix with -f 0.05 and verbose output (-O) for k=1 and k=2. Parse each run's output rows (CMH p-value, genes with encoding annotation, SNP positions) to count significant patterns and extract the smallest CMH p-value per run, holding the target FWER at 0.05. The additive mode is judged preferable if it recovers more significant patterns and a smaller smallest CMH p-value at each k, since wrongly binarized markers retain only a fraction of their association with the phenotype.

## Related tools

- **HOGImine** (Pattern-mining algorithm run in both additive- and binary-encoding modes to obtain CMH p-values for genetic meta-markers under each encoding) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (The compiled additive-mode executable invoked on both the additive and the dominant-binarized marker matrices with -f 0.05 and -O verbose output) — https://github.com/BorgwardtLab/HOGImine
- **gcc-12** (Compiler used to build HOGImine from source before running the encoding comparison)
- **make** (Build system for compiling HOGImine (cd src; make))

## Examples

```
cd src && make && ./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt -f 0.05 -O
```

## Evaluation signals

- At each k (1 and 2), the additive-encoding run on the original additive data yields more significant patterns than the run on dominant-binarized data, reflecting the recessive signal planted in the labels.
- The smallest CMH p-value from the additive-mode run on the original data is smaller than the smallest CMH p-value from the mis-specified dominant-binarized run at both k=1 and k=2.
- All runs hold the target FWER at 0.05 (configured via -f 0.05), so the comparison is power-based rather than a false-positive artifact.
- HOGImine compiles cleanly with make under gcc-12 and each run produces parseable verbose (-O) output rows containing CMH p-value, gene encoding annotation, and SNP positions.
- Tabulated results in encoding_comparison.csv cover all four cells (binary/additive x k=1/k=2) with consistent run configuration.

## Limitations

- The conclusion is specific to the synthetic scenario where the phenotype signal follows the recessive encoding but data are binarized with the dominant encoding; other encoding mismatches are not evaluated here.
- The result motivates a preference, not a guarantee: when the correct binarization is unknown, additive encoding is preferred, but it may not dominate under every encoding-mis-specification scenario.
- The article notes 'Currently the code is compiled with gcc-12, but other compilers might work as well', so compilation failures or behavioral differences may arise with other toolchains.

## Evidence

- [intro] HOGImine accepts both binary encodings and additive encodings for the markers.: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [methods] ./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt: "./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt"
- [readme] Compilation uses make. Currently the code is compiled with gcc-12, but other compilers might work as well.: "Compilation uses make. Currently the code is compiled with gcc-12```, but other compilers might work as well."
- [methods] output file name, with -o in compressed format and with -O in verbose format.: "output file name, with -o in compressed format and with -O in verbose format."
- [abstract] the algorithm has a substantially higher statistical power compared to previous methods: "Our experimental evaluation shows that the algorithm has a substantially higher statistical power compared to previous methods"
