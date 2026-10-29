---
name: cell-specific-network-construction
description: Use when a normalized scRNA-seq gene expression matrix (m genes × n cells)
  needs to be transformed into direct gene–gene association networks per cell, or
  into a CNDM for downstream clustering/dimension reduction.
when_to_use_negative:
- Input is not single-cell expression data or already consists of network/association
  data rather than a gene expression matrix
- The goal is to infer causal gene association networks or directions of gene regulation
  — the article states c-CSN is not designed for this and directions cannot be obtained
- Only a bulk RNA-seq profile with a single sample is available, since CCSNs are constructed
  per individual cell
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: c-CSN
  role: Core method that constructs the conditional cell-specific network (CCSN) for
    each cell via conditional independence testing and assembles the CNDM
  repo: https://github.com/LinLi-0909/c-CSN
- name: SC3
  role: Downstream clustering method applicable to the c-CSN-transformed degree matrix
    (CNDM) instead of the gene expression matrix
- name: Seurat
  role: Downstream clustering method applied to the CNDM for benchmarking ARI against
    the original GEM
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:406dea8d9b55c5edded4d8476b2424eeae70066bdf219ab32c0dc648ee15149b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - SC3
  - Seurat
  - c-CSN
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-specific-network-construction

## Summary

Constructs a conditional cell-specific network (CCSN) for each single cell from a normalized gene expression matrix by testing conditional independence of gene pairs given top-degree hub genes, then aggregates per-cell CCSNs into an m × n conditional network degree matrix (CNDM) that removes CSN's overestimated indirect associations.

## When to use

Use when a normalized scRNA-seq gene expression matrix (m genes × n cells) needs to be transformed into direct gene–gene association networks per cell, or into a CNDM for downstream clustering/dimension reduction.

## When NOT to use

- Input is not single-cell expression data or already consists of network/association data rather than a gene expression matrix
- The goal is to infer causal gene association networks or directions of gene regulation — the article states c-CSN is not designed for this and directions cannot be obtained
- Only a bulk RNA-seq profile with a single sample is available, since CCSNs are constructed per individual cell

## Inputs

- Normalized gene expression matrix (m genes × n cells) from scRNA-seq data, e.g. the Buettner mESC matrix
- Significance level alpha for the standard-normal quantile N_alpha
- Number of conditional hub genes G (G >= 1)

## Outputs

- Per-cell CCSN C_zg(k) for each cell k and each conditional gene z
- Integrated per-cell gene-gene association matrix C^-k = (1/G) * sum_g C_zg(k)
- m × n conditional network degree matrix (CNDM), saved as cndm.csv

## How to apply

For each cell, first build the plain CSN using the rho_xy statistic tested against the N_alpha quantile of the standard normal at the chosen significance level, then rank genes by conditional-gene importance Dz(k) (sum of CSN edge weights, Equation 8) and select the top G (G >= 1) largest-'importance' hub genes as the conditional gene set. For each conditional gene z, estimate neighborhood counts (nz(k), nxz(k), nyz(k), nxyz(k)) from the scatter diagram of expression values in 3D space, compute rho_xy|z(k) = nxyz(k)/nz(k) - nxz(k)*nyz(k)/nz(k)^2, and normalize it (mu_xy|z(k) = 0 with sigma_xy|z(k) per the paper's formula). Apply the hypothesis test H0: genes x and y conditionally independent given z, setting edge weight omega_xy|z(k) = 1 when the normalized statistic exceeds N_alpha, yielding a CCSN C_zg(k) per conditional gene. Integrate the G per-cell CCSNs by averaging C^-k = (1/G) * sum_g C_zg(k) = c_ij^k and compute per-gene degrees v_i^k = sum_j c_ij^k to assemble the m × n CNDM, saving it as cndm.csv. The rationale is that conditioning on hub genes filters out indirect associations that make CSN networks denser than the true molecular network.

## Related tools

- **c-CSN** (Core method that constructs the conditional cell-specific network (CCSN) for each cell via conditional independence testing and assembles the CNDM) — https://github.com/LinLi-0909/c-CSN
- **SC3** (Downstream clustering method applicable to the c-CSN-transformed degree matrix (CNDM) instead of the gene expression matrix)
- **Seurat** (Downstream clustering method applied to the CNDM for benchmarking ARI against the original GEM)

## Examples

```
python c_csn.py --input gem_normalized.csv --alpha 0.05 --G 3 --output cndm.csv
```

## Evaluation signals

- cndm.csv exists and has the same dimension (m genes × n cells) as the input gene expression matrix
- Each constructed CCSN is sparser than the corresponding CSN (indirect associations filtered out)
- Mean per-cell edge counts of the CCSN are substantially lower than those of the CSN across all cells
- Clustering the CNDM (K-means, hierarchical, K-medoids, SC3) yields higher ARI against gold-standard cell type labels than clustering the GEM on Buettner, Kolodziejczyk, and Chu-time datasets
- Conditional gene sets are reproducibly the top-degree hub genes ranked by Dz(k) (sum of CSN edge weights, Equation 8)

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state it is desired
- c-CSN is not designed to construct causal gene association networks, and the directions of the gene associations cannot be obtained
- Computational cost scales with the number of conditional genes G, since one CCSN is built per conditional gene per cell

## Evidence

- [abstract] c-CSN method can measure the direct associations between genes by eliminating the indirect associations.: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [intro] CSN overestimation problem: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [methods] conditional gene selection: [evidence span withheld — anchor status: not_found]
- [methods] CCSN integration into CNDM: [evidence span withheld — anchor status: not_found]
- [abstract] repository availability: "c-CSN is publicly available at https://github.com/LinLi-0909/c-CSN."
