---
name: closed-pattern-discovery-in-snp-data
description: Use when mining significant higher-order combinations of SNPs (genetic
  meta-markers) across interacting genes, given binary or additive-encoded marker
  matrices, phenotype labels, a covariate, and a PPI/pathway/complex prior network.
when_to_use_negative:
- You are only testing single SNPs or predefined fixed genomic intervals — FastCMH-style
  interval search, not higher-order network-guided pattern mining, is the right tool.
- You have no prior gene-interaction knowledge (PPI network, pathways, complexes)
  and cannot restrict the search to connected gene subgraphs.
- Markers are not binary- or additive-encoded; other encodings fall outside HOGImine's
  supported inputs.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0091
tools:
- name: HOGImine (hogimine_additive)
  role: Implements the breadth-first pattern enumeration with Tarone pruning and CMH
    testing; invoked with -i/-l/-e/-f/-d/-o/-O options
  repo: https://github.com/BorgwardtLab/HOGImine
- name: gcc-12
  role: Reference compiler used to build the src directory with make
- name: make
  role: Build system used to compile HOGImine from src
- name: unzip
  role: Decompresses zipped SNP data files, e.g. BMC.Mode_X_T.txt.zip in data/mmusculus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:c9bc21e1c48f8c20c6f4d193b8e2b1e9faf98b491e3cb0f1e24ec9c2e532b940
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# closed-pattern-discovery-in-snp-data

## Summary

Reproduce HOGImine's breadth-first pattern enumeration with Tarone pruning: the poset of higher-order genetic interaction patterns spanning gene-interaction network cliques is expanded breadth-first, untestable branches are pruned against the updated Tarone significance threshold delta_tar, and only closed patterns are emitted with their CMH p-value, genes, and SNP positions.

## When to use

Use when mining significant higher-order combinations of SNPs (genetic meta-markers) across interacting genes, given binary or additive-encoded marker matrices, phenotype labels, a covariate, and a PPI/pathway/complex prior network.

## When NOT to use

- You are only testing single SNPs or predefined fixed genomic intervals — FastCMH-style interval search, not higher-order network-guided pattern mining, is the right tool.
- You have no prior gene-interaction knowledge (PPI network, pathways, complexes) and cannot restrict the search to connected gene subgraphs.
- Markers are not binary- or additive-encoded; other encodings fall outside HOGImine's supported inputs.

## Inputs

- Marker matrix file (binary or additive encoding), n. SNPs x n. samples, e.g. avrRpm1_X.txt
- Binary phenotype labels file (n rows), e.g. avrRpm1_Y.txt
- Categorical covariate file (n rows of class indices)
- SNP-ID and SNP-to-gene map files
- Biological-priors gene-interaction file (edges + 3-cliques + complexes), e.g. genes+edges+3cliques+complexes.txt
- Parameters: -f target FWER (e.g. 0.05), -d maximum SNP interval length, -o/-O output format

## Outputs

- Output file (e.g. out_athaliana) of closed patterns, one per row: CMH p-value; interacting genes spanned by the pattern; SNP positions composing the pattern
- Optionally, permutation-testing results when -p > 0

## How to apply

Compile the src directory with make (reference compiler gcc-12) to build hogimine_additive. Implement recursive_gen to expand, breadth-first, the poset of patterns spanned by each gene interaction from the biological-priors file (e.g. genes+edges+3cliques+complexes.txt), restricting each gene to its mapped SNP interval with -d bounding the maximum interval length. In process_pattern, compute the Cochran-Mantel-Haenszel statistic conditioned on covariate classes and discard untestable patterns whose minimum achievable p-value cannot fall below the current Tarone threshold delta_tar, which is updated incrementally so the family-wise error rate is controlled at the -f target (e.g. 0.05). Emit only closed patterns, each on one row as CMH p-value; genes; SNP positions, using -O for verbose output. Validate on data/athaliana/interactome_0kb/avrRpm1 with -f 0.05 and check that the top patterns reproduce highly significant results (e.g. CMH p-value of order 1e-12 for the AT3G07040, AT3G25070 pair).

## Related tools

- **HOGImine (hogimine_additive)** (Implements the breadth-first pattern enumeration with Tarone pruning and CMH testing; invoked with -i/-l/-e/-f/-d/-o/-O options) — https://github.com/BorgwardtLab/HOGImine
- **gcc-12** (Reference compiler used to build the src directory with make)
- **make** (Build system used to compile HOGImine from src)
- **unzip** (Decompresses zipped SNP data files, e.g. BMC.Mode_X_T.txt.zip in data/mmusculus)

## Examples

```
./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -e data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt -f 0.05 -O
```

## Evaluation signals

- Running on data/athaliana/interactome_0kb/avrRpm1 with -f 0.05 creates the output file out_athaliana following the 'p-value; genes; SNP-positions' row format.
- Top reported patterns are highly significant, e.g. CMH p-value of order 1e-12 for the AT3G07040, AT3G25070 pair.
- Every emitted pattern is closed: no reported pattern is a subset of another reported pattern spanning the same gene subgraph.
- All reported patterns satisfy the Tarone testability criterion — their minimum achievable p-value falls below the final delta_tar at the target FWER (default 0.05).
- Emitted patterns span connected subgraphs of the gene-interaction prior file, with each gene restricted to its mapped SNP interval bounded by -d.

## Limitations

- The reference code is compiled with gcc-12; other compilers might work but are not guaranteed.
- SNP data must be decompressed first for some datasets (e.g. data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt.zip).
- The search space is restricted by biological priors (PPI networks, genetic pathways, protein complexes); interactions absent from the prior network will not be discovered.
- Only closed patterns are reported, so sub-patterns of significant meta-markers are not emitted.

## Evidence

- [methods] HOGImine explores the pattern search space in a breadth-first fashion, and each pattern is processed by the procedure process_pattern, which implements Tarone's method by iteratively updating the significance threshold δtar and the testable pattern set T.: [evidence span withheld — anchor status: not_found]
- [readme] Compilation uses make. Currently the code is compiled with gcc-12`` but other compilers might work as well.: "Compilation uses make. Currently the code is compiled with gcc-12`` but other compilers might work as well."
- [methods] output file name, with -o in compressed format and with -O in verbose format.: "output file name, with -o in compressed format and with -O in verbose format."
- [abstract] Our method can exploit prior biological knowledge on gene interactions, such as protein-prote
in interaction networks, genetic pathways, and protein complexes, to restrict its search space.: "Our method can exploit prior biological knowledge on gene interactions, such as protein-prote"
- [methods] number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure.: "number $p$ of permutations (default $0$), if $p > 0$ it runs a permutation testing procedure."
