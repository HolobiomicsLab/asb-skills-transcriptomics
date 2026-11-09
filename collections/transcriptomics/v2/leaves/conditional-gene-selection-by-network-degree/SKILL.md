---
name: conditional-gene-selection-by-network-degree
description: 'Use when constructing c-CSN networks from scRNA-seq data: after building
  each cell''s ordinary CSN, choose the top-G largest-degree genes as conditional
  genes before testing conditional independence of every gene pair.'
when_to_use_negative:
- 'When the goal is a causal gene regulatory network: the article states c-CSN cannot
  obtain directions of gene associations and causal construction is left as future
  work'
- When no per-cell CSN is available, since importance Dz(k) is defined as the sum
  of CSN edge weights and requires the baseline CSN first
- 'When computational budget is severely limited and datasets are large: the G conditional
  tests multiply runtime G-fold and the authors note no parallel scheme is provided'
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0202
tools:
- name: c-CSN
  role: Implements CSN construction, importance-based conditional gene selection,
    conditional independence testing (ρ̂xy|z(k) vs Nα), and averaging of the G per-conditional-gene
    CCSNs into CNDM
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:84bb906dc616ca8c16892b36a1ab0c119766441025970c27e4f793a1de80b77e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# conditional-gene-selection-by-network-degree

## Summary

Selects the top G largest-degree ('importance') genes from each cell's CSN as the conditional gene set used to build conditional cell-specific networks (CCSNs), pruning indirect associations by conditioning on hub genes.

## When to use

Use when constructing c-CSN networks from scRNA-seq data: after building each cell's ordinary CSN, choose the top-G largest-degree genes as conditional genes before testing conditional independence of every gene pair.

## When NOT to use

- When the goal is a causal gene regulatory network: the article states c-CSN cannot obtain directions of gene associations and causal construction is left as future work
- When no per-cell CSN is available, since importance Dz(k) is defined as the sum of CSN edge weights and requires the baseline CSN first
- When computational budget is severely limited and datasets are large: the G conditional tests multiply runtime G-fold and the authors note no parallel scheme is provided

## Inputs

- Normalized scRNA-seq gene expression matrix (cells × genes), e.g. Buettner mESC or Gokce neural-differentiation datasets
- Per-cell ordinary CSN adjacency (edge weights from ρxy hypothesis tests)

## Outputs

- Per-cell list of top-G conditional genes ranked by importance Dz(k) (Equation 8)
- Per-conditional-gene CCSN adjacency matrices Czg(k) with binary edge weights ωxy|z(k)
- Integrated per-cell gene–gene interaction degree matrix C̄k (Equation 9)

## How to apply

For each cell k, first build the ordinary CSN by testing every gene pair with the ρxy statistic against the Nα quantile of the standard normal at significance level α, then compute each gene's importance Dz(k) as the sum of its CSN edge weights (Equation 8). Rank genes by Dz(k) and select the top G ≥ 1 largest-degree genes as the conditional gene set for that cell. For every gene pair (x, y), estimate the conditional statistic ρxy|z(k) via scatter-diagram neighborhood counts (nxyz(k), nxz(k), nyz(k), nz(k)) in 3D expression space, normalize it to ρ̂xy|z(k), and set the edge ωxy|z(k)=1 when ρ̂xy|z(k) exceeds Nα (Equation 7). Repeat per conditional gene to obtain G CCSNs, Czg(k), for cell k, and integrate them by averaging, C̄k = (1/G)Σg Czg(k) (Equation 9), into a gene–gene interaction degree matrix. The rationale is that high-degree CSN genes carry the most conditional information for removing indirect (overestimated) associations, which is why the selection is restricted to hub genes of the CSN.

## Related tools

- **c-CSN** (Implements CSN construction, importance-based conditional gene selection, conditional independence testing (ρ̂xy|z(k) vs Nα), and averaging of the G per-conditional-gene CCSNs into CNDM) — https://github.com/LinLi-0909/c-CSN

## Evaluation signals

- The selected conditional gene set consists of exactly the top G genes ranked by Dz(k) = sum of CSN edge weights per cell (Equation 8)
- Each binary edge ωxy|z(k)=1 is set only when the normalized statistic ρ̂xy|z(k) exceeds the Nα quantile of the standard normal at significance level α (Equation 7)
- Averaging over the G conditional CCSNs yields C̄k = (1/G)Σg Czg(k) with CNDM dimension m × n matching the gene expression matrix
- Downstream CCSN edge counts are substantially lower than the baseline CSN's, reproducing the reported per-cell sparsity finding (result_ccsn_sparser_csn)
- On the Gokce dataset, module detection on the integrated CCSN highlights ONECUT1, ASCL1, and NAR6A1 as hub genes

## Limitations

- The G-fold conditional testing increases computation time by a factor of G and no parallel computation scheme is provided; the authors state a parallel scheme is desired
- The conditional gene set size G ≥ 1 is a user-chosen parameter that controls the trade-off between computation cost and removal of indirect associations
- Directions of gene associations cannot be obtained and causal network construction is not supported

## Evidence

- [methods] Selection of conditional genes via top-degree hub genes in CSN: [evidence span withheld — anchor status: not_found]
- [methods] CCSN construction per conditional gene: "CCSN Czg(k) is obtained for cell k given conditional gene zg"
- [methods] Integration of per-conditional-gene CCSNs: [evidence span withheld — anchor status: not_found]
- [intro] CSN overestimation motivation: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations"
- [discussion] Computational cost of G conditional tests: "a parallel computation scheme is desired to reduce the computation time"
