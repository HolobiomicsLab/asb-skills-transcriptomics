---
name: genetic-meta-marker-pattern-mining
description: Use when a case/control GWAS-style dataset has SNPs mapped to genes plus
  a gene-interaction prior (PPI edges, 3-cliques, protein complexes) and you need
  significant multi-gene marker combinations stronger than FastCMH or SiNIMin can
  find.
when_to_use_negative:
- The analysis goal is finding contiguous significant intervals with covariate conditioning
  in a single encoding — FastCMH is the appropriate (lower-order) alternative
- No SNP-to-gene mapping or gene-interaction prior is available, so the network-restricted
  search space HOGImine exploits cannot be defined
- The data are already a summarized feature table rather than a per-SNP genotype matrix
  (n_SNPs x n_samples) with per-sample labels
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2269
tools:
- name: HOGImine (hogimine_additive)
  role: Core pattern-mining executable run in binary or additive encoding mode with
    -f FWER target, -p Westfall-Young permutations, and -O verbose output
  repo: https://github.com/BorgwardtLab/HOGImine
- name: gcc-12
  role: Compiler used to build HOGImine via make (other compilers might work as well)
  repo: https://github.com/BorgwardtLab/HOGImine
- name: make
  role: 'Build entry point: run ''cd src; make'' to compile the HOGImine executables'
  repo: https://github.com/BorgwardtLab/HOGImine
- name: unzip
  role: Decompresses packaged SNP data, e.g. data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt.zip
    before running
- name: FastCMH
  role: Baseline miner for comparison — discovers significant intervals but not higher-order
    meta-markers
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: SiNIMin
  role: Baseline network-guided interval miner for comparison against HOGImine meta-markers
  repo: https://github.com/BorgwardtLab/SiNIMin
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:22ee65f365f5cd969fda929cce26c49676b2a2ac4dafef498219c82c318cb58d
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
  - unzip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# genetic-meta-marker-pattern-mining

## Summary

Discover combinations of genetic markers (genetic meta-markers) statistically associated with a binary phenotype using HOGImine, a pattern-mining algorithm that supports higher-order gene interactions (k up to 4), binary and additive encodings, and network-based priors (PPI networks, genetic pathways, protein complexes) while controlling FWER.

## When to use

Use when a case/control GWAS-style dataset has SNPs mapped to genes plus a gene-interaction prior (PPI edges, 3-cliques, protein complexes) and you need significant multi-gene marker combinations stronger than FastCMH or SiNIMin can find.

## When NOT to use

- The analysis goal is finding contiguous significant intervals with covariate conditioning in a single encoding — FastCMH is the appropriate (lower-order) alternative
- No SNP-to-gene mapping or gene-interaction prior is available, so the network-restricted search space HOGImine exploits cannot be defined
- The data are already a summarized feature table rather than a per-SNP genotype matrix (n_SNPs x n_samples) with per-sample labels

## Inputs

- Marker file (binary or additive encoding), matrix of size n_SNPs x n_samples, columns as samples (-i)
- Binary labels file (case/control phenotype) (-l)
- Covariate classes file, one class index per sample (-c)
- SNP names file (-s)
- SNP-to-gene mapping file (-m)
- Gene-interaction prior file (PPI edges, 3-cliques, protein complexes), e.g. data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt (-e)
- Target FWER (default 0.05, -f) and optional number of permutations for Westfall-Young testing (-p)

## Outputs

- Per-pattern output rows (verbose -O format): CMH p-value, interacting genes spanned by the meta-marker, SNP positions composing the pattern
- Tabulated results of significant meta-markers per phenotype, with most significant pattern(s) under FWER 0.05

## How to apply

Compile HOGImine from source with make (gcc-12 toolchain) in the src directory. Prepare the marker file as an n_SNPs x n_samples matrix (columns are samples) in either binary or additive encoding, plus a binary labels file, categorical covariate classes, SNP names, a SNP-to-gene map, and a gene-interaction edge file such as data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt. Invoke ./src/hogimine_additive (or the binary-mode binary) with -i (markers), -l (labels), -c (covariates), -s (SNP names), -m (gene map), -e (interaction prior), -f 0.05 as target FWER, and optionally -p 10000 to activate the Westfall-Young permutation testing procedure, limiting interaction order to k up to 4. Request verbose output with -O so every surviving pattern is written as one row containing the CMH p-value, the genes spanned by the pattern, and the SNP positions composing it. When the correct binarization of the genotypes is unknown, prefer additive mode: a mis-specified dominant binarization of recessively associated markers retains only a fraction of the association and loses statistical power.

## Related tools

- **HOGImine (hogimine_additive)** (Core pattern-mining executable run in binary or additive encoding mode with -f FWER target, -p Westfall-Young permutations, and -O verbose output) — https://github.com/BorgwardtLab/HOGImine
- **gcc-12** (Compiler used to build HOGImine via make (other compilers might work as well)) — https://github.com/BorgwardtLab/HOGImine
- **make** (Build entry point: run 'cd src; make' to compile the HOGImine executables) — https://github.com/BorgwardtLab/HOGImine
- **unzip** (Decompresses packaged SNP data, e.g. data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt.zip before running)
- **FastCMH** (Baseline miner for comparison — discovers significant intervals but not higher-order meta-markers) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **SiNIMin** (Baseline network-guided interval miner for comparison against HOGImine meta-markers) — https://github.com/BorgwardtLab/SiNIMin

## Examples

```
cd src && make; ./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -e data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt -f 0.05 -p 10000 -O
```

## Evaluation signals

- All reported patterns meet the FWER target of 0.05 (or under 10^4 Westfall-Young permutations with -p 10000)
- Verbose output (-O) rows parse into the expected schema: CMH p-value, gene list, SNP positions
- Higher-order discoveries are confirmed against baselines: e.g. top Anthocyanin22 hit is the three-gene meta-marker AT2G45660-AT4G37940-AT3G61120, a pattern FastCMH and SiNIMin could not discover
- In encoding-power-loss tests, additive mode on correctly encoded data yields more significant patterns and a smaller smallest CMH p-value than binary mode on mis-specified (dominant) binarization at each k=1,2
- Compilation completes with make under gcc-12 without errors before any run

## Limitations

- Using the wrong binary encoding for data whose phenotype follows a different encoding (e.g. dominant binarization of a recessive signal) causes a serious loss of statistical power; additive mode is preferred when the correct binarization is unknown
- The search space depends on the supplied gene-interaction prior; prior-free higher-order discovery is not supported
- SiNIMin compilation may require adjusting the Makefile and linking the Eigen library; HOGImine currently compiles with gcc-12 but 'other compilers might work as well'
- No changelog is available for the repository

## Evidence

- [abstract] abstract: "Our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways, and protein complexes, to restrict its search space."
- [intro] intro: "is a pattern-mining-based algorithm for finding genetic meta-markers, i.e. combinations of genetic markers, that show a statistical association with a phenotype"
- [methods] methods: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [readme] readme: "- ```-i file```: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$, i.e. with each column being a sample."
- [readme] readme_compilation: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
