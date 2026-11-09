---
name: multi-phenotype-gwas-pattern-scan
description: Use when a binary/additive-encoded SNP x sample matrix with case/control
  labels, covariate classes, and a PPI/gene-interaction prior is available and the
  goal is to discover higher-order gene meta-markers associated with multiple phenotypes
  beyond single-interval methods like FastCMH or SiNIMin.
when_to_use_negative:
- When the goal is to test single genomic intervals conditioned on a covariate only
  — FastCMH (R package or C code) is the appropriate tool, not higher-order meta-marker
  mining
- When no prior biological knowledge of gene interactions (PPI network, pathways,
  complexes) is available or the features cannot be mapped to genes/nodes in such
  a network
- When labels are not binary case/control; HOGImine expects a binary labels matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
tools:
- name: HOGImine (hogimine_additive)
  role: 'Primary miner: enumerates higher-order gene interaction meta-markers with
    CMH testing and optional Westfall-Young permutation testing'
  repo: https://github.com/BorgwardtLab/HOGImine
- name: FastCMH
  role: Baseline significant-interval search (C or R fastcmh) used to benchmark meta-marker
    discoveries
  repo: https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- name: SiNIMin
  role: Baseline network-guided interval mining (with SiNIMin-WY permutation variant)
    used for comparison
  repo: https://github.com/BorgwardtLab/SiNIMin
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:0a5c7c071067f548110e423a46898cedc4a5fbb2eb127edcbbe0153f8e7ef29e
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multi-phenotype-gwas-pattern-scan

## Summary

Run HOGImine (hogimine_additive) over each phenotype of a GWAS dataset (e.g. Atwell et al. 2010 A. thaliana) to mine significant gene-interaction meta-markers, using network priors up to k=4 interacting genes and Westfall-Young permutation testing to control family-wise error rate at 0.05.

## When to use

Use when a binary/additive-encoded SNP x sample matrix with case/control labels, covariate classes, and a PPI/gene-interaction prior is available and the goal is to discover higher-order gene meta-markers associated with multiple phenotypes beyond single-interval methods like FastCMH or SiNIMin.

## When NOT to use

- When the goal is to test single genomic intervals conditioned on a covariate only — FastCMH (R package or C code) is the appropriate tool, not higher-order meta-marker mining
- When no prior biological knowledge of gene interactions (PPI network, pathways, complexes) is available or the features cannot be mapped to genes/nodes in such a network
- When labels are not binary case/control; HOGImine expects a binary labels matrix

## Inputs

- Marker matrix file (-i), n.SNPs x n.samples, binary or additive encoding
- Binary phenotype label file (-l), one label per sample
- Covariate class file (-c), one class index per sample
- SNP names file (-s)
- SNP-to-gene mapping file (-m)
- Gene interaction prior file (-e), e.g. genes+edges+3cliques+complexes.txt (PPI edges, 3-cliques, protein complexes)

## Outputs

- Verbose (-O) output file listing each significant meta-marker as one row: CMH p-value, interacting genes spanned, and composing SNP positions
- Per-phenotype table of number of significant meta-markers and most significant pattern(s)

## How to apply

Compile HOGImine from the src directory with make (gcc-12). For each phenotype, invoke ./src/hogimine_additive on the transposed marker matrix (-i, n.SNPs x n.samples, binary or additive encoding), binary labels (-l), covariate classes (-c), SNP names (-s), a SNP-to-gene map (-m), and the prior interaction file (-e, e.g. data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt combining PPI edges, 3-cliques and protein complexes) with interaction order k up to 4. Set the target FWER with -f 0.05 and enable Westfall-Young permutation testing with -p 10000; use -O so every surviving pattern is printed in verbose format as one row of CMH p-value, interacting genes, and composing SNP positions. Parse the verbose output per phenotype to tabulate the number of significant meta-markers and the most significant pattern(s), then compare against FastCMH and SiNIMin baselines to confirm discoveries those methods cannot make.

## Related tools

- **HOGImine (hogimine_additive)** (Primary miner: enumerates higher-order gene interaction meta-markers with CMH testing and optional Westfall-Young permutation testing) — https://github.com/BorgwardtLab/HOGImine
- **FastCMH** (Baseline significant-interval search (C or R fastcmh) used to benchmark meta-marker discoveries) — https://github.com/BorgwardtLab/Genetic-Heterogeneity-Discovery-FastCMH
- **SiNIMin** (Baseline network-guided interval mining (with SiNIMin-WY permutation variant) used for comparison) — https://github.com/BorgwardtLab/SiNIMin

## Examples

```
./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -e data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt -f 0.05 -p 10000 -O
```

## Evaluation signals

- All reported patterns meet the target FWER of 0.05 under 10^4 Westfall-Young permutations
- Top Anthocyanin22 hit is the three-gene meta-marker AT2G45660-AT4G37940-AT3G61120, spanning three genes in a connected subgraph of the prior network
- Verbose output rows contain exactly three fields in order: CMH p-value, interacting genes, SNP positions
- Discovered meta-markers span connected subgraphs of the interaction prior (PPI edges, 3-cliques, complexes)
- The pattern set includes associations that FastCMH and SiNIMin baselines do not recover, confirming the higher-order power claim

## Limitations

- The class of discoverable meta-markers is restricted to combinations of genes that form connected subgraphs in the provided interaction prior; genes absent from the prior cannot participate
- Permutation testing with -p 10000 incurs substantial runtime; p defaults to 0 (no permutations)
- Compilation is only verified with gcc-12; other compilers 'might work as well' per the README
- No changelog is available for the repository, so version drift in results is possible

## Evidence

- [abstract] We propose HOGImine, a novel algorithm that expands the class of discoverable genetic meta-markers by considering higher-order interactions of genes and by allowing multiple encodings for the genetic: "We propose HOGImine, a novel algorithm that expands the class of discoverable genetic meta-markers by considering higher-order interactions of genes and by allowing multiple encodings for the genetic"
- [results] on the Anthocyanin22 and LY phenotypes, the most significant meta-marker spans an interaction of three genes (including the AT2G45660-AT4G37940-AT3G61120 combination): [evidence span withheld — anchor status: not_found]
- [methods] number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [methods] output file name, with -o in compressed format and with -O in verbose format: "output file name, with -o in compressed format and with -O in verbose format."
- [readme] Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
