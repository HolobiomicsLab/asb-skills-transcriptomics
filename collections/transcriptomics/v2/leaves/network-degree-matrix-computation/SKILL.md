---
name: network-degree-matrix-computation
description: Use when per-cell CCSNs from c-CSN are available and the goal is to convert
  them into an m genes × n cells degree matrix (CNDM) so clustering, dimension reduction,
  or NFE analysis can operate on network data instead of the raw gene expression matrix.
when_to_use_negative:
- The goal is causal gene association networks or directed gene-gene edges — the article
  states c-CSN does not obtain directions of gene associations
- A plain (unconditional) cell-specific network is sufficient and the G-fold computational
  cost of testing G conditional genes per cell is prohibitive, especially since no
  parallel computation scheme is provided
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: c-CSN
  role: Constructs the per-cell CCSNs that are averaged into the CNDM and provides
    the CSN-based hub-gene ranking (Dz(k)) used to select conditional genes
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:c025bcb280896b81b5d566b153702dde15f914e414c6d3400e1a5bf10bcc2b21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - SC3
  - c-CSN
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# network-degree-matrix-computation

## Summary

Computes the conditional network degree matrix (CNDM) by averaging the per-cell CCSNs produced by c-CSN into an m × n matrix of gene-gene interaction degrees, transforming noisy single-cell gene expression into robust network-based features that downstream methods can consume in place of the GEM.

## When to use

Use when per-cell CCSNs from c-CSN are available and the goal is to convert them into an m genes × n cells degree matrix (CNDM) so clustering, dimension reduction, or NFE analysis can operate on network data instead of the raw gene expression matrix.

## When NOT to use

- The goal is causal gene association networks or directed gene-gene edges — the article states c-CSN does not obtain directions of gene associations
- A plain (unconditional) cell-specific network is sufficient and the G-fold computational cost of testing G conditional genes per cell is prohibitive, especially since no parallel computation scheme is provided

## Inputs

- Normalized scRNA-seq gene expression matrix (m genes × n cells)
- Per-cell CCSNs C_zg(k) constructed by c-CSN for each of the G conditional genes

## Outputs

- Conditional network degree matrix (CNDM), an m genes × n cells matrix of per-gene degrees v_i^k
- cndm.csv table of degrees compatible with downstream clustering and dimension-reduction tools

## How to apply

After constructing the G per-cell CCSNs C_zg(k) for cell k (each derived from the hypothesis test on the normalized conditional statistic rho_xy|z with the standard-normal significance quantile N_alpha), integrate them by averaging: C^-k = (1/G) * sum_g C_zg(k) = c_ij^k, yielding per-cell gene-gene interaction degrees. Then compute per-gene degrees v_i^k = sum_j c_ij^k and assemble them column-wise into the m × n CNDM. The conditional gene set should be the top G (G >= 1) largest-'importance' hub genes ranked by CSN degree Dz(k) from Equation (8). Save the CNDM as cndm.csv and validate that its dimensions match the input gene expression matrix and that each CCSN is sparser than the corresponding CSN because indirect associations were filtered out.

## Related tools

- **c-CSN** (Constructs the per-cell CCSNs that are averaged into the CNDM and provides the CSN-based hub-gene ranking (Dz(k)) used to select conditional genes) — https://github.com/LinLi-0909/c-CSN

## Examples

```
python construct_cndm.py --expression gem.csv --top-hub-genes G --output cndm.csv
```

## Evaluation signals

- cndm.csv exists and has the same m genes × n cells dimension as the input gene expression matrix
- Each constructed CCSN is sparser than the corresponding CSN, confirming indirect associations were filtered out
- Per-gene degrees v_i^k equal sum_j c_ij^k of the averaged CCSN C^-k = (1/G) * sum_g C_zg(k)
- Downstream clustering on the CNDM yields higher adjusted Rand index (ARI) than on the original GEM (e.g., on Buettner, Kolodziejczyk, and Chu-time datasets)

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state a parallel computation scheme is desired
- c-CSN is not designed to construct causal gene association networks; directions of gene associations cannot be obtained
- The method requires choosing G conditional genes (G >= 1), and each conditional gene multiplies the per-cell hypothesis-testing cost

## Evidence

- [methods] Integration of CCSNs into cell-specific gene-gene interaction degree matrix: [evidence span withheld — anchor status: not_found]
- [methods] CNDM construction: "an m × n matrix conditional network degree matrix (CNDM) is obtained"
- [methods] Conditional gene selection: [evidence span withheld — anchor status: not_found]
- [intro] Overestimation motivation: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations"
- [abstract] Direct association finding: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
