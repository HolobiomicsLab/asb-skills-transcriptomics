---
name: cross-method-grn-comparison-analysis
description: Use when evaluating a GRN inference method on time-stamped cross-sectional
  single-cell data against competitors (e.g., SINCERITIES, SCODE, SCRIBE, SINGE) using
  early-precision curves.
when_to_use_negative:
- The dataset is a true longitudinal time series from the same cells rather than time-stamped
  cross-sectional (TSCS) data, so pseudo-time-free comparison framing is unnecessary.
- The goal is to infer the dense GRN maintaining a single cell stage rather than the
  sparse GRN driving a cell state transition.
- No reference regulatory edges are available, so early precision cannot be computed
  against ground truth.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COSLIR (Python version)
  role: Reconstructs the directed sparse GRN from first and second moments of samples
    between consecutive time points; the method under benchmark
  repo: github.com/Ge-lab-pku/COSLIR
- name: COSLIR (MATLAB version)
  role: Alternative implementation of the same inference method (tested on MATLAB
    9.4/R2018a) usable for cross-checking
  repo: github.com/Ge-lab-pku/COSLIR
- name: Python 3.6.4
  role: Runtime environment in which the Python COSLIR version and the benchmark evaluation
    are executed
- name: NumPy 1.14
  role: Basic numeric dependency for running COSLIR and computing early precision
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:6f59e813c1b3a3509fb9d3b19c76101766cfd033a785032cb8f63e132eed4b59
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  - Python 3.6.4
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-method-grn-comparison-analysis

## Summary

Benchmark a newly inferred GRN method (here COSLIR on TSCS single-cell data) against established pseudo-time-based GRN inference algorithms by computing early precision at increasing edge counts (k=1..40) against reference regulatory edges, as done for the RT-PCR mouse embryo ICM->EPI and ICM->PE transitions (Fig. 3E-F).

## When to use

Use when evaluating a GRN inference method on time-stamped cross-sectional single-cell data against competitors (e.g., SINCERITIES, SCODE, SCRIBE, SINGE) using early-precision curves.

## When NOT to use

- The dataset is a true longitudinal time series from the same cells rather than time-stamped cross-sectional (TSCS) data, so pseudo-time-free comparison framing is unnecessary.
- The goal is to infer the dense GRN maintaining a single cell stage rather than the sparse GRN driving a cell state transition.
- No reference regulatory edges are available, so early precision cannot be computed against ground truth.

## Inputs

- 442-cell mouse embryo single-cell RT-PCR expression data (shipped in the COSLIR repository)
- Reference (ground-truth) regulatory edges for the ICM->EPI and ICM->PE cell fate transitions
- Published early-precision curves of SINCERITIES, SCODE, SCRIBE, and SINGE from Fig. 3E-F for comparison

## Outputs

- early_precision_results.csv containing early-precision values at k=1..40 for COSLIR on each transition
- Early-precision comparison against the four existing GRN inference methods
- Inferred directed sparse GRNs (with signs and weights) for ICM->EPI and ICM->PE transitions

## How to apply

Load the target single-cell expression dataset — for this benchmark, the 442-cell mouse embryo single-cell RT-PCR data shipped in the Ge-lab-pku/COSLIR repository — and run COSLIR (Python version, tested on Python v3.6.4 with NumPy v1.14) on each cell-fate transition of interest (ICM->EPI and ICM->PE), reconstructing the directed sparse GRN from only the first and second moments of samples between the two consecutive time points. Then evaluate each inferred network under a BEELINE-style setup by computing early precision at k=1,2,...,40 against the reference regulatory edges. Plot or tabulate the resulting early-precision curves and compare them to the published curves of SINCERITIES, SCODE, SCRIBE, and SINGE from Fig. 3E-F, which rely on pseudo-time construction. The rationale is that COSLIR bypasses pseudo-time inference, so parity in early precision with pseudo-time-dependent methods demonstrates competitive performance with fewer assumptions.

## Related tools

- **COSLIR (Python version)** (Reconstructs the directed sparse GRN from first and second moments of samples between consecutive time points; the method under benchmark) — github.com/Ge-lab-pku/COSLIR
- **COSLIR (MATLAB version)** (Alternative implementation of the same inference method (tested on MATLAB 9.4/R2018a) usable for cross-checking) — github.com/Ge-lab-pku/COSLIR
- **Python 3.6.4** (Runtime environment in which the Python COSLIR version and the benchmark evaluation are executed)
- **NumPy 1.14** (Basic numeric dependency for running COSLIR and computing early precision)

## Examples

```
python coslir.py --data rt_pcr_embryo_442cells --transition ICM_to_EPI; # then evaluate early precision at k=1..40 against reference edges and write early_precision_results.csv
```

## Evaluation signals

- The early-precision curve for COSLIR on ICM->EPI and ICM->PE reproduces the Fig. 3E-F pattern in which COSLIR competed favorably with SINCERITIES, SCODE, SCRIBE, and SINGE.
- Early precision is computed at k=1,2,...,40 edges, matching the BEELINE-style evaluation setup used in the paper.
- The inferred networks are directed, sparse, and carry signs and weights, reconstructed without any pseudo-time construction.
- Only the first and second moments of samples between the two consecutive time points are used as inference inputs.
- Results for both transitions (ICM->EPI and ICM->PE) are written to early_precision_results.csv.

## Limitations

- This is a beta version of COSLIR; the README warns that some bugs may still remain and asks users to apply it with caution.
- The comparison depends on competitor curves (SINCERITIES, SCODE, SCRIBE, SINGE) reported in Fig. 3E-F, which rely on pseudo-time construction and whose accuracy is sensitive to pseudo-time trajectory precision.
- COSLIR's GRN inference is restricted to two consecutive time points of TSCS data and cannot use true temporal linkage across time points.

## Evidence

- [results] On the RT-PCR mouse embryo dataset, COSLIR competed favorably with the four existing methods, which rely on pseudo-time construction, in early precision (Fig. 3E-F).: [evidence span withheld — anchor status: not_found]
- [intro] Existing GRN inference methods are sensitive to pseudo-time construction quality, motivating a pseudo-time-free benchmark.: [evidence span withheld — anchor status: not_found]
- [methods] COSLIR runtime and dependency requirements for the benchmark environment.: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)"
- [abstract] COSLIR models the transition using only first and second moments between consecutive time points.: "utilizing only the first and second moments of samples between two consecutive time points"
- [readme] Repository structure and usage instructions for the Python and MATLAB versions.: "the MATLAB version and the python version of COSLIR. Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
