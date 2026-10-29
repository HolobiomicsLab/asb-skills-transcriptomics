---
name: tarone-fwer-pruning-threshold-updating
description: Use when mining high-order genetic interaction patterns (binary or additive
  encoded SNPs within mapped gene intervals) with a CMH test and you must avoid enumerating
  the full pattern space while controlling FWER.
when_to_use_negative:
- When the analysis conditions on a single marker interval rather than higher-order
  gene interaction patterns — FastCMH/SiNIMin address significant interval search
  and are the wrong tool for the HOGImine pattern poset.
- When no prior gene-interaction network or gene-to-SNP interval mapping exists and
  an unrestricted enumeration is required without Tarone pruning over a biologically
  constrained poset.
- When the phenotype is not binary/case-control and a CMH test conditioned on covariate
  classes is not applicable.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: HOGImine
  role: Implements the breadth-first pattern enumeration (recursive_gen) and Tarone-pruned
    processing (process_pattern); invoked as hogimine_additive for additive-encoded
    markers.
  repo: https://github.com/BorgwardtLab/HOGImine
- name: gcc-12
  role: Compiler used to build HOGImine before running the Tarone-pruned enumeration.
- name: make
  role: Build tool for compiling HOGImine's src directory.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:1129396a1e1c0db2751e9606ed375420750bf2e9cbb222f992c7a467ceb05e70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  - gcc-12
  - make
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Tarone FWER-pruning threshold updating

## Summary

Applies Tarone's method inside HOGImine's breadth-first pattern enumeration: untestable patterns whose minimum achievable p-value cannot fall below the current Tarone threshold are pruned, and the threshold is updated incrementally as new testable patterns are discovered, so family-wise error rate is controlled at the target level.

## When to use

Use when mining high-order genetic interaction patterns (binary or additive encoded SNPs within mapped gene intervals) with a CMH test and you must avoid enumerating the full pattern space while controlling FWER.

## When NOT to use

- When the analysis conditions on a single marker interval rather than higher-order gene interaction patterns — FastCMH/SiNIMin address significant interval search and are the wrong tool for the HOGImine pattern poset.
- When no prior gene-interaction network or gene-to-SNP interval mapping exists and an unrestricted enumeration is required without Tarone pruning over a biologically constrained poset.
- When the phenotype is not binary/case-control and a CMH test conditioned on covariate classes is not applicable.

## Inputs

- Binary or additive-encoded marker matrix (rows = SNPs, columns = samples), e.g. avrRpm1_X.txt or BMC.Mode_X_T.txt
- Phenotype labels file (binary matrix, e.g. avrRpm1_Y.txt or BMC.Mode_y_bin.txt)
- Covariate file giving the covariate class for each sample
- Biological-priors gene-interaction file (e.g. genes+edges+3cliques+complexes.txt) defining the pattern poset
- SNP-to-gene interval map restricting each gene to its mapped SNP positions
- Target FWER value (e.g. 0.05 via -f)

## Outputs

- Set of closed, testable patterns, one row each: CMH p-value; interacting genes spanned by the pattern; SNP positions composing the pattern (verbose -O format)
- Updated Tarone significance threshold used for the final FWER-controlled result
- Output file such as out_athaliana

## How to apply

During HOGImine's breadth-first traversal of the pattern poset (expanded from the biological-priors gene-interaction file by recursive_gen, with each gene restricted to its mapped SNP interval, optionally bounded by -d), process each candidate pattern via process_pattern: compute the Cochran-Mantel-Haenszel statistic conditioned on covariate classes, then evaluate the pruning criterion that discards patterns whose minimum achievable p-value cannot drop below the current Tarone threshold. Maintain the testable pattern set T and update the significance threshold incrementally as new testable patterns are discovered, so that FWER is controlled at the level set with -f (e.g. 0.05). Emit only the closed patterns that survive pruning, one row per pattern, matching the -O verbose format: CMH p-value, the interacting genes spanned by the pattern, and the SNP positions composing it. Rationale: Tarone's method guarantees that the adjusted threshold stays valid as it is monotonically tightened during enumeration, avoiding both a full enumeration of the pattern space and the loss of family-wise error control.

## Related tools

- **HOGImine** (Implements the breadth-first pattern enumeration (recursive_gen) and Tarone-pruned processing (process_pattern); invoked as hogimine_additive for additive-encoded markers.) — https://github.com/BorgwardtLab/HOGImine
- **gcc-12** (Compiler used to build HOGImine before running the Tarone-pruned enumeration.)
- **make** (Build tool for compiling HOGImine's src directory.)

## Examples

```
./src/hogimine_additive -i data/athaliana/interactome_0kb/avrRpm1/avrRpm1_X.txt -l data/athaliana/interactome_0kb/avrRpm1/avrRpm1_Y.txt -e data/athaliana/athal_ppi/genes+edges+3cliques+complexes.txt -f 0.05 -O out_athaliana
```

## Evaluation signals

- Running on data/athaliana/interactome_0kb/avrRpm1 with -f 0.05 should produce out_athaliana in the CMH p-value; genes; SNP-positions row format.
- Top reported patterns should include a CMH p-value of order 1e-12 for the AT3G07040, AT3G25070 pair.
- No pattern in the output should have a minimum achievable p-value above the final Tarone threshold (untestable patterns pruned).
- Every emitted pattern should correspond to a connected subgraph in the provided gene-interaction network, and only closed patterns should appear.
- FWER at the target level set by -f (e.g. 0.05) is respected: the number of discovered patterns must not exceed what Tarone's adjustment permits.

## Limitations

- The reference code is compiled with gcc-12; other compilers may or may not work, so build environment mismatches can break the run.
- Maximum SNP interval length per gene is bounded by -d, so patterns spanning intervals longer than this bound are excluded from the search.
- The pruning threshold is incremental: if the initial Tarone threshold is too loose, extra testable patterns are enumerated before it is tightened.
- Only patterns that remain testable under the current Tarone threshold are reported; patterns with p-values just above the threshold are silently pruned even if biologically interesting.

## Evidence

- [methods] HOGImine explores the pattern search space in a breadth-first fashion, and each pattern is processed by the procedure process_pattern, which implements Tarone's method by iteratively updating the significance threshold δtar and the testable pattern set T.: [evidence span withheld — anchor status: not_found]
- [methods] Currently the code is compiled with ```gcc-12```, but other compilers might work as well.: "Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [intro] our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways and protein complexes, to restrict its search space: "our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways and protein complexes, to restrict its search space"
- [readme] Compilation uses make.: "Compilation uses make. Currently the code is compiled with ```gcc-12```, but other compilers might work as well."
- [methods] output file name, with -o in compressed format and with -O in verbose format.: "output file name, with -o in compressed format and with -O in verbose format."
