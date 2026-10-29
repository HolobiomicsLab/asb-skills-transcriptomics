---
name: archetype-number-sweep-benchmarking
description: Use when you need to determine how many archetype components suffice
  to recover all ground-truth cell-identity patterns, e.g. synthetic scRNA-seq data
  with known activity patterns evaluated across noise levels.
when_to_use_negative:
- Real datasets without ground-truth activity patterns — pattern coverage (fraction
  of the 13 true patterns recovered) cannot be computed
- Analyses that need a single fixed-k decomposition rather than a k-sweep comparison
  of decomposition methods
- Downstream steps that operate on already-pruned, nonredundant archetype sets (e.g.
  ACTIONet network construction) rather than on raw per-k decomposition traces
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ACTIONet
  role: Provides the randomized SVD-based reduction step (an.pp.reduce_kernel) and
    the nested ACTION archetype sweep over k = 2..30 used to build the SPA+ACTION
    coverage curves
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTION
  role: Reference implementation of runACTION (nested k_min..k_max sweep) and runSPA
    (plain SPA per k) used to count nontrivial archetype columns at each k
  repo: https://github.com/shmohammadi86/ACTION
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:d484025e5f83e3ff08664b73b92f0becbc287bd163c413ecb22f1e7b169c7a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTION
  - ACTIONet
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# archetype-number-sweep-benchmarking

## Summary

A benchmarking procedure that sweeps the number of hidden archetype factors (k = 2..30) for SPA and SPA+ACTION decompositions on a synthetic scRNA-seq dataset with 13 known activity patterns, measuring identity-pattern coverage versus k at multiple noise levels.

## When to use

Use when you need to determine how many archetype components suffice to recover all ground-truth cell-identity patterns, e.g. synthetic scRNA-seq data with known activity patterns evaluated across noise levels.

## When NOT to use

- Real datasets without ground-truth activity patterns — pattern coverage (fraction of the 13 true patterns recovered) cannot be computed
- Analyses that need a single fixed-k decomposition rather than a k-sweep comparison of decomposition methods
- Downstream steps that operate on already-pruned, nonredundant archetype sets (e.g. ACTIONet network construction) rather than on raw per-k decomposition traces

## Inputs

- Synthetic scRNA-seq dataset from ref. 12 (20 instances of cells with 13 activity patterns), at multiple noise levels
- Gene x cell expression matrix in Matrix Market format with gene_names.txt and sample_annotations.txt (as used by the ACTION demo layout)

## Outputs

- Coverage curves (fraction of the 13 true activity patterns recovered vs k) per noise level for SPA+ACTION and SPA alone, e.g. supp_fig1bcd_coverage_curves.png
- Counts of nontrivial archetype columns per k for each decomposition
- Saturation point (smallest k at which all existing identity patterns are discovered) per method per noise level

## How to apply

Load the synthetic scRNA-seq benchmark dataset from ref. 12 (20 instances of cells with 13 activity patterns) and, for each noise level, apply the randomized SVD-based reduction step (an.pp.reduce_kernel) to obtain the low-rank kernel space used by ACTION. Run the ACTION decomposition over the archetype-number sweep k = 2..30, exploiting its nested/ranged formulation so that all solutions for k <= 30 come from a single run (in the ACTION reference implementation: runACTION(S_r, k_min, k_max, thread_no)), and count nontrivial archetype columns per k. Independently run plain SPA-only archetypal analysis for each k in the same sweep (runSPA) and count its nontrivial columns. Compute pattern coverage (fraction of the 13 true activity patterns recovered) as a function of k for SPA+ACTION versus SPA alone at each noise level, and plot the coverage curves (Supplementary Fig. 1b-d). Validate that the nontrivial-column count plateaus with increasing k, that SPA+ACTION recovers all 13 patterns at every noise level, and that ACTION-based coverage saturates at a smaller k than SPA alone.

## Related tools

- **ACTIONet** (Provides the randomized SVD-based reduction step (an.pp.reduce_kernel) and the nested ACTION archetype sweep over k = 2..30 used to build the SPA+ACTION coverage curves) — https://github.com/shmohammadi86/ACTIONet
- **ACTION** (Reference implementation of runACTION (nested k_min..k_max sweep) and runSPA (plain SPA per k) used to count nontrivial archetype columns at each k) — https://github.com/shmohammadi86/ACTION

## Examples

```
S_r <- reduceGeneExpression(expression, dim = 30, method = 1); out <- runACTION(S_r, k_min = 2, k_max = 20, thread_no = 8)
```

## Evaluation signals

- The number of nontrivial archetype columns plateaus as k increases (Supplementary Fig. 1)
- SPA+ACTION coverage reaches all 13 identity patterns at every tested noise level
- ACTION-based coverage saturates (all patterns discovered) at a smaller k than SPA alone
- For each k, C columns are convex combinations of cells and H columns are convex mixtures of archetypes (S ~ C * H factorisation)
- Coverage curves for all noise levels are saved and match the Supplementary Fig. 1b-d layout

## Limitations

- The synthetic benchmark dataset is cited only by reference number (ref. 12) without accessions or download links
- No software/tool versions are stated for ACTIONet, R, or Python (CMake 3.19 or newer is the only version given)
- Supplementary Fig. 1 is referenced but not included in the provided text, so exact plateau values must be reproduced rather than checked against the figure

## Evidence

- [discussion] Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
- [discussion] Randomized SVD reduction step: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [discussion] Plateau of nontrivial columns: "the number of nontrivial columns plateaus (Supplementary Fig. 1)"
- [readme] ACTION R sweep invocation: [evidence span withheld — anchor status: external_doc]
- [readme] SPA and AA low-level routines: [evidence span withheld — anchor status: external_doc]
