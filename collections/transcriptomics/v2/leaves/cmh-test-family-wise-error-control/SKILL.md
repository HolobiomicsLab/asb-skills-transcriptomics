---
name: cmh-test-family-wise-error-control
description: Use when discovering combinations of genetic markers (meta-markers) associated
  with a binary phenotype and the number of tested patterns is large enough that raw
  CMH p-values would inflate family-wise error; supports conditioning on covariate
  classes.
when_to_use_negative:
- When testing a single, pre-specified SNP association with no multiple-testing burden
  — a plain CMH test without permutation FWER control suffices
- When the phenotype is not binary (HOGImine's labels file must be a binary matrix
  of cases/controls)
- When no biological prior on gene interactions (PPI network, pathways, protein complexes)
  is available and the unrestricted search is infeasible for the target k
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0625
tools:
- name: HOGImine (hogimine_additive)
  role: Executes CMH significance testing of higher-order gene interaction meta-markers
    with Westfall-Young permutation-based FWER control at the target FWER (-f) over
    p permutations (-p)
  repo: https://github.com/BorgwardtLab/HOGImine
- name: FastCMH
  role: Baseline significant interval search using CMH tests for genetic heterogeneity
    discovery; lacks HOGImine's higher-order gene interaction power
  repo: BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: SiNIMin / SiNIMin-WY
  role: Baseline network-guided significant network interval mining with permutation
    testing (SiNIMin-WY); cannot discover the three-gene meta-markers HOGImine finds
  repo: BorgwardtLab/SiNIMin
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:5011a67eaa2762e63d1fba5eecd627522722ec284e4f5e8584534c97cff9737c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - FastCMH
  - HOGImine
  - HOGImine (hogimine_additive)
  - SiNIMin / SiNIMin-WY
  - gcc-12
  - make
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cmh-test-family-wise-error-control

## Summary

Assess statistical significance of genetic meta-markers using CMH (Cochran-Mantel-Haenszel) p-values with family-wise error rate (FWER) control at 0.05 via a Westfall-Young permutation procedure, as implemented in HOGImine for higher-order gene interaction discovery.

## When to use

Use when discovering combinations of genetic markers (meta-markers) associated with a binary phenotype and the number of tested patterns is large enough that raw CMH p-values would inflate family-wise error; supports conditioning on covariate classes.

## When NOT to use

- When testing a single, pre-specified SNP association with no multiple-testing burden — a plain CMH test without permutation FWER control suffices
- When the phenotype is not binary (HOGImine's labels file must be a binary matrix of cases/controls)
- When no biological prior on gene interactions (PPI network, pathways, protein complexes) is available and the unrestricted search is infeasible for the target k

## Inputs

- Marker matrix file (-i), binary or additive encoding, size n.SNPs x n.samples (e.g. data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt)
- Binary phenotype labels file (-l), one label per sample
- Covariate class file (-c), one class index per sample (categorical covariate to condition on)
- SNP names file (-s)
- SNP-to-gene mapping file (-m)
- Gene interaction prior file (-e), e.g. data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt combining PPI edges, 3-cliques and protein complexes

## Outputs

- Verbose output file (-O) with one row per surviving pattern: CMH p-value, interacting genes spanned by the pattern, and SNP positions composing the pattern
- Compressed output file (-o) alternative format
- Tabulation per phenotype of the number of significant meta-markers and the most significant pattern(s)

## How to apply

Compile HOGImine in src with make (gcc-12), then run ./src/hogimine_additive on binary- or additive-encoded marker matrices (-i, n.SNPs x n.samples), binary labels (-l), covariate classes (-c), SNP names (-s), a SNP-to-gene map (-m), and a biological prior file of PPI edges, 3-cliques and protein complexes (-e), allowing gene interactions up to order k=4. Set the target FWER with -f 0.05 and activate the Westfall-Young permutation testing procedure with -p 10000 (default -p 0 disables permutations). Each surviving pattern's CMH p-value is computed conditional on covariate classes; the permutation step guarantees all reported patterns jointly meet the FWER target. Request verbose output with -O (rather than compressed -o) so every surviving pattern is written as one row: CMH p-value, followed by the interacting genes, and finally the SNP positions composing the pattern. Validate by confirming all reported patterns meet FWER 0.05 under 10^4 permutations.

## Related tools

- **HOGImine (hogimine_additive)** (Executes CMH significance testing of higher-order gene interaction meta-markers with Westfall-Young permutation-based FWER control at the target FWER (-f) over p permutations (-p)) — https://github.com/BorgwardtLab/HOGImine
- **FastCMH** (Baseline significant interval search using CMH tests for genetic heterogeneity discovery; lacks HOGImine's higher-order gene interaction power) — BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **SiNIMin / SiNIMin-WY** (Baseline network-guided significant network interval mining with permutation testing (SiNIMin-WY); cannot discover the three-gene meta-markers HOGImine finds) — BorgwardtLab/SiNIMin

## Examples

```
./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -c cov.txt -s snps.txt -m map.txt -e data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt -f 0.05 -p 10000 -O
```

## Evaluation signals

- Every reported pattern meets the FWER target of 0.05 under 10^4 Westfall-Young permutations
- On the Atwell et al. 2010 Arabidopsis GWAS data, the top Anthocyanin22 hit is the three-gene meta-marker AT2G45660-AT4G37940-AT3G61120
- Verbose (-O) output rows are parseable as: CMH p-value, then interacting genes, then composing SNP positions
- Baseline methods FastCMH and SiNIMin fail to recover the same multi-gene meta-markers on Anthocyanin22 and LY phenotypes
- Number of significant meta-markers per phenotype is tabulated and consistent with the FWER-controlled cutoff

## Limitations

- The permutation testing procedure is optional (-p, default 0); without it, FWER control is not enforced
- Prior biological knowledge (PPI networks, genetic pathways, protein complexes) is used to restrict the search space, so discovered meta-markers are limited to connected subgraphs in the provided network
- The code is compiled with gcc-12; other compilers 'might work as well' but are not guaranteed
- SNP data for some datasets (e.g. data/mmusculus/BMC.Mode) ships zipped and must be decompressed (unzip) before running

## Evidence

- [methods] Optional permutation testing procedure: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [methods] Verbose vs compressed output: "output file name, with -o in compressed format and with -O in verbose format."
- [methods] Prior file for Arabidopsis PPI: "-e data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt"
- [abstract] Statistical power over previous methods: "Our experimental evaluation shows that the algorithm has a substantially higher statistical power compared to previous methods, allowing it to discover genetic mutations statistically associated with"
- [readme] Prior biological knowledge restricts search space: "our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways and protein complexes, to restrict its search space."
