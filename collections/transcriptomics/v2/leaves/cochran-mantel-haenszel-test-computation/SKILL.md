---
name: cochran-mantel-haenszel-test-computation
description: Use when scoring binary- or additive-encoded SNP meta-markers against
  case/control labels in samples split into categorical covariate classes, where CMH
  p-values must be computed per pattern and a FWER-controlled significance threshold
  applied.
when_to_use_negative:
- Data are continuous-valued or labels are not binary case/control — the CMH stratified
  contingency-table test requires binary labels and categorical covariate classes
- Samples have no categorical covariate structure to stratify over and a simple univariate
  association test suffices
- The goal is detecting heterogeneous subgroups rather than overall phenotype association
  (see SiNIMin / FastCMH lineage)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: HOGImine
  role: Pattern-mining algorithm that evaluates candidate higher-order gene-interaction
    meta-markers by their CMH p-value within a Tarone-style FWER-controlled search
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: Binary compiled for additive-encoded marker matrices; computes CMH p-values
    on additive-encoded SNPs (e.g. M. musculus BMC.Mode run)
  repo: https://github.com/BorgwardtLab/HOGImine
- name: make
  role: Builds the HOGImine C++ binaries (cd src; make) so the CMH-test-based mining
    can be run
  repo: https://github.com/BorgwardtLab/HOGImine
- name: gcc-12
  role: Compiler used for HOGImine; other compilers might work as well
  repo: https://github.com/BorgwardtLab/HOGImine
- name: unzip
  role: Decompresses the mmusculus SNP matrix (BMC.Mode_X_T.txt.zip) before CMH-based
    mining
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:0613789b7ed9183c8d1e89b115e82b026ee66a8a01e335c34b77a2554cd05f6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  - gcc-12
  - make
  - unzip
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cochran-mantel-haenszel-test-computation

## Summary

Computes the Cochran–Mantel–Haenszel (CMH) p-value of candidate genetic meta-markers in HOGImine, stratified over categorical covariate classes (e.g. kinship/population structure), to test phenotype association while controlling FWER at alpha=0.05.

## When to use

Use when scoring binary- or additive-encoded SNP meta-markers against case/control labels in samples split into categorical covariate classes, where CMH p-values must be computed per pattern and a FWER-controlled significance threshold applied.

## When NOT to use

- Data are continuous-valued or labels are not binary case/control — the CMH stratified contingency-table test requires binary labels and categorical covariate classes
- Samples have no categorical covariate structure to stratify over and a simple univariate association test suffices
- The goal is detecting heterogeneous subgroups rather than overall phenotype association (see SiNIMin / FastCMH lineage)

## Inputs

- Marker matrix (-i), SNPs x samples, binary or additive encoding (e.g. avrRpm1_X.txt, BMC.Mode_X_T.txt)
- Binary phenotype labels file (-l), e.g. avrRpm1_Y.txt or BMC.Mode_y_bin.txt
- Categorical covariate class file (-c), e.g. avrRpm1_covar_n2.txt or BMC.Mode_kinship.txt
- Optional gene interaction priors file (-e), e.g. genes+edges+3cliques+complexes.txt
- SNP-to-gene mapping (-m) and SNP name file (-s)
- Target FWER (-f, e.g. 0.05) and number of permutations (-p)

## Outputs

- Output file with one row per significant pattern: CMH p-value, interacting genes with dominant/recessive encoding, and SNP positions (verbose -O or compressed -o format)
- With -p > 0: a permutation-testing-based FWER estimate (FWER^wy) used to set the final significance cutoff

## How to apply

For each candidate meta-marker (a combination of SNP intervals in interacting genes, using dominant or recessive encodings), build its pattern support across samples and compute its CMH p-value on the binary labels, conditioning on the covariate classes supplied via the -c file (e.g. BMC.Mode_kinship.txt). HOGImine embeds this test in a Tarone-style branch-and-bound search: the significance cutoff delta_tar is updated as mining proceeds (initially bounded by |T|·delta_tar), pruning patterns whose best achievable p-value cannot pass the cutoff for target FWER -f 0.05. When the Westfall–Young permutation extension is enabled (-p > 0), K label permutations are pre-computed, the running minimum p-value per permutation is tracked over all processed patterns, and the cutoff is decreased using the estimated FWER^wy instead of |T|·delta_tar. Patterns passing the final threshold are written to the output file, each row reporting the CMH p-value, the interacting genes with their encoding, and the SNP positions.

## Related tools

- **HOGImine** (Pattern-mining algorithm that evaluates candidate higher-order gene-interaction meta-markers by their CMH p-value within a Tarone-style FWER-controlled search) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (Binary compiled for additive-encoded marker matrices; computes CMH p-values on additive-encoded SNPs (e.g. M. musculus BMC.Mode run)) — https://github.com/BorgwardtLab/HOGImine
- **make** (Builds the HOGImine C++ binaries (cd src; make) so the CMH-test-based mining can be run) — https://github.com/BorgwardtLab/HOGImine
- **gcc-12** (Compiler used for HOGImine; other compilers might work as well) — https://github.com/BorgwardtLab/HOGImine
- **unzip** (Decompresses the mmusculus SNP matrix (BMC.Mode_X_T.txt.zip) before CMH-based mining)

## Examples

```
./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt -c data/mmusculus/BMC.Mode/BMC.Mode_kinship.txt -s data/mmusculus/BMC.Mode/SNPs_ids.txt -m data/mmusculus/BMC.Mode/gene2snps.txt -e data/mmusculus/BMC.Mode/all_int_compl.txt -f 0.05 -O out_mus
```

## Evaluation signals

- Run terminates and output contains one row per significant pattern with CMH p-value, interacting genes, and SNP positions, all passing the FWER-estimated threshold at level 0.05
- Top output row on M. musculus BMC.Mode (30,890 significant patterns, 251 distinct gene interactions) attains CMH p-value 3.65197e-25 for the ENSMUSG00000047040 (dominant)/ENSMUSG00000048175 (recessive) pattern
- With Westfall–Young extension (-p > 0), the cutoff decreases using FWER^wy from per-permutation minimum p-values instead of |T|·delta_tar
- No output pattern exceeds the target FWER (-f 0.05) significance threshold

## Limitations

- Permutation testing (p > 0) requires pre-computing K label permutations and tracking running minimum p-values per permutation, increasing runtime
- Stratified CMH requires covariate classes as positive integer indices with samples grouped/orderable by class (FastCMH lineage requirement)
- The code is compiled with gcc-12; other compilers 'might work as well' but are not guaranteed

## Evidence

- [methods] Usage example / CMH p-value reporting: "./src/hogimine_additive -i data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt -l data/mmusculus/BMC.Mode/BMC.Mode_y_bin.txt"
- [methods] Permutation testing option: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
- [methods] Output format: "output file name, with -o in compressed format and with -O in verbose format."
- [readme] Compilation: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [abstract] Pattern mining description: "is a pattern-mining-based algorithm for finding genetic meta-markers, i.e. combinations of genetic markers, that show a statistical association with a phenotype"
