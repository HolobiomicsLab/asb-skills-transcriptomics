---
name: gene-gene-association-network-construction
description: Use when you need per-cell, direct gene-gene association networks from
  a scRNA-seq expression matrix, e.g. to remove CSN's indirect-association overestimation
  or to identify hub genes in differentiation modules.
when_to_use_negative:
- When causal direction of gene associations is required — c-CSN is not designed to
  construct causal gene association networks and cannot obtain edge directions
- When a single population-level static network suffices and per-cell resolution or
  removal of indirect associations is not needed
- 'When computational cost is prohibitive: c-CSN incurs a G-fold cost increase and
  no parallel computation scheme is provided in the paper'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: c-CSN
  role: 'Primary method: constructs the conditional cell-specific network for each
    cell by conditional independence testing on scRNA-seq data'
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:df0205d912436df4b0b97f655d0ff904da2e96b1ee0e30351363358d756fba54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-gene-association-network-construction

## Summary

Constructs a conditional cell-specific network (CCSN) from scRNA-seq gene expression data per cell using the c-CSN method, testing conditional independence of gene pairs to retain only direct gene-gene associations and eliminating indirect associations that inflate ordinary CSNs.

## When to use

Use when you need per-cell, direct gene-gene association networks from a scRNA-seq expression matrix, e.g. to remove CSN's indirect-association overestimation or to identify hub genes in differentiation modules.

## When NOT to use

- When causal direction of gene associations is required — c-CSN is not designed to construct causal gene association networks and cannot obtain edge directions
- When a single population-level static network suffices and per-cell resolution or removal of indirect associations is not needed
- When computational cost is prohibitive: c-CSN incurs a G-fold cost increase and no parallel computation scheme is provided in the paper

## Inputs

- scRNA-seq gene expression matrix (e.g. the Gokce [46] neural-differentiation dataset)
- Significance level α for the conditional independence test
- Number of conditional genes G (top-degree hub genes selected from the ordinary CSN)

## Outputs

- Per-cell CCSN / integrated gene-gene interaction degree matrix C̄k (Equation 9)
- Conditional network degree matrix (CNDM, m × n)
- Module membership and network degree rankings of hub genes (e.g. ccsn_hubs_gokce.csv)

## How to apply

For each cell k, first build an ordinary CSN by testing every gene pair with the statistic ρxy and setting edge_xy(k)=1 when independence is rejected. Compute importance Dz(k) (sum of edges to gene z), select the top G largest-degree genes as conditional genes, and for each pair estimate ρxy|z from scatter-diagram neighborhood counts (nxyz(k), nxz(k), nyz(k), nz(k)); normalize to ρ̂xy|z(k) and draw edge ωxy|z(k)=1 when ρ̂xy|z(k) exceeds the Nα quantile of the standard normal at significance level α. Average the G per-conditional-gene CCSNs via C̄k = (1/G)Σg Czg(k) (Equation 9) to obtain the integrated gene-gene interaction degree matrix. Downstream, run module detection on the integrated CCSN and rank genes by network degree to extract hub genes. The rationale is that conditioning on hub genes removes indirect associations, so the resulting degree matrix is direct and robust and can replace the gene expression matrix in standard analyses.

## Related tools

- **c-CSN** (Primary method: constructs the conditional cell-specific network for each cell by conditional independence testing on scRNA-seq data) — https://github.com/LinLi-0909/c-CSN

## Examples

```
git clone https://github.com/LinLi-0909/c-CSN  # then run the c-CSN per-cell CCSN construction on the Gokce expression matrix
```

## Evaluation signals

- Edge insertion must be gated by ρ̂xy|z(k) exceeding the Nα quantile of the standard normal at significance level α — verify all edges meet this criterion
- The integrated degree matrix equals the average of the G per-conditional-gene CCSNs, C̄k = (1/G)Σg Czg(k); check the averaging identity holds
- The selected conditional gene set must consist of the top G largest-importance (degree) genes Dz(k) from the ordinary CSN
- Networks should be sparser than ordinary CSNs, since indirect associations are eliminated by conditioning
- For the Gokce dataset task, ONECUT1, ASCL1, and NAR6A1 should emerge as highlighted hub genes of their modules after degree-based ranking

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel scheme is desired to reduce computation time
- c-CSN is not designed to construct causal gene association networks; directions of gene associations cannot be obtained
- CSN (and by extension the conditional construction) overestimates gene-gene associations if conditioning is skipped, since indirect associations inflate network density

## Evidence

- [abstract] c-CSN method can measure the direct associations between genes by eliminating the indirect associations.: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [methods] we choose the top GG≥1 largest 'importance' genes as the conditional genes: [evidence span withheld — anchor status: not_found]
- [methods] We numerically estimate the value of ρxy|z by making a scatter diagram based on gene expression data: "We numerically estimate the value of ρxy|z by making a scatter diagram based on gene expression data"
- [methods] C-k=1G∑g=1GCzg(k)=cijk to represent the degrees of gene–gene interaction network of cell k: [evidence span withheld — anchor status: not_found]
- [discussion] a parallel computation scheme is desired to reduce the computation time: "a parallel computation scheme is desired to reduce the computation time"
