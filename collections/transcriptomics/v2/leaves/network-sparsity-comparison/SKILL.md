---
name: network-sparsity-comparison
description: Use when comparing per-cell edge counts of CSN versus CCSN built from
  a normalized scRNA-seq matrix (e.g., Buettner mESC) to verify that conditional independence
  testing yields sparser, direct-association-only gene–gene networks.
when_to_use_negative:
- The goal is a denser co-expression network that includes indirect associations —
  CSN alone is intended for that, and CCSN filtering would remove them by design
- Input is bulk RNA-seq or an already-aggregated gene–gene association matrix — CCSN
  construction is defined per single cell and requires per-cell expression vectors
- The analysis requires causal/directed gene interactions — the paper states directions
  of gene associations cannot be obtained with c-CSN
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: Constructs the CSN and the conditional CCSNs per cell and computes the conditional
    statistics ρ̂xy|z(k) used for the sparsity comparison
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:b83261b0da24e2d366cd154dfa1e6e11389173a819016f440444fd745ed42ca9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# network-sparsity-comparison

## Summary

Reproduces the c-CSN paper's finding that conditional cell-specific networks (CCSNs) are much sparser per cell than baseline CSNs, because conditional independence testing filters out indirect gene–gene associations that cause CSN's overestimation.

## When to use

Use when comparing per-cell edge counts of CSN versus CCSN built from a normalized scRNA-seq matrix (e.g., Buettner mESC) to verify that conditional independence testing yields sparser, direct-association-only gene–gene networks.

## When NOT to use

- The goal is a denser co-expression network that includes indirect associations — CSN alone is intended for that, and CCSN filtering would remove them by design
- Input is bulk RNA-seq or an already-aggregated gene–gene association matrix — CCSN construction is defined per single cell and requires per-cell expression vectors
- The analysis requires causal/directed gene interactions — the paper states directions of gene associations cannot be obtained with c-CSN

## Inputs

- Normalized scRNA-seq gene expression matrix (e.g., Buettner mESC dataset, genes × cells)
- Significance level α for the Nα quantile threshold of the standard normal distribution
- Number of conditional genes G (top-degree hub genes selected per cell)

## Outputs

- Per-cell CSN with edge weights from ρxy statistic testing
- G conditional CCSNs per cell with binary edge weights ωxy|z(k)
- Per-cell edge count comparison table (CSN vs. CCSN) demonstrating CCSN sparsity

## How to apply

For each cell of the normalized gene expression matrix, first build the baseline CSN by testing every gene pair with the ρxy statistic against the Nα quantile of the standard normal distribution at the chosen significance level. Then compute per-cell conditional-gene importance Dz(k) as the sum of CSN edge weights (Equation 8) and select the top G highest-degree 'importance' genes as the conditional gene set. For each cell and each conditional gene z, build the CCSN by testing conditional independence with the normalized statistic ρ̂xy|z(k), setting edge weight ωxy|z(k)=1 when it exceeds Nα (Equation 7). Count edges per cell in the CSN and in the CCSN (summed over the G conditional CCSNs per Equation 9) and compare the per-cell edge counts. The rationale is that CSN includes both direct and indirect associations, so filtering via conditional independence should systematically reduce edge density while retaining only direct gene–gene interactions.

## Related tools

- **c-CSN** (Constructs the CSN and the conditional CCSNs per cell and computes the conditional statistics ρ̂xy|z(k) used for the sparsity comparison) — https://github.com/LinLi-0909/c-CSN

## Evaluation signals

- Mean per-cell CCSN edge counts are substantially lower than CSN edge counts across all cells (result_ccsn_sparser_csn acceptance criterion)
- Every retained CCSN edge (ωxy|z(k)=1) corresponds to a conditional statistic ρ̂xy|z(k) exceeding Nα of the standard normal distribution
- Conditional gene sets contain exactly the top G genes by per-cell importance Dz(k) (sum of CSN edge weights, Equation 8)
- Per-cell CCSN edge count equals the sum over the G conditional CCSNs per Equation 9
- Retained edges represent direct gene–gene associations only — indirect associations present in the CSN are removed

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase; the authors state it is desired
- c-CSN is not designed to construct causal gene association networks, and the directions of the gene associations cannot be obtained
- The conditional independence test requires estimating ρxy|z numerically from scatter diagram neighborhoods in 3D expression space, introducing estimation sensitivity

## Evidence

- [abstract] c-CSN method can measure the direct associations between genes by eliminating the indirect associations.: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [intro] CSN overestimation motivation: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [methods] conditional gene selection: [evidence span withheld — anchor status: not_found]
- [supplementary] code availability: "CCSN is available at https://github.com/LinLi-0909/c-CSN."
