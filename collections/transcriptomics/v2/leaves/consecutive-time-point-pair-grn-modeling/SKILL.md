---
name: consecutive-time-point-pair-grn-modeling
description: Use when input is time-stamped cross-sectional single-cell expression
  data (no true longitudinal linkage) and the goal is a GRN driving cell-state transitions
  between consecutive time points.
when_to_use_negative:
- The goal is to infer the GRN that maintains a single current cell state/time point
  rather than the mechanism driving the transition along the cell lineage (most alternative
  TSCS methods are limited to a single stage)
- Data are true longitudinal single-cell time series with real temporal linkage of
  individual cells — COSLIR is designed for TSCS data lacking pseudo-time/real temporal
  information
- An accurate pseudo-time trajectory already exists and downstream methods depend
  on it; COSLIR intentionally bypasses pseudo-time inference rather than refining
  it
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: COSLIR (Python version)
  role: 'Primary inference tool: solves the covariance-restricted sparse linear regression
    on each consecutive time-point pair using first and second moments'
  repo: github.com/Ge-lab-pku/COSLIR
- name: COSLIR (MATLAB version)
  role: Alternative implementation of the same model, tested on MATLAB 9.4 (R2018a)
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Only basic dependency for the Python version (tested on v1.14)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:206b91c936bdcd8f9349cd9345b742012b7ee265dc8749d8033d0341fad162ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# consecutive-time-point-pair-grn-modeling

## Summary

Reconstructs directed, signed, weighted gene regulatory networks (GRNs) governing cell-state transitions from time-stamped cross-sectional (TSCS) single-cell RNA-seq/qPCR data by running COSLIR on each consecutive time-point pair, using only the first and second moments of the samples instead of pseudo-time inference.

## When to use

Use when input is time-stamped cross-sectional single-cell expression data (no true longitudinal linkage) and the goal is a GRN driving cell-state transitions between consecutive time points.

## When NOT to use

- The goal is to infer the GRN that maintains a single current cell state/time point rather than the mechanism driving the transition along the cell lineage (most alternative TSCS methods are limited to a single stage)
- Data are true longitudinal single-cell time series with real temporal linkage of individual cells — COSLIR is designed for TSCS data lacking pseudo-time/real temporal information
- An accurate pseudo-time trajectory already exists and downstream methods depend on it; COSLIR intentionally bypasses pseudo-time inference rather than refining it

## Inputs

- Time-stamped cross-sectional (TSCS) single-cell RNA-seq expression dataset (e.g., BEELINE hESC or mESC), split into consecutive time-point pairs
- Ground truth GRN (e.g., BEELINE cell-type-specific networks) for evaluation
- First and second moments (mean and covariance) of the samples for each consecutive time-point pair

## Outputs

- Directed, signed, weighted transition GRN per consecutive time-point pair
- Early-precision curves over ranked edges at varying k, per dataset
- Comparison figure of COSLIR vs. baselines (e.g., fig4ab_early_precision_curves.png)

## How to apply

Obtain COSLIR from the released package (Python or MATLAB folder) and check its DESCRIPTION file for usage details; the Python version is tested on Python 3.6.4 with NumPy 1.14 as the only basic requirement, and MATLAB 9.4 (R2018a) for the MATLAB version. Load the TSCS dataset, split it into consecutive time-point pairs, and run COSLIR on each pair using only the first and second moments of the samples to infer a directed, signed, weighted transition GRN without pseudo-time construction. Follow the README's general workflow: solve the model multiple times after bootstrapping, then apply a clipping on the rescaled value and the confidence. Evaluate the inferred networks against ground truth (e.g., BEELINE cell-type-specific networks) with early precision at varying k, comparing ranked-edge curves against baselines such as SINCERITIES, SCODE, SCRIBE, and SINGE.

## Related tools

- **COSLIR (Python version)** (Primary inference tool: solves the covariance-restricted sparse linear regression on each consecutive time-point pair using first and second moments) — github.com/Ge-lab-pku/COSLIR
- **COSLIR (MATLAB version)** (Alternative implementation of the same model, tested on MATLAB 9.4 (R2018a)) — github.com/Ge-lab-pku/COSLIR
- **NumPy** (Only basic dependency for the Python version (tested on v1.14))

## Examples

```
cd python && python coslir.py --input hesc_t1_t2.csv  # per DESCRIPTION; solve on each consecutive time-point pair with bootstrapping + clipping
```

## Evaluation signals

- On mESC and hESC, COSLIR's early precision at varying k should compete favorably with SINCERITIES, SCODE, SCRIBE, and SINGE, matching the qualitative pattern of Fig. 4A-B
- In simulations, COSLIR should achieve perfect accuracy in the oracle case and robust performance in real-world scenarios
- Running time should remain nearly independent of the number of cells
- The output network must be directed, signed, and weighted, and sparsified via the README's clipping on the rescaled value and the confidence after bootstrap solving

## Limitations

- This is a beta version; some bugs may still remain, so use with caution and report issues
- COSLIR relies exclusively on the first and second moments of samples, discarding higher-order structure
- It requires time-stamped cross-sectional data grouped into consecutive time-point pairs; it cannot exploit real temporal linkage within individual cells

## Evidence

- [abstract] utilizing only the first and second moments of samples between two consecutive time points: "utilizing only the first and second moments of samples between two consecutive time points"
- [methods] COSLIR is tested on Python v3.6.4: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)"
- [readme] README workflow: bootstrapping and clipping: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [results] Early precision against BEELINE ground truth: [evidence span withheld — anchor status: not_found]
- [readme] TSCS data lack real temporal information: "Time-stamped cross-sectional (TSCS) single-cell expression data is commonly generated, which lacks real temporal information."
