---
name: scna-seq-null-model-generation
description: 'Use when you must demonstrate that signalling entropy (SR) depends on
  network-constrained expression: i.'
when_to_use_negative:
- The goal is to estimate differentiation potency of cells in the original (unpermuted)
  data rather than to test SR's dependence on network structure
- The input data lacks an associated protein interaction network, since the reshuffling
  is explicitly network-constrained
- The analysis concerns transcriptomic entropy measures (e.g., those used in StemID
  or SLICE) that do not use network information and thus cannot be ablated by network-constrained
  reshuffling
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: Recomputes signalling entropy (SR) for every cell in each permuted dataset
    to test loss of pluripotency discrimination and of monotonic decrease with developmental
    stage
  repo: aet21/SCENT
- name: R
  role: Implementation environment for the reshuffling procedure, SCENT execution,
    and aggregation of permutation results into permutation_significance_results.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:9a632e3fa912349e38beaa3988b3666798481929b53957f8bf1f08230aff3916
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scna-seq-null-model-generation

## Summary

Generates network-constrained null datasets by randomly reshuffling single-cell gene expression values over the protein-protein interaction (PPI) network, then recomputes signalling entropy (SR) with SCENT to test whether the pluripotency discrimination and developmental-stage trends of SR depend on network-constrained expression patterns.

## When to use

Use when you must demonstrate that signalling entropy (SR) depends on network-constrained expression: i.e., test whether SR loses its ability to discriminate pluripotent from non-pluripotent cells, or its consistent decrease with developmental stage, under expression reshuffling over the PPI network.

## When NOT to use

- The goal is to estimate differentiation potency of cells in the original (unpermuted) data rather than to test SR's dependence on network structure
- The input data lacks an associated protein interaction network, since the reshuffling is explicitly network-constrained
- The analysis concerns transcriptomic entropy measures (e.g., those used in StemID or SLICE) that do not use network information and thus cannot be ablated by network-constrained reshuffling

## Inputs

- Chu et al. scRNA-Seq expression matrix (pluripotent vs. multipotent/differentiated cells)
- Treutlein et al. scRNA-Seq expression matrix (developmental time-course)
- Protein interaction network compatible with SCENT

## Outputs

- 100 network-constrained permuted Chu datasets with per-permutation SR values and discrimination assessment
- 1,000 network-constrained permuted Treutlein datasets with per-permutation SR values and trend statistics
- permutation_significance_results.csv summarizing permutation distributions for both datasets

## How to apply

Load the Chu et al. and/or Treutlein et al. scRNA-Seq expression matrices together with the protein interaction network into R for use with SCENT. For the Chu set, generate 100 permuted datasets by randomly reshuffling gene expression values over the PPI network while preserving network structure; for the Treutlein set, generate 1,000 permuted datasets by the same network-constrained reshuffling procedure. Recompute SR for every cell in each permuted dataset with SCENT and, per permutation, record whether pluripotent vs. multipotent/differentiated cells can still be discriminated (Chu) and whether the monotonic decrease of SR with developmental stage is abolished (Treutlein). Aggregate the permutation distributions for both datasets into a summary table written to permutation_significance_results.csv. The rationale is that SR is driven in part by a subtle positive correlation between the transcriptome and connectome, so destroying that coupling should eliminate SR's biological signal if SR is genuinely network-dependent.

## Related tools

- **SCENT** (Recomputes signalling entropy (SR) for every cell in each permuted dataset to test loss of pluripotency discrimination and of monotonic decrease with developmental stage) — aet21/SCENT
- **R** (Implementation environment for the reshuffling procedure, SCENT execution, and aggregation of permutation results into permutation_significance_results.csv)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- On Chu permutations, SR loses its power to discriminate pluripotent from multipotent/differentiated cells in the permuted datasets, matching the reported finding
- On Treutlein permutations, SR no longer shows a consistent (monotonic) decrease with developmental stage, with the trend statistic per permutation recorded
- The number of permutations matches the protocol: 100 for Chu and 1,000 for Treutlein, all generated by reshuffling expression values over the PPI network while preserving network structure
- A summary table permutation_significance_results.csv is produced aggregating the permutation distributions for both datasets
- The null reshuffling is accepted as reproducing the reported finding only if both the Chu discrimination loss and the Treutlein trend abolition are observed, consistent with SR's dependence on network-constrained expression patterns

## Limitations

- The null is valid only for network-based entropy measures; transcriptomic entropies used in StemID and SLICE do not use network information and are not ablated by this procedure
- The test is specific to the Chu and Treutlein scRNA-Seq datasets as described; applying it to other datasets requires new permutation counts and validation criteria

## Evidence

- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [abstract] SR robustness is driven by transcriptome-connectome coupling: [evidence span withheld — anchor status: not_found]
- [readme] SCENT installation in R: "library(devtools)
devtools::install_github("aet21/SCENT")"
