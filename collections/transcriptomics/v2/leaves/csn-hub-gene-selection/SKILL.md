---
name: csn-hub-gene-selection
description: Use when a per-cell scRNA-seq gene expression matrix (m genes x n cells)
  must be converted to conditional cell-specific networks and a conditional gene set
  (hub genes) is needed to condition the gene-gene independence tests per cell.
when_to_use_negative:
- Input is bulk RNA-seq or aggregated expression without per-cell vectors — the conditional
  hub set is defined independently for each cell k
- The expression matrix is unnormalized/noisy raw counts, since rho_xy is estimated
  from the scatter of expression values and requires normalization first
- The goal is a single population-level gene-gene network rather than per-cell conditional
  networks (a plain CSN suffices, but it overestimates associations by including indirect
  ones)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: Supplies the CSN construction (rho_xy statistic, Equation 3) whose per-gene
    edge-weight sums Dz(k) (Equation 8) rank hub genes, and the downstream CCSN framework
    that conditions on the selected hubs
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:91b7b69348b7bcb8b40f21052186468a3331a5487151022378b58e56d2b58aa7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# csn-hub-gene-selection

## Summary

Selects the conditional gene set for c-CSN by building each cell's plain CSN with the rho_xy statistic, summing edge weights to each gene z via Equation (8) Dz(k), and taking the top G (G >= 1) largest-'importance' hub genes. These hubs define the conditioning variables under which direct gene-gene associations are tested, filtering indirect associations.

## When to use

Use when a per-cell scRNA-seq gene expression matrix (m genes x n cells) must be converted to conditional cell-specific networks and a conditional gene set (hub genes) is needed to condition the gene-gene independence tests per cell.

## When NOT to use

- Input is bulk RNA-seq or aggregated expression without per-cell vectors — the conditional hub set is defined independently for each cell k
- The expression matrix is unnormalized/noisy raw counts, since rho_xy is estimated from the scatter of expression values and requires normalization first
- The goal is a single population-level gene-gene network rather than per-cell conditional networks (a plain CSN suffices, but it overestimates associations by including indirect ones)

## Inputs

- Normalized gene expression matrix with m genes and n cells (one expression vector per cell)
- Plain per-cell CSN edge weights from the rho_xy statistic (Equation 3)
- Hub-gene count G (G >= 1) controlling how many top-'importance' genes enter the conditional set

## Outputs

- Per-cell conditional gene set {z_1, ..., z_G}: the top G genes ranked by Equation (8) degree Dz(k)
- Per-gene importance ranking Dz(k) (sum of CSN edge weights to gene z) for each cell k
- Conditional gene sets consumed downstream to build CCSN C_zg(k) and the m x n CNDM (cndm.csv)

## How to apply

For each cell k, first construct the plain CSN from the normalized expression scatter using the rho_xy statistic, then rank every gene by its 'importance' Dz(k) — the sum of its edge weights in the CSN as defined by Equation (8). Select the top G (G >= 1) largest-'importance' genes as the conditional gene set; the paper applied exactly this CSN-based hub definition for the c-CSN method. These G genes become the conditioning variables z_g for the subsequent conditional independence tests of every gene pair (x, y). The rationale is that conditioning on high-degree hub genes removes interactive effects from other genes, so the resulting CCSN C_zg(k) captures direct rather than indirect associations, overcoming CSN's overestimation of gene-gene associations.

## Related tools

- **c-CSN** (Supplies the CSN construction (rho_xy statistic, Equation 3) whose per-gene edge-weight sums Dz(k) (Equation 8) rank hub genes, and the downstream CCSN framework that conditions on the selected hubs) — https://github.com/LinLi-0909/c-CSN

## Evaluation signals

- For each cell k, the selected conditional genes are exactly the top G genes by Dz(k) = sum of CSN edge weights to gene z per Equation (8)
- Downstream cndm.csv has the same dimension (m genes x n cells) as the input gene expression matrix
- Each constructed CCSN is sparser than the corresponding plain CSN, confirming indirect associations were filtered out
- Increasing G raises the number of per-cell CCSNs integrated via C^-k = (1/G) * sum_g C_zg(k) proportionally to G
- Each chosen hub z satisfies G >= 1 and produces a well-defined conditional test statistic rho_xy|z(k) with mu_xy|z(k) = 0 normalization

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel scheme is desired
- Hub selection depends on the plain CSN, which overestimates gene-gene associations by including both direct and indirect associations — hubs may reflect indirect edges
- Choices of G and the significance-level quantile N_alpha are user parameters; the article gives no adaptive rule for choosing them
- c-CSN is not designed to construct causal gene association networks, and directions of gene associations cannot be obtained

## Evidence

- [methods] Selection of conditional genes via top-degree hub genes in CSN: [evidence span withheld — anchor status: not_found]
- [methods] Hypothesis test for conditional independence given conditional gene z: "we first use CSN or Equation (3) to distinguish the independence of gene x and gene y and we then use the following hypothesis test"
- [methods] CCSN construction per cell per conditional gene: "CCSN Czg(k) is obtained for cell k given conditional gene zg"
- [abstract] c-CSN measures direct associations: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [intro] Motivation: CSN overestimation: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
