---
name: neighborhood-count-probability-estimation
description: Use when building a conditional cell-specific network (CCSN) from a normalized
  scRNA-seq gene expression matrix and you must estimate rho_xy|z(k) via scatter-diagram
  neighborhood counts given a conditional gene z.
when_to_use_negative:
- The goal is a plain (unconditioned) CSN of gene-gene associations without filtering
  indirect associations — no conditional gene z is involved
- The input is already a network/degree matrix (e.g., a CNDM) rather than raw normalized
  gene expression values, so no scatter diagram can be constructed
- Causal gene association networks with directed edges are required — c-CSN explicitly
  does not obtain directions of gene associations
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2269
tools:
- name: c-CSN
  role: Method whose Construction-of-CCSN step uses this neighborhood-count estimation
    of rho_xy|z to build the per-cell CCSN
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:b765abb35450da76f430c411eb613ffc4303f3d708cacf4dffdf3997011fa367
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Neighborhood-count probability estimation

## Summary

Numerically estimates the conditional co-expression statistic rho_xy|z(k) for the c-CSN method by counting scatter-diagram neighborhood points in 3D gene expression space (genes x, y, conditional gene z) for each cell k, then normalizing it for the conditional-independence hypothesis test.

## When to use

Use when building a conditional cell-specific network (CCSN) from a normalized scRNA-seq gene expression matrix and you must estimate rho_xy|z(k) via scatter-diagram neighborhood counts given a conditional gene z.

## When NOT to use

- The goal is a plain (unconditioned) CSN of gene-gene associations without filtering indirect associations — no conditional gene z is involved
- The input is already a network/degree matrix (e.g., a CNDM) rather than raw normalized gene expression values, so no scatter diagram can be constructed
- Causal gene association networks with directed edges are required — c-CSN explicitly does not obtain directions of gene associations

## Inputs

- Normalized gene expression matrix with m genes x n cells (scRNA-seq data)
- Conditional gene set z (top G >= 1 largest-'importance' hub genes selected from the plain CSN via rho_xy and Dz(k) ranking)
- Per-cell scatter of expression values for the gene triplet (x, y, z) in 3D space

## Outputs

- Neighborhood counts (nz(k), nxz(k), nyz(k), nxyz(k)) per cell k and conditional gene z
- Estimated conditional statistic rho_xy|z(k) = nxyz(k)/nz(k) - nxz(k)*nyz(k)/nz(k)^2
- Normalized statistic used to set binary edge weights omega_xy|z(k) in the per-cell CCSN C_zg(k)

## How to apply

For each cell k and each conditional gene z (chosen as one of the top G largest-'importance' hub genes ranked by the plain CSN using the rho_xy statistic and Equation (8) Dz(k)), make a scatter diagram in the 3D expression space of genes x, y, and z. Count neighborhood points nz(k), nxz(k), nyz(k), and nxyz(k), then compute rho_xy|z(k) = nxyz(k)/nz(k) - nxz(k)*nyz(k)/nz(k)^2. Normalize this statistic with mu_xy|z(k) = 0 and the paper's sigma_xy|z(k) so the conditional-independence hypothesis test (H0: genes x and y conditionally independent given z) can threshold edge weights omega_xy|z(k) at the significance-level quantile N_alpha. This scatter-diagram counting procedure is the numerical estimation step that grounds conditional independence in the actual expression values of each individual cell.

## Related tools

- **c-CSN** (Method whose Construction-of-CCSN step uses this neighborhood-count estimation of rho_xy|z to build the per-cell CCSN) — https://github.com/LinLi-0909/c-CSN

## Examples

```
rho_xyz = n_xyz / n_z - (n_xz * n_yz) / n_z**2  # neighborhood-count estimate of rho_xy|z(k) from 3D scatter counts
```

## Evaluation signals

- All neighborhood counts nz(k), nxz(k), nyz(k), nxyz(k) are non-negative integers derived from the 3D scatter diagram of cell k
- Computed rho_xy|z(k) matches the formula nxyz(k)/nz(k) - nxz(k)*nyz(k)/nz(k)^2 exactly
- After normalization, the hypothesis test yields binary edge weights omega_xy|z(k) in {0,1} at the N_alpha significance-level quantile
- The resulting CCSN C_zg(k) is sparser than the corresponding plain CSN, confirming indirect associations were filtered out
- Aggregated degrees v_i^k = sum_j c_ij^k assemble an m x n CNDM with the same dimensions as the input gene expression matrix

## Limitations

- Conditional genes are only as good as the upstream plain CSN ranking; the top G largest-'importance' hub genes (G >= 1) must be selected first
- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel scheme is desired
- c-CSN is not designed to construct causal gene association networks, and directions of gene associations cannot be obtained

## Evidence

- [methods] numerical estimation of rho_xy|z: "We numerically estimate the value of ρxy|z by making a scatter diagram based on gene expression data"
- [methods] selection of conditional genes: [evidence span withheld — anchor status: not_found]
- [methods] CCSN construction output: "CCSN Czg(k) is obtained for cell k given conditional gene zg"
- [intro] purpose of the transformed network data: "c-CSN can transform the original gene expression data of each cell to the direct and robust gene–gene association data (or network data) of the same cell"
- [abstract] conditional-independence rationale: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
