---
name: genetic-interaction-pattern-mining
description: Use when genotype data (SNPs x samples, binary or additive encoding),
  binary phenotype labels, covariate classes, and a gene-interaction priors file (PPI
  edges, pathways, complexes) are available, and the goal is to discover higher-order
  (k>=2) multi-gene meta-markers associated with the phenotype.
when_to_use_negative:
- The phenotype labels are not binary case/control — HOGImine's CMH test and the -l
  file require a binary label vector
- No gene-interaction priors (PPI network, pathways, complexes) are available and
  the goal is single-SNP or two-locus interval search — FastCMH or SiNIMin are the
  appropriate baselines instead
- Only additive-encoded markers are available and the binary-encoding binary (not
  hogimine_additive) is used — the additive variant must be selected to match the
  encoding
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HOGImine
  role: 'Main pattern miner: breadth-first branch-and-bound enumeration with Tarone
    pruning and CMH testing on binary-encoded markers'
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: Additive-encoding variant of the miner (e.g. for the M. musculus BMC.Mode
    matrix BMC.Mode_X_T.txt)
  repo: https://github.com/BorgwardtLab/HOGImine
- name: gcc-12
  role: Reference compiler used to build the C++ source with make; other compilers
    may work but are unverified
- name: unzip
  role: Decompresses the M. musculus SNP data (BMC.Mode_X_T.txt.zip) before running
    the additive miner
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_008
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:07c2d01b118e0b0f0401b4bcfee082ff759990853465368c3adcedee0578cf9d
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# genetic-interaction-pattern-mining

## Summary

Mine higher-order genetic meta-markers with HOGImine: enumerate connected gene-interaction patterns over SNP intervals in a breadth-first branch-and-bound search, test each pattern with the Cochran-Mantel-Haenszel (CMH) statistic, and control FWER at alpha=0.05 using Tarone pruning with optional Westfall-Young permutation testing.

## When to use

Use when genotype data (SNPs x samples, binary or additive encoding), binary phenotype labels, covariate classes, and a gene-interaction priors file (PPI edges, pathways, complexes) are available, and the goal is to discover higher-order (k>=2) multi-gene meta-markers associated with the phenotype under FWER control.

## When NOT to use

- The phenotype labels are not binary case/control — HOGImine's CMH test and the -l file require a binary label vector
- No gene-interaction priors (PPI network, pathways, complexes) are available and the goal is single-SNP or two-locus interval search — FastCMH or SiNIMin are the appropriate baselines instead
- Only additive-encoded markers are available and the binary-encoding binary (not hogimine_additive) is used — the additive variant must be selected to match the encoding

## Inputs

- Marker matrix file (-i), size n.SNPs x n.samples, with each column a sample; values binary-encoded or additive-encoded
- Binary label file (-l), one label (0/1) per sample
- Covariate-class file (-c), one class index per sample (e.g. kinship classes)
- SNP-ID file (-s) and SNP-to-gene map file (-m) linking SNPs to genes
- Gene-interaction priors file (-e), e.g. genes+edges+3cliques+complexes.txt for PPI edges, triadic cliques and complexes
- Parameters: target FWER (-f, e.g. 0.05), number of permutations (-p, default 0), max SNP interval length per gene (-d), output file (-o compressed or -O verbose)

## Outputs

- Output file (e.g. out_athaliana, out_mus) with one row per significant pattern: CMH p-value; interacting genes (with dominant/recessive encoding annotation); SNP positions composing the pattern
- Optionally, count of significant patterns and distinct gene interactions per phenotype (e.g. 30,890 patterns in 251 interactions for BMC.Mode)

## How to apply

Compile the C++ miner with `cd src && make` under gcc-12, then run it with the marker matrix (-i, size n.SNPs x n.samples, one sample per column), binary label file (-l), covariate-class file (-c), SNP-ID file (-s), SNP-to-gene map (-m), and gene-interaction priors (-e), setting the target FWER with -f 0.05 and verbose output with -O. The miner expands the poset of patterns spanned by each gene interaction breadth-first (recursive_gen), bounding each gene to its mapped SNP interval (parameter -d limits the maximum interval length), and evaluates each candidate pattern by computing the CMH statistic conditioned on covariate classes. Tarone's method is applied in process_pattern: untestable patterns whose minimum achievable p-value cannot drop below the current threshold delta_tar are pruned, and delta_tar is decreased incrementally as testable patterns accumulate so family-wise error rate is controlled at the -f level. Only closed patterns are emitted, one row per pattern as: CMH p-value; interacting genes; composing SNP positions. For p > 0 (permutations, default 0), K label permutations are precomputed and the Westfall-Young estimator FWER^wy replaces the |T|*delta_tar bound when lowering the significance cutoff.

## Related tools

- **HOGImine** (Main pattern miner: breadth-first branch-and-bound enumeration with Tarone pruning and CMH testing on binary-encoded markers) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (Additive-encoding variant of the miner (e.g. for the M. musculus BMC.Mode matrix BMC.Mode_X_T.txt)) — https://github.com/BorgwardtLab/HOGImine
- **gcc-12** (Reference compiler used to build the C++ source with make; other compilers may work but are unverified)
- **unzip** (Decompresses the M. musculus SNP data (BMC.Mode_X_T.txt.zip) before running the additive miner)

## Examples

```
./src/hogimine_additive -i avrRpm1_X.txt -l avrRpm1_Y.txt -c avrRpm1_covar_n2.txt -s avrRpm1_snpID.txt -m avrRpm1_snp_map.txt -e genes+edges+3cliques+complexes.txt -f 0.05 -O out_athaliana
```

## Evaluation signals

- Output rows follow the verbose format: CMH p-value; genes; SNP positions, with all reported patterns passing the FWER-controlled threshold at -f 0.05
- On the a. thaliana avrRpm1 binary example (-f 0.05), the top pattern for the AT3G07040/AT3G25070 pair reaches a CMH p-value of order 1e-12
- On M. musculus BMC.Mode, the top row attains CMH p-value 3.65197e-25 for the ENSMUSG00000047040 (dominant) / ENSMUSG00000048175 (recessive) pattern, and 30,890 patterns across 251 gene interactions are found
- With k=2, the reported pattern list (p-value, genes, SNP positions) is identical to SiNIMin's, confirming the same testable-pattern set
- Compiled binary exists after `cd src && make`, and decompressed BMC.Mode_X_T.txt is present in data/mmusculus/BMC.Mode/ before the additive run

## Limitations

- The reference code is compiled with gcc-12; other compilers 'might work as well' but are not guaranteed
- Permutation testing is optional (default -p 0); with p>0 the threshold update switches to the Westfall-Young estimator FWER^wy instead of |T|*delta_tar, changing which patterns are reported
- The search space is restricted by the supplied gene-interaction priors and the -d bound on maximum SNP interval length, so results depend on the quality/coverage of the priors file
- The M. musculus SNP data ship compressed and must be unzipped before use

## Evidence

- [abstract] Our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways, and protein complexes, to restrict its search space.: "Our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways, and protein complexes, to restrict its search space."
- [intro] HOGImine accepts both binary encodings and additive encodings for the markers.: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [methods] number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure.: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [readme] Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well.: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [readme] -i file```: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$: "- ```-i file```: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$, i.e. with each column being a sample."
