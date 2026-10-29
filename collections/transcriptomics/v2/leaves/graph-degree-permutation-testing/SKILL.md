---
name: graph-degree-permutation-testing
description: Use when validating that discovered gene modules (e.g. DELVE dynamic
  vs. static modules) have statistically significant experimental evidence of co-regulation,
  by testing network degree against random size-matched gene sets.
when_to_use_negative:
- The modules were not defined from the same feature universe as the random gene sets
  — size-matched sampling requires sampling without replacement from the analyzed
  genes (e.g. the 2000 selected HVGs), not from the whole genome
- You need directional or causal regulation evidence rather than aggregate experimental
  association scores; STRING experimental edges are undirected co-association evidence,
  not causal regulation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: DELVE
  role: Provides the dynamic and static feature modules whose co-regulation is being
    tested; installed via pip as delve-fs
  repo: https://github.com/jranek/delve
- name: Python
  role: Implementation language for fetching STRING scores, building networks, and
    running the permutation test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:4adff8c3fe151a5b97f03fc7b72eb29b26de6f0c8ac3870ef01a470dbe7b407b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - UMAP (uniform manifold approximation and projection)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Graph degree permutation testing for co-regulation significance

## Summary

Assess whether gene modules identified by DELVE (or similar module discovery) show statistically significant experimental evidence of co-regulation by comparing observed average network degree in an experimentally-derived STRING association network against a size-matched null distribution via one-sided permutation testing.

## When to use

Use when validating that discovered gene modules (e.g. DELVE dynamic vs. static modules) have statistically significant experimental evidence of co-regulation, by testing network degree against random size-matched gene sets.

## When NOT to use

- The modules were not defined from the same feature universe as the random gene sets — size-matched sampling requires sampling without replacement from the analyzed genes (e.g. the 2000 selected HVGs), not from the whole genome
- You need directional or causal regulation evidence rather than aggregate experimental association scores; STRING experimental edges are undirected co-association evidence, not causal regulation

## Inputs

- Feature-to-module assignment table from DELVE clustering (e.g. genes assigned to dynamic 0/1/2 and static modules from the 2000-HVG CD8+ T cell matrix)
- Gene lists per module (gene symbols, e.g. mouse genes from GSE131847 LCMV CD8+ T cell data)
- Experimentally-derived association scores from the STRING database for module genes

## Outputs

- Per-module empirical p-values from the one-sided permutation test (e.g. p = 0.001 for dynamic modules, p = 1.0 for the static module)
- Gene association networks (networkx graph objects) per module with nodes as genes and edges weighted by experimental evidence
- Results table such as cd8_module_string_permutation_results.csv containing feature-to-module assignments and per-module permutation p-values

## How to apply

For each gene module (e.g. DELVE dynamic 0: early activation/proliferation, dynamic 1: effector, dynamic 2: memory, and the static module), fetch experimentally-derived association scores from the STRING database for its member genes and build a gene association network with networkx, with nodes as genes and edges weighted by experimental evidence. Compute the observed test statistic T_obs as the average network degree across the module's genes. Generate a null distribution by sampling R=1000 random gene sets without replacement, each size-matched to the module, from the analyzed feature universe, and compute the average degree T_r for each. Compute a one-sided empirical p-value as p = (N + 1)/(R + 1), where N is the number of null statistics at least as extreme as T_obs. Rationale: dynamic modules reflecting core regulatory complexes should show enriched experimental co-regulation evidence, whereas static (noisy) modules should not, which discriminates biologically meaningful modules from noise-driven ones.

## Related tools

- **DELVE** (Provides the dynamic and static feature modules whose co-regulation is being tested; installed via pip as delve-fs) — https://github.com/jranek/delve
- **Python** (Implementation language for fetching STRING scores, building networks, and running the permutation test)

## Examples

```
kh_p = (sum(t_r >= t_obs) + 1) / (1000 + 1); nx.average_degree_connectivity(G_string.subarray(module_genes))
```

## Evaluation signals

- All three dynamic modules (early activation/proliferation, effector, memory) yield p-value = 0.001 with R = 1000 permutations, matching the reported Fig. 5b result
- The static (noisy) module yields p-value = 1.0, i.e. no evidence of co-regulation
- Each null gene set is sampled without replacement and is exactly size-matched to the module being tested
- Empirical p-values are bounded by (N + 1)/(R + 1), so the minimum attainable p-value is 1/(R+1) = 0.001 for R = 1000
- Output CSV (e.g. cd8_module_string_permutation_results.csv) contains a complete feature-to-module mapping plus one p-value per module

## Limitations

- P-value resolution is limited by the number of permutations; with R = 1000 the smallest reportable p-value is 0.001
- Co-regulation evidence depends entirely on STRING experimental association coverage; poorly characterized genes may show low degree regardless of true co-regulation
- As noted in the article, feature selection can greatly bias the interpretation of the underlying cellular trajectory, so module definitions feeding this test should be made carefully

## Evidence

- [other] Fetch experimentally-derived association scores from the STRING database for genes in each module and build gene association networks with networkx, with nodes as genes and edges weighted by experimental evidence.: [evidence span withheld — anchor status: not_found]
- [other] Perform a one-sided permutation test (R = 1000 random gene sets sampled without replacement, size-matched to each module) comparing the observed average network degree T_obs against the null distribution T_r, computing p-value = (N + 1)/(R + 1).: [evidence span withheld — anchor status: not_found]
- [results] Across all three dynamic modules (early activation/proliferation, effector, memory), DELVE identified groups of genes with statistically significant experimental evidence of co-regulation (p-value = 0.001), whereas the static module had little to no evidence of co-regulation (p value = 1.0).: [evidence span withheld — anchor status: not_found]
- [readme] DELVE is available as an open-source python package: "DELVE is available as an open-source python package: https://github.com/jranek/delve"
