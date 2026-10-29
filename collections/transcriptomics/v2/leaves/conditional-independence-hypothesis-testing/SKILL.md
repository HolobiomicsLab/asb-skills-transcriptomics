---
name: conditional-independence-hypothesis-testing
description: Use when a normalized scRNA-seq or RNA-seq gene expression matrix (m
  genes x n cells) must be transformed into per-cell direct gene-gene association
  networks, e.g. to reproduce CSN/CCSN sparsity, hub genes, or per-sample pathway
  connectivity.
when_to_use_negative:
- The goal is to infer causal direction of gene-gene associations — the authors state
  c-CSN is not designed to construct causal gene association networks and directions
  cannot be obtained
- The input is already a network or degree matrix (CNDM); this skill transforms raw/normalized
  expression matrices into network data
- Parallel/computationally cheap processing is required for very large cell counts
  — no parallel computation scheme is provided despite the G-fold computational cost
  increase
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: c-CSN
  role: Implements the conditional independence test and CCSN/CNDM construction from
    scRNA-seq expression matrices
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:4e31f80adfd5637bade41bbeaaa5fd6db989c025e971ba94d71e73dd7e0044e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# conditional-independence-hypothesis-testing

## Summary

Implementation of the c-CSN statistical procedure, which builds a conditional cell-specific network (CCSN) for each cell by hypothesis-testing conditional independence of gene pairs given top-degree hub genes, thereby eliminating indirect associations that cause CSN overestimation.

## When to use

Use when a normalized scRNA-seq or RNA-seq gene expression matrix (m genes x n cells) must be transformed into per-cell direct gene-gene association networks, e.g. to reproduce CSN/CCSN sparsity, hub genes, or per-sample pathway connectivity.

## When NOT to use

- The goal is to infer causal direction of gene-gene associations — the authors state c-CSN is not designed to construct causal gene association networks and directions cannot be obtained
- The input is already a network or degree matrix (CNDM); this skill transforms raw/normalized expression matrices into network data
- Parallel/computationally cheap processing is required for very large cell counts — no parallel computation scheme is provided despite the G-fold computational cost increase

## Inputs

- Normalized gene expression matrix with m genes and n cells (e.g. Buettner mESC scRNA-seq matrix or TCGA-LUAD RNA-seq profiles)
- Significance level alpha defining the N_alpha quantile of the standard normal distribution
- Number of conditional genes G (G >= 1), chosen as top-degree hub genes from the CSN

## Outputs

- Per-cell CCSN C_zg(k) for each conditional gene of each cell
- Integrated per-cell gene-gene interaction degree matrix (average of the G CCSNs, Equation 9)
- Conditional network degree matrix (CNDM), an m x n matrix, e.g. saved as cndm.csv
- Per-cell edge counts for CSN vs CCSN comparison

## How to apply

For each cell k, first build the plain CSN with the rho_xy statistic, compute the 'importance' Dz(k) (sum of CSN edge weights to gene z, Equation 8), and select the top G (G >= 1) largest-importance hub genes as the conditional gene set. For each conditional gene z, estimate neighborhood counts (nz(k), nxz(k), nyz(k), nxyz(k)) from scatter diagrams of expression values in 3D space, compute rho_xy|z(k) = nxyz(k)/nz(k) - nxz(k)*nyz(k)/nz(k)^2, and normalize it to rho_hat_xy|z(k). Test H0 (genes x and y conditionally independent given z) by comparing the normalized statistic against the N_alpha quantile of the standard normal at significance level alpha, setting edge weight omega_xy|z(k)=1 when it exceeds N_alpha. Average the G per-conditional-gene CCSNs via Equation 9 (C^-k = (1/G) * sum_g C_zg(k)) to obtain per-cell network degrees c_ij^k, then assemble the m x n conditional network degree matrix (CNDM). The rationale is that conditional independence filtering removes indirect associations, overcoming CSN's overestimation of gene-gene interactions.

## Related tools

- **c-CSN** (Implements the conditional independence test and CCSN/CNDM construction from scRNA-seq expression matrices) — https://github.com/LinLi-0909/c-CSN

## Evaluation signals

- Output cndm.csv has the same m genes x n cells dimension as the input gene expression matrix
- Each constructed CCSN is sparser than the corresponding CSN (indirect associations filtered out)
- Mean per-cell edge counts of the CCSN are substantially lower than those of the CSN across all cells (result_ccsn_sparser_csn)
- Edge weights omega_xy|z(k)=1 are assigned only where the normalized statistic rho_hat_xy|z(k) exceeds the N_alpha quantile of the standard normal at significance level alpha
- Per-sample degree summaries are reproducible, e.g. EGF/EGFR CCSN degrees consistently higher across TCGA-LUAD tumor samples

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel scheme is desired
- c-CSN is not designed to construct causal gene association networks, and the directions of the gene associations cannot be obtained
- The G conditional-gene choice (top G largest-'importance' hub genes from the CSN) introduces a user-set parameter G >= 1 that influences cost and integration

## Evidence

- [abstract] Direct associations via conditional independence: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [abstract] CCSN per cell: "we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell"
- [methods] Conditional gene selection: [evidence span withheld — anchor status: not_found]
- [methods] Scatter-diagram estimation: "We numerically estimate the value of ρxy|z by making a scatter diagram based on gene expression data"
- [methods] Integration into CNDM: [evidence span withheld — anchor status: not_found]
