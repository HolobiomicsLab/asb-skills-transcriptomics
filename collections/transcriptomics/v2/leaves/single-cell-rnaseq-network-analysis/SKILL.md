---
name: single-cell-rnaseq-network-analysis
description: Use when a scRNA-seq gene expression matrix must be transformed into
  per-cell direct gene-gene association networks, or when cell-specific hub genes/module
  structure (e.g., Gokce neural-differentiation hubs) need to be identified.
when_to_use_negative:
- The goal is a causal gene association network or directed edges — c-CSN is explicitly
  not designed to construct causal networks and cannot obtain directions of gene associations
- Input is bulk RNA-seq or aggregated expression with no single-cell resolution —
  c-CSN constructs one network per cell and requires per-cell data
- Only indirect-correction-free co-expression is needed at population level rather
  than cell-specific direct associations (ordinary CSN may suffice, though it overestimates
  associations)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: c-CSN
  role: Core method that constructs the conditional cell-specific network (CCSN) per
    cell, testing direct gene-gene associations via conditional independence and eliminating
    indirect associations
  repo: https://github.com/LinLi-0909/c-CSN
- name: SC3
  role: Optional downstream clustering method applicable to the c-CSN-transformed
    degree matrix (CNDM)
- name: SIMLR
  role: Optional downstream clustering method applicable to the c-CSN-transformed
    degree matrix (CNDM)
- name: Monocle
  role: Optional trajectory inference method for analyzing the transformed gene-gene
    association data along differentiation trajectories
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:ab5885f121d7d44c4185da4d8257d86512c5a4fbca7357a60f8bf850beba05e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - Monocle
  - SC3
  - SIMLR
  - c-CSN
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-rnaseq-network-analysis

## Summary

Constructs a conditional cell-specific network (CCSN) from scRNA-seq gene expression matrices using c-CSN, which tests direct gene-gene associations per cell by eliminating indirect associations, then supports downstream analyses such as hub-gene identification of differentiation modules (e.g., ONECUT1, ASCL1, NAR6A1 in the Gokce dataset).

## When to use

Use when a scRNA-seq gene expression matrix must be transformed into per-cell direct gene-gene association networks, or when cell-specific hub genes/module structure (e.g., Gokce neural-differentiation hubs) need to be identified.

## When NOT to use

- The goal is a causal gene association network or directed edges — c-CSN is explicitly not designed to construct causal networks and cannot obtain directions of gene associations
- Input is bulk RNA-seq or aggregated expression with no single-cell resolution — c-CSN constructs one network per cell and requires per-cell data
- Only indirect-correction-free co-expression is needed at population level rather than cell-specific direct associations (ordinary CSN may suffice, though it overestimates associations)

## Inputs

- scRNA-seq gene expression matrix (cells × genes), e.g., the Gokce [46] dataset
- Significance level α for the conditional independence test (edge drawn when ρ̂xy|z(k) exceeds the Nα quantile of the standard normal)
- Number of conditional genes G (top largest-degree/importance genes selected from the ordinary CSN)

## Outputs

- Per-cell CCSNs (direct gene-gene association networks) for each conditional gene
- Integrated gene-gene interaction degree matrix per cell (CNDM, m × n)
- Hub genes with module memberships and network degrees (ccsn_hubs_gokce.csv)

## How to apply

Load the scRNA-seq gene expression matrix (e.g., Gokce dataset [46]) and, for each cell, first build an ordinary CSN by testing every gene pair with statistic ρxy, setting edge_xy(k)=1 when the null hypothesis of independence is rejected. Compute importance Dz(k) (sum of edges to gene z) and select the top G largest-degree genes as the conditional gene set per cell, since c-CSN chooses the top G≥1 largest 'importance' genes as conditional genes. For each gene pair and each conditional gene z, estimate ρxy|z from scatter-diagram neighborhood counts (nxyz(k), nxz(k), nyz(k), nz(k)) in 3D expression space, normalize it to ρ̂xy|z(k), and draw an edge (ωxy|z(k)=1) when ρ̂xy|z(k) exceeds the Nα quantile of the standard normal at significance level α — this conditional-independence filtering is what removes indirect associations that ordinary CSN overestimates. Average the G per-conditional-gene CCSNs via C̄k = (1/G)Σg Czg(k) (Equation 9) to obtain the integrated gene-gene interaction degree matrix per cell; downstream methods such as PCA, t-SNE, K-means, SC3, SIMLR, and Seurat are applicable to this degree matrix. Run module detection on the integrated CCSN, rank genes by network degree, and confirm the expected hubs (e.g., ONECUT1, ASCL1, NAR6A1) emerge, saving hubs with module memberships and degrees as ccsn_hubs_gokce.csv.

## Related tools

- **c-CSN** (Core method that constructs the conditional cell-specific network (CCSN) per cell, testing direct gene-gene associations via conditional independence and eliminating indirect associations) — https://github.com/LinLi-0909/c-CSN
- **SC3** (Optional downstream clustering method applicable to the c-CSN-transformed degree matrix (CNDM))
- **SIMLR** (Optional downstream clustering method applicable to the c-CSN-transformed degree matrix (CNDM))
- **Monocle** (Optional trajectory inference method for analyzing the transformed gene-gene association data along differentiation trajectories)

## Examples

```
git clone https://github.com/LinLi-0909/c-CSN
```

## Evaluation signals

- Per-cell edges are drawn only when normalized ρ̂xy|z(k) exceeds the Nα quantile of the standard normal at the chosen significance level α — verify the quantile threshold is applied
- The conditional gene set must consist of the top G largest-degree ('importance' Dz(k)) genes from the per-cell ordinary CSN, not random genes
- The integrated per-cell degree matrix equals the average of the G per-conditional-gene CCSNs, C̄k = (1/G)Σg Czg(k) (Equation 9) — check the averaging step
- Hub-gene ranking on the integrated CCSN should highlight the expected hubs ONECUT1, ASCL1, and NAR6A1 in the Gokce neural-differentiation modules
- Output file ccsn_hubs_gokce.csv must contain hub genes with their module memberships and network degrees

## Limitations

- The G conditional genes multiply computational cost G-fold, and no parallel computation scheme is provided; the authors state a parallel scheme is desired to reduce computation time
- c-CSN does not construct causal gene association networks; the directions of gene associations cannot be obtained and are left as future work
- CSN (the per-cell base network) overestimates gene-gene associations by including indirect associations, which is why conditional filtering is required — skipping it yields denser-than-real networks

## Evidence

- [abstract] we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell: "we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell"
- [abstract] eliminating indirect associations: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [methods] conditional gene selection: [evidence span withheld — anchor status: not_found]
- [methods] degree matrix integration: [evidence span withheld — anchor status: not_found]
- [intro] CSN overestimation rationale: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [supplementary] repository availability: "CCSN is available at https://github.com/LinLi-0909/c-CSN."
