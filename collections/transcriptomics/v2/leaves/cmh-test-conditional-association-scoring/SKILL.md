---
name: cmh-test-conditional-association-scoring
description: Use when samples carry a categorical covariate (e.g. country of origin)
  and you must test SNP-interval patterns for binary phenotype association while controlling
  FWER via Tarone's method, as in HOGImine/FastCMH-style mining.
when_to_use_negative:
- The goal is a plain association scan without covariate conditioning or network priors
  — a simple univariate test or FastCMH single-interval search (data.txt, label.txt,
  cov.txt) is the simpler fit.
- The phenotype is not binary or the features are not SNP markers ordered by genomic
  position — the CMH/Tarone machinery assumes binary labels and naturally ordered
  features.
- You need calibrated permutation p-values by default — permutation testing is off
  unless -p > 0 is explicitly set.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: HOGImine (hogimine_additive)
  role: Executes the CMH-based conditional scoring of higher-order gene-interaction
    patterns with Tarone pruning; run as ./src/hogimine_additive after compiling with
    make under gcc-12.
  repo: https://github.com/BorgwardtLab/HOGImine
- name: FastCMH (fastcmh)
  role: Predecessor method performing significant-interval CMH search conditioned
    on a categorical covariate; reference for the data format (data.txt, label.txt,
    cov.txt) and conditioning design.
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: SiNIMin / SiNIMin-WY
  role: Related network-guided significant-interval mining pipeline using the same
    Tarone/Westfall–Young-style FWER control; alternative when permutation-based testing
    is preferred.
  repo: https://github.com/BorgwardtLab/SiNIMin
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:26cdda7e7b0675ab9adca6833ae3bdf0e8bc0461f4fa3bc507488952431fd101
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - FastCMH (fastcmh)
  - HOGImine
  - HOGImine (hogimine_additive)
  - SiNIMin / SiNIMin-WY
  - gcc-12
  - make
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# cmh-test-conditional-association-scoring

## Summary

Scores candidate genetic meta-markers (combinations of SNP intervals spanning interacting genes) with the Cochran–Mantel–Haenszel statistic conditioned on covariate classes, integrated with Tarone's testability pruning so that only patterns testable at the target FWER are enumerated and reported.

## When to use

Use when samples carry a categorical covariate (e.g. country of origin) and you must test SNP-interval patterns for binary phenotype association while controlling FWER via Tarone's method, as in HOGImine/FastCMH-style mining.

## When NOT to use

- The goal is a plain association scan without covariate conditioning or network priors — a simple univariate test or FastCMH single-interval search (data.txt, label.txt, cov.txt) is the simpler fit.
- The phenotype is not binary or the features are not SNP markers ordered by genomic position — the CMH/Tarone machinery assumes binary labels and naturally ordered features.
- You need calibrated permutation p-values by default — permutation testing is off unless -p > 0 is explicitly set.

## Inputs

- Marker file (-i): binary or additively encoded SNP matrix of size n.SNPs × n.samples (one column per sample)
- Labels file (-l): binary phenotype matrix, one row per sample
- Covariate file: one class index per sample (samples ordered to match data and label files)
- Gene interaction file (-e): genes+edges+3cliques+complexes.txt (PPI network, pathways, complexes)
- SNP-ID and SNP-map files linking SNPs to gene intervals
- Target FWER (e.g. -f 0.05)

## Outputs

- Significant closed patterns, each on one row: CMH p-value; interacting genes spanned by the pattern; SNP positions composing the pattern (-O verbose format)
- Optionally compressed-format output (-o) or permutation-test p-values when -p > 0

## How to apply

For each candidate pattern (a combination of SNP intervals, each gene restricted to its mapped SNP interval), compute the Cochran–Mantel–Haenszel statistic stratified over the K covariate classes, conditioned on the covariate the samples belong to. Implement Tarone's method inside the breadth-first pattern traversal: process_pattern iteratively updates the significance threshold delta_tar and the testable pattern set T, discarding branches whose minimum achievable p-value cannot fall below the current delta_tar. Update delta_tar incrementally as new testable patterns are discovered so the family-wise error rate is controlled at the target FWER (set with -f, e.g. 0.05). Emit only closed patterns; with -O verbose output each row reports the CMH p-value, the interacting genes, and the SNP positions composing the pattern. Optionally enable a permutation testing procedure (-p permutations, default 0).

## Related tools

- **HOGImine (hogimine_additive)** (Executes the CMH-based conditional scoring of higher-order gene-interaction patterns with Tarone pruning; run as ./src/hogimine_additive after compiling with make under gcc-12.) — https://github.com/BorgwardtLab/HOGImine
- **FastCMH (fastcmh)** (Predecessor method performing significant-interval CMH search conditioned on a categorical covariate; reference for the data format (data.txt, label.txt, cov.txt) and conditioning design.) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **SiNIMin / SiNIMin-WY** (Related network-guided significant-interval mining pipeline using the same Tarone/Westfall–Young-style FWER control; alternative when permutation-based testing is preferred.) — https://github.com/BorgwardtLab/SiNIMin

## Examples

```
./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -f 0.05 -O out_athaliana
```

## Evaluation signals

- Run on data/athaliana/interactome_0kb/avrRpm1 with -f 0.05 and confirm the output file out_athaliana exists.
- Check every output row matches the verbose format: p-value; genes; SNP-positions.
- Verify the top reported pattern for the AT3G07040, AT3G25070 pair shows a CMH p-value of order 1e-12.
- Confirm FWER control at the target level (-f 0.05) by checking the reported significance threshold delta_tar was reached via incremental updates over the testable set T.
- Verify only closed patterns whose minimum achievable p-value falls below delta_tar appear in the output (untestable branches pruned).

## Limitations

- Permutation testing is disabled by default (p = 0); it must be explicitly enabled and costs extra compute.
- The reference code is compiled with gcc-12; other compilers 'might work as well' but are not guaranteed.
- SNP data for the mouse dataset must be decompressed (unzip) before use, and covariate classes must be integer-coded with samples sorted by class.

## Evidence

- [methods] Permutation testing off by default: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [methods] Output format options: "output file name, with -o in compressed format and with -O in verbose format."
- [methods] Concrete run command on athaliana data: "./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt"
- [readme] Encodings accepted by HOGImine: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [readme] Covariate conditioning design (FastCMH lineage): [evidence span withheld — anchor status: external_doc]
