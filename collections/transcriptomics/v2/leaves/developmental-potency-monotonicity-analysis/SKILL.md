---
name: developmental-potency-monotonicity-analysis
description: 'Use when validating that a network-based potency metric like signalling
  entropy (SR) genuinely relies on the PPI network: the goal is a null ablation showing
  SR loses pluripotency discrimination and stage monotonicity after network-constrained
  reshuffling.'
when_to_use_negative:
- Input is bulk rather than single-cell expression data and you only need the bulk-level
  SR/plasticity estimate rather than a network-dependence ablation
- You want an alternative ordering of cells in pseudo-time (e.g. Monocle or MPath)
  rather than testing the network dependence of signalling entropy
- You are using entropy measures without network information (StemID, SLICE) — this
  ablation is specific to network-constrained signalling entropy
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: Recomputes signalling entropy (SR) for every cell in each permuted dataset
    to evaluate loss of pluripotency discrimination and stage monotonicity
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment for running SCENT, generating the 100/1,000 network-constrained
    permutations, and writing permutation_significance_results.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:b54a52e442fad6765eb4d23b456010efaff6771ecb00d4232a70e7bd8e8c32ce
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# developmental-potency-monotonicity-analysis

## Summary

A network-constrained permutation ablation that tests whether signalling entropy (SR) depends on network-constrained expression patterns: gene expression values are randomly reshuffled over the PPI network in scRNA-Seq data and SR is recomputed to check whether pluripotency discrimination and monotonic decrease with developmental stage are abolished.

## When to use

Use when validating that a network-based potency metric like signalling entropy (SR) genuinely relies on the PPI network: the goal is a null ablation showing SR loses pluripotency discrimination and stage monotonicity after network-constrained reshuffling.

## When NOT to use

- Input is bulk rather than single-cell expression data and you only need the bulk-level SR/plasticity estimate rather than a network-dependence ablation
- You want an alternative ordering of cells in pseudo-time (e.g. Monocle or MPath) rather than testing the network dependence of signalling entropy
- You are using entropy measures without network information (StemID, SLICE) — this ablation is specific to network-constrained signalling entropy

## Inputs

- Chu et al. scRNA-Seq expression matrix (pluripotent vs. multipotent/differentiated cells)
- Treutlein et al. scRNA-Seq expression matrix ordered by developmental stage
- Protein interaction network (connectome) compatible with SCENT
- SCENT package installed in R

## Outputs

- Permutation distributions of SR discrimination power for the Chu set (100 permutations)
- Permutation distributions of the SR-vs-developmental-stage trend statistic for the Treutlein set (1,000 permutations)
- permutation_significance_results.csv summary table

## How to apply

Load the Chu et al. and Treutlein et al. scRNA-Seq expression matrices plus the protein interaction network into R for use with SCENT. For the Chu set, generate 100 permuted datasets by randomly reshuffling gene expression values over the network while preserving network structure; for the Treutlein set, generate 1,000 such permutations. Recompute SR for every cell in each permuted dataset with SCENT and assess whether pluripotent vs. multipotent/differentiated cells can still be discriminated (Chu) and whether the monotonic decrease of SR with developmental stage is abolished (Treutlein), recording the loss-of-discrimination power and trend statistic per permutation. Aggregate the permutation distributions for both datasets and write the summary table to permutation_significance_results.csv. The rationale is that SR's accuracy is driven by a subtle positive correlation between the transcriptome and connectome, so destroying the gene-network correspondence should destroy SR's performance.

## Related tools

- **SCENT** (Recomputes signalling entropy (SR) for every cell in each permuted dataset to evaluate loss of pluripotency discrimination and stage monotonicity) — https://github.com/aet21/SCENT
- **R** (Environment for running SCENT, generating the 100/1,000 network-constrained permutations, and writing permutation_significance_results.csv)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- On Chu permutations, SR loses its power to discriminate pluripotent from multipotent/differentiated cells (discrimination statistic collapses toward null across the 100 permutations)
- On Treutlein permutations, the monotonic decrease of SR with developmental stage is abolished (trend statistic no longer consistently decreasing across the 1,000 permutations)
- Network structure is preserved during reshuffling — only gene expression values are randomly reshuffled over the fixed PPI network
- Permutation distributions for both datasets are aggregated into permutation_significance_results.csv without missing per-permutation statistics
- Untouched (unpermuted) SCENT runs reproduce the reported finding that signalling entropy discriminates pluripotent from multipotent and differentiated cells

## Limitations

- The null reshuffling destroys the transcriptome-connectome correspondence, so it tests network dependence specifically, not all possible confounders of SR
- Entropy-based measures that do not use network information (StemID, SLICE) behave differently and are not covered by this ablation
- SCENT estimates potency without assuming prior biological knowledge such as marker expression or timepoint; results can be affected by high dropout rates in scRNA-Seq data

## Evidence

- [abstract] Signalling entropy provides a more accurate and robust potency estimate than other entropy-based measures, driven in part by a subtle positive correlation between the transcriptome and connectome: [evidence span withheld — anchor status: not_found]
- [discussion] SR discriminates pluripotent cells without feature selection: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] SCENT estimates potency without prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
