---
name: network-hub-gene-detection
description: Use when per-cell scRNA-seq gene expression data has been transformed
  into a CCSN/CNDM degree matrix and you need to detect modules and rank hub genes
  by network degree to pinpoint key regulators of differentiation.
when_to_use_negative:
- When the input is raw gene expression and you need clustering or dimension reduction
  on the expression matrix directly rather than network-degree-derived hub ranking
  — use CNDM-based PCA/t-SNE or clustering methods instead.
- When the goal is to infer causal directionality of gene associations — c-CSN is
  not designed for causal network construction and does not provide directed edges.
- When gene-gene associations should include indirect effects (e.g., exploratory co-expression
  screening) — CSN overestimation may be preferred over the c-CSN direct-association
  filtering.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: c-CSN
  role: Constructs the per-cell conditional cell-specific network (CCSN) and the integrated
    gene-gene interaction degree matrix from which hub genes are ranked by degree
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:68d7253227bdea536d4562363b19f3101a45494670fa4e8627fd0bf95b91376e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# network-hub-gene-detection

## Summary

Identify hub genes by ranking genes by network degree in an integrated conditional cell-specific network (CCSN) built per cell by c-CSN, then extract module hubs such as ONECUT1, ASCL1, and NAR6A1 from the Gokce neural-differentiation scRNA-seq dataset.

## When to use

Use when per-cell scRNA-seq gene expression data has been transformed into a CCSN/CNDM degree matrix and you need to detect modules and rank hub genes by network degree to pinpoint key regulators of differentiation.

## When NOT to use

- When the input is raw gene expression and you need clustering or dimension reduction on the expression matrix directly rather than network-degree-derived hub ranking — use CNDM-based PCA/t-SNE or clustering methods instead.
- When the goal is to infer causal directionality of gene associations — c-CSN is not designed for causal network construction and does not provide directed edges.
- When gene-gene associations should include indirect effects (e.g., exploratory co-expression screening) — CSN overestimation may be preferred over the c-CSN direct-association filtering.

## Inputs

- scRNA-seq gene expression matrix (e.g., Gokce [46] neural-differentiation dataset)
- Per-cell CSN edge sets (edge_xy(k) from ρxy independence tests)
- Conditional gene set (top G largest-degree genes by importance Dz(k))
- Scatter-diagram neighborhood counts (nxyz(k), nxz(k), nyz(k), nz(k)) for estimating ρxy|z
- Significance level α for the Nα quantile threshold on ρ̂xy|z(k)

## Outputs

- Integrated per-cell gene–gene interaction degree matrix C̄k = (1/G)Σg Czg(k)
- Detected network modules from the integrated CCSN
- Hub genes ranked by network degree per module
- Hub-gene table with module memberships and network degrees saved as ccsn_hubs_gokce.csv

## How to apply

First, for each cell in the Gokce scRNA-seq gene expression matrix, construct an ordinary CSN by testing every gene pair with the statistic ρxy and setting edge_xy(k)=1 when independence is rejected. Next, compute importance Dz(k) (the sum of edges to gene z), select the top G largest-degree genes as the conditional gene set, and for every gene pair estimate ρxy|z from scatter-diagram neighborhood counts (nxyz(k), nxz(k), nyz(k), nz(k)), normalizing it to ρ̂xy|z(k) and drawing an edge (ωxy|z(k)=1) when ρ̂xy|z(k) exceeds the Nα quantile of the standard normal at significance level α. Average the G per-conditional-gene CCSNs via C̄k = (1/G)Σg Czg(k) to obtain the integrated gene–gene interaction degree matrix for each cell, then run module detection on this integrated CCSN and rank genes by network degree to identify hub genes, verifying that ONECUT1, ASCL1, and NAR6A1 emerge as highlighted hubs. The rationale is that c-CSN measures direct associations between genes by eliminating indirect associations, so degree in the CCSN reflects genuine regulatory centrality rather than indirect co-expression.

## Related tools

- **c-CSN** (Constructs the per-cell conditional cell-specific network (CCSN) and the integrated gene-gene interaction degree matrix from which hub genes are ranked by degree) — https://github.com/LinLi-0909/c-CSN

## Examples

```
git clone https://github.com/LinLi-0909/c-CSN && python c_csn.py --input gokce_expression.csv --top_g 10 --alpha 0.05 --output ccsn_hubs_gokce.csv
```

## Evaluation signals

- The integrated CCSN degree matrix equals the average of the G per-conditional-gene CCSNs: C̄k = (1/G)Σg Czg(k) for every cell k.
- Every edge ωxy|z(k)=1 satisfies ρ̂xy|z(k) > Nα quantile of the standard normal at the chosen significance level α.
- The conditional gene set size equals the specified top-G largest-degree genes by importance Dz(k) in the initial CSN.
- ONECUT1, ASCL1, and NAR6A1 are present among the top-degree hub genes of their respective detected modules in the Gokce dataset.
- The output ccsn_hubs_gokce.csv contains, for each hub gene, both its module membership and its network degree.

## Limitations

- c-CSN costs G-fold more computation than ordinary CSN, and no parallel computation scheme is provided; the authors state 'a parallel computation scheme is desired to reduce the computation time'.
- c-CSN is not designed to construct causal gene association networks; the directions of gene associations cannot be obtained.
- The method requires choosing the number of conditional genes G, which affects the stability of the integrated CCSN and thus the hub-gene ranking.

## Evidence

- [methods] importance Dz(k) and select top-G conditional genes: [evidence span withheld — anchor status: not_found]
- [methods] CCSN edge drawing threshold: [evidence span withheld — anchor status: not_found]
- [methods] integration of CCSNs: [evidence span withheld — anchor status: not_found]
- [abstract] c-CSN measures direct associations: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [discussion] no parallel computation: "a parallel computation scheme is desired to reduce the computation time"
