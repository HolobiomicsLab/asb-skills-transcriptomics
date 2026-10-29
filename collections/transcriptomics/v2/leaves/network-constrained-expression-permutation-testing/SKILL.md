---
name: network-constrained-expression-permutation-testing
description: Use when you need to show that SR-based potency discrimination (e.g.
  pluripotent vs non-pluripotent cells) or its monotonic decrease with developmental
  stage collapses when expression values are reshuffled over the PPI network — i.
when_to_use_negative:
- When the goal is simply to estimate differentiation potency of cells — use SCENT's
  standard SR estimation workflow on the unpermuted expression matrix instead.
- When the expression matrix has no mapping onto the PPI network — unrestricted (non-network-constrained)
  permutation destroys the very network-dependence this ablation is designed to test.
- When the claim to validate involves transcriptomic entropy measures that do not
  use network information (e.g. those used in StemID and SLICE) — this skill tests
  network-constrained signalling entropy, not those measures.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: SCENT
  role: Recomputes signalling entropy (SR) for every cell in each network-permuted
    expression dataset, providing the discrimination and developmental-stage trend
    statistics evaluated under the null
  repo: aet21/SCENT
- name: R
  role: Analysis environment in which the expression matrices, PPI network, and SCENT
    are loaded; permutation generation and aggregation into permutation_significance_results.csv
    are scripted in R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:066a081558be3319e946816c9780d77078fe9158d51100b992c2957abff4af75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# network-constrained-expression-permutation-testing

## Summary

A network-reshuffling null ablation that randomly permutes gene expression values over a fixed PPI network (preserving network structure) to test whether signalling entropy (SR), as computed by SCENT, depends on network-constrained expression patterns rather than expression alone.

## When to use

Use when you need to show that SR-based potency discrimination (e.g. pluripotent vs non-pluripotent cells) or its monotonic decrease with developmental stage collapses when expression values are reshuffled over the PPI network — i.e. a criticality/ablation control for network-dependence of signalling entropy.

## When NOT to use

- When the goal is simply to estimate differentiation potency of cells — use SCENT's standard SR estimation workflow on the unpermuted expression matrix instead.
- When the expression matrix has no mapping onto the PPI network — unrestricted (non-network-constrained) permutation destroys the very network-dependence this ablation is designed to test.
- When the claim to validate involves transcriptomic entropy measures that do not use network information (e.g. those used in StemID and SLICE) — this skill tests network-constrained signalling entropy, not those measures.

## Inputs

- scRNA-Seq gene expression matrix for the Chu et al. dataset with pluripotent vs multipotent/differentiated cell labels
- scRNA-Seq gene expression matrix for the Treutlein et al. dataset annotated with developmental stage
- protein interaction (PPI) network with gene-to-node mapping for the assayed genes
- permutation counts: 100 for Chu, 1,000 for Treutlein

## Outputs

- permutation_significance_results.csv summarizing permutation distributions for both datasets
- per-permutation loss-of-discrimination statistics for the Chu set (pluripotent vs multipotent/differentiated)
- per-permutation developmental-stage trend statistics for the Treutlein set

## How to apply

Load the scRNA-Seq expression matrices (Chu et al. and Treutlein et al.) and the protein interaction network into R for use with SCENT. For the Chu set, generate 100 permuted datasets by randomly reshuffling gene expression values over the network while preserving the network structure; for the Treutlein set, generate 1,000 permuted datasets by the same network-constrained reshuffling. Recompute SR for every cell in each permuted dataset with SCENT. For Chu permutations, test whether pluripotent vs multipotent/differentiated cells can still be discriminated and record the loss of discrimination power per permutation; for Treutlein permutations, test whether the monotonic decrease of SR with developmental stage is abolished and record the trend statistic per permutation. Aggregate the permutation distributions for both datasets and write the summary table to permutation_significance_results.csv.

## Related tools

- **SCENT** (Recomputes signalling entropy (SR) for every cell in each network-permuted expression dataset, providing the discrimination and developmental-stage trend statistics evaluated under the null) — aet21/SCENT
- **R** (Analysis environment in which the expression matrices, PPI network, and SCENT are loaded; permutation generation and aggregation into permutation_significance_results.csv are scripted in R)

## Examples

```
devtools::install_github("aet21/SCENT"); library(SCENT)
```

## Evaluation signals

- On Chu permutations, SR loses its power to discriminate pluripotent from non-pluripotent (multipotent/differentiated) cells relative to unpermuted SR.
- On Treutlein permutations, the monotonic decrease of SR with developmental stage is abolished (trend statistic falls within the permuted null distribution).
- Permutation counts match the reported protocol: 100 permuted datasets for Chu and 1,000 for Treutlein.
- Only expression values are reshuffled; the PPI network structure is preserved across all permutations.
- The aggregated summary table permutation_significance_results.csv is produced and its per-permutation statistics are populated for both datasets.

## Limitations

- The ablation is a null test: acceptance of the finding requires SR to LOSE both pluripotency discrimination (Chu) and the stage-monotonic trend (Treutlein); a surviving signal indicates the result is not driven by network-constrained expression patterns.
- Signalling entropy's behaviour under reshuffling does not transfer to other entropy measures — StemID and SLICE use transcriptomic entropies that do not use network information.
- The protocol is compute-intensive: recomputing per-cell SR with SCENT across 1,000 Treutlein permutations scales with dataset size and permutation count.

## Evidence

- [results] Upon randomly reshuffling gene expression values over the PPI network, signalling entropy lost its power to discriminate pluripotent from non-pluripotent cells: [evidence span withheld — anchor status: not_found]
- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need"
- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
