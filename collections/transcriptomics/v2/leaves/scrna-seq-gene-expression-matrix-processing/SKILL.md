---
name: scrna-seq-gene-expression-matrix-processing
description: Use when the input is a normalized single-cell gene expression matrix
  (genes × cells) and the goal is direct gene–gene association networks per cell,
  overcoming CSN's overestimation of indirect associations.
when_to_use_negative:
- The goal is causal network construction or obtaining directions of gene associations
  — the article states c-CSN is not designed for causal gene association networks
  and directions cannot be obtained
- The input is not single-cell expression data, e.g. bulk RNA-seq with no per-cell
  resolution for building cell-specific networks
- Runtime is severely constrained, since the conditional testing incurs a G-fold computational
  cost increase with no parallel scheme provided
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: c-CSN
  role: Core method implementing CSN construction, conditional-gene selection, CCSN
    hypothesis testing (ρ̂xy|z vs Nα), and CNDM computation
  repo: https://github.com/LinLi-0909/c-CSN
- name: PCA
  role: Linear dimension reduction applied to the CNDM output for downstream visualization
- name: t-SNE
  role: Nonlinear dimension reduction applied to the CNDM output for downstream visualization
- name: SC3
  role: Clustering method applied to the c-CSN-transformed degree matrices
- name: SIMLR
  role: Clustering method applied to the c-CSN-transformed degree matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:32a9004c046ed9ad0700070bb5b4045b582f382e7474e9696564e2cb46278f44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - PCA
  - SC3
  - SIMLR
  - c-CSN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-gene-expression-matrix-processing

## Summary

Transforms a normalized scRNA-seq gene expression matrix into per-cell conditional cell-specific networks (CCSNs) by filtering indirect gene–gene associations via conditional independence testing, producing a sparse conditional network degree matrix (CNDM) usable for downstream clustering and differentiation-potency analysis.

## When to use

Use when the input is a normalized single-cell gene expression matrix (genes × cells) and the goal is direct gene–gene association networks per cell, overcoming CSN's overestimation of indirect associations.

## When NOT to use

- The goal is causal network construction or obtaining directions of gene associations — the article states c-CSN is not designed for causal gene association networks and directions cannot be obtained
- The input is not single-cell expression data, e.g. bulk RNA-seq with no per-cell resolution for building cell-specific networks
- Runtime is severely constrained, since the conditional testing incurs a G-fold computational cost increase with no parallel scheme provided

## Inputs

- Normalized scRNA-seq gene expression matrix (genes × cells), e.g. the Buettner mESC dataset
- Significance level α for the Nα quantile threshold of the standard normal distribution
- Number of conditional genes G (top-degree hub genes selected from the CSN)

## Outputs

- Per-cell CSN and per-cell CCSN edge weight matrices
- Per-cell gene–gene interaction degree matrix C-k from integrating the G CCSNs (Equation 9)
- Conditional network degree matrix (CNDM), an m × n matrix across all cells

## How to apply

First build the baseline CSN for each cell by testing every gene pair with the ρxy statistic against the Nα quantile of the standard normal distribution at the chosen significance level α. Then compute the conditional-gene importance Dz(k) as the sum of CSN edge weights (Equation 8) and select the top G largest 'importance' genes as the conditional gene set. For each cell and each conditional gene z, construct the CCSN by testing conditional independence with the normalized statistic ρ̂xy|z(k), estimated numerically from scatter diagram neighborhoods in 3D expression space, and set edge weight ωxy|z(k)=1 when ρ̂xy|z(k) exceeds Nα (Equation 7). Finally integrate the G CCSNs per cell via C-k=(1/G)ΣCzg(k) (Equation 9) to obtain degrees of the gene–gene interaction network, and assemble the m × n conditional network degree matrix (CNDM) across cells.

## Related tools

- **c-CSN** (Core method implementing CSN construction, conditional-gene selection, CCSN hypothesis testing (ρ̂xy|z vs Nα), and CNDM computation) — https://github.com/LinLi-0909/c-CSN
- **PCA** (Linear dimension reduction applied to the CNDM output for downstream visualization)
- **t-SNE** (Nonlinear dimension reduction applied to the CNDM output for downstream visualization)
- **SC3** (Clustering method applied to the c-CSN-transformed degree matrices)
- **SIMLR** (Clustering method applied to the c-CSN-transformed degree matrices)

## Examples

```
git clone https://github.com/LinLi-0909/c-CSN && python c_csn.py --expression buettner_mesc_normalized.csv --alpha 0.05 --G 10 --output cndm.csv
```

## Evaluation signals

- Mean per-cell edge counts of the CCSN are substantially lower than those of the CSN across all cells, reproducing the paper's reported sparsity finding (result_ccsn_sparser_csn)
- For every gene pair, ωxy|z(k)=1 only when the normalized statistic ρ̂xy|z(k) exceeds Nα; pairs at or below Nα must have zero weight
- The G conditional genes per cell correspond to the top-G largest Dz(k) importance scores computed as sums of CSN edge weights (Equation 8)
- The integrated degree matrix C-k equals (1/G)Σg=1G Czg(k) for each cell, and the CNDM has shape m × n (genes × cells) matching the input expression matrix
- Downstream clustering on the CNDM yields higher ARI than clustering on the original gene expression matrix (GEM) across benchmark datasets such as Buettner, Kim, Kolodziejczyk, and Chu-time

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state 'a parallel computation scheme is desired to reduce the computation time'
- c-CSN is not designed to construct causal gene association networks, and the directions of the gene associations cannot be obtained — left as future work
- Requires choice of significance level α and number of conditional genes G, which affect sparsity and computational cost

## Evidence

- [intro] CSN overestimation rationale: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [methods] Conditional gene selection: [evidence span withheld — anchor status: not_found]
- [methods] CCSN construction per cell: "CCSN Czg(k) is obtained for cell k given conditional gene zg"
- [methods] Integration into degree matrix: [evidence span withheld — anchor status: not_found]
- [methods] CNDM output: "an m × n matrix conditional network degree matrix (CNDM) is obtained"
- [abstract] Repository availability: "c-CSN is publicly available at https://github.com/LinLi-0909/c-CSN."
- [discussion] No parallel scheme limitation: "a parallel computation scheme is desired to reduce the computation time"
