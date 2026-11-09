---
name: baseline-grn-method-benchmarking
description: Use when a new GRN inference method for time-stamped cross-sectional
  (TSCS) single-cell data must be compared to baselines like SINCERITIES, SCODE, SCRIBE,
  and SINGE using early precision against BEELINE ground truth networks.
when_to_use_negative:
- The data are true longitudinal single-cell time series with linkage across time
  points rather than time-stamped cross-sectional (TSCS) data from independent batches
- The goal is to evaluate methods that rely on pseudo-time trajectory construction
  rather than directly comparing inference performance against BEELINE ground truth
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: COSLIR (Python version)
  role: Method under test; infers directed, signed, weighted transition GRNs from
    first and second moments of consecutive time-point samples
  repo: github.com/Ge-lab-pku/COSLIR
- name: SINCERITIES
  role: Baseline GRN inference method run on the same consecutive time-point pairs
- name: SCODE
  role: Baseline GRN inference method run on the same consecutive time-point pairs
- name: SCRIBE
  role: Baseline GRN inference method run on the same consecutive time-point pairs
- name: SINGE
  role: Baseline GRN inference method run on the same consecutive time-point pairs
- name: Python 3.6.4
  role: Runtime environment for executing the COSLIR Python version
- name: NumPy 1.14
  role: Only basic dependency of the COSLIR Python version
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:6707fa8f8b8fddc80d8700eea4ff8413f3a63576bfdc0cda67625906290a30b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  - Python 3.6.4
  - SCODE
  - SCRIBE
  - SINCERITIES
  - SINGE
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# baseline-grn-method-benchmarking

## Summary

Benchmark a GRN inference method (COSLIR) against established baselines (SINCERITIES, SCODE, SCRIBE, SINGE) on TSCS single-cell RNA-seq datasets using early precision against cell-type-specific ground truth networks.

## When to use

Use when a new GRN inference method for time-stamped cross-sectional (TSCS) single-cell data must be compared to baselines like SINCERITIES, SCODE, SCRIBE, and SINGE using early precision against BEELINE ground truth networks.

## When NOT to use

- The data are true longitudinal single-cell time series with linkage across time points rather than time-stamped cross-sectional (TSCS) data from independent batches
- The goal is to evaluate methods that rely on pseudo-time trajectory construction rather than directly comparing inference performance against BEELINE ground truth

## Inputs

- BEELINE hESC single-cell RNA-seq time-stamped cross-sectional (TSCS) dataset
- BEELINE mESC single-cell RNA-seq time-stamped cross-sectional (TSCS) dataset
- BEELINE cell-type-specific ground truth networks for hESC and mESC
- COSLIR Python version package (with DESCRIPTION usage details)
- Baseline method implementations: SINCERITIES, SCODE, SCRIBE, SINGE

## Outputs

- Inferred directed, signed, weighted transition GRNs for COSLIR and each baseline on every consecutive time-point pair
- Early-precision curves over ranked edges at varying k for hESC and mESC
- Comparative figure of COSLIR vs. baselines early precision (Fig. 4A-B, e.g. fig4ab_early_precision_curves.png)

## How to apply

Load the BEELINE hESC and mESC single-cell RNA-seq time-stamped cross-sectional datasets together with their cell-type-specific ground truth networks. Run COSLIR (Python 3.6.4 with NumPy 1.14, obtained from the released package folder after checking its DESCRIPTION file) separately on each consecutive time-point pair of each dataset, using only the first and second moments of the samples to infer the directed, signed, weighted transition GRN. Run SINCERITIES, SCODE, SCRIBE, and SINGE on the same consecutive time-point pairs to obtain their inferred networks. Evaluate every inferred network against the ground truth using early precision at varying k, computing the curve over ranked edges for both hESC and mESC. Plot the early-precision curves of COSLIR versus the four baselines for hESC and mESC (Fig. 4A-B) and save the figure.

## Related tools

- **COSLIR (Python version)** (Method under test; infers directed, signed, weighted transition GRNs from first and second moments of consecutive time-point samples) — github.com/Ge-lab-pku/COSLIR
- **SINCERITIES** (Baseline GRN inference method run on the same consecutive time-point pairs)
- **SCODE** (Baseline GRN inference method run on the same consecutive time-point pairs)
- **SCRIBE** (Baseline GRN inference method run on the same consecutive time-point pairs)
- **SINGE** (Baseline GRN inference method run on the same consecutive time-point pairs)
- **Python 3.6.4** (Runtime environment for executing the COSLIR Python version)
- **NumPy 1.14** (Only basic dependency of the COSLIR Python version)

## Evaluation signals

- COSLIR's early-precision values rank on par with the best-performing baseline method on both hESC and mESC datasets
- Early precision is computed at varying k against the BEELINE cell-type-specific ground truth for every inferred network
- The reproduced curves match the qualitative pattern reported in Fig. 4A-B
- COSLIR uses only the first and second moments of samples, not pseudo-time ordering, when inferring each transition GRN

## Limitations

- COSLIR is a beta version; the README notes that some bugs may still remain and it should be used with caution
- Only the first and second moments of samples are used, so temporal linkage across time points is not modeled
- The comparison covers consecutive time-point pairs on the BEELINE hESC and mESC datasets with cell-type-specific ground truth; other datasets or ground truths may behave differently

## Evidence

- [results] On the mESC and hESC scRNA-seq datasets, early precision computed against the BEELINE cell-type-specific ground truth at varying k showed that COSLIR's performance competed favorably with the four existing methods.: [evidence span withheld — anchor status: not_found]
- [abstract] Run COSLIR using first and second moments: [evidence span withheld — anchor status: not_found]
- [methods] Runtime environment: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)"
- [readme] Beta-version caveat: "Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present."
