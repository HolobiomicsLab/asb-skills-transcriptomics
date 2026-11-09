---
name: cell-state-transition-network-modeling
description: Use when analyzing TSCS single-cell RT-PCR or RNA-seq data from two consecutive
  stages of a cell-state transition (e.g., ICM->EPI, ICM->PE) and needing a directed,
  weighted, sparse GRN without constructing pseudo-time.
when_to_use_negative:
- The data are a single cell stage/time point only, where the goal is the dense GRN
  maintaining the current state rather than the sparse mechanism driving a transition
- The data constitute a true longitudinal time series with linkage across time points
  rather than independently sampled, time-stamped cross-sectional batches
- Accurate pseudo-time trajectories already exist and a pseudo-time-based method (SINCERITIES,
  SCODE, SCRIBE, SINGE) is preferred for the analysis
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COSLIR (Python version)
  role: 'Primary inference engine: solves the covariance-restricted sparse linear
    regression on moments between two consecutive time points to output the directed
    sparse GRN'
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: COSLIR (MATLAB version)
  role: Alternative implementation of the same model, tested on MATLAB 9.4 (R2018a)
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: Python 3.6.4
  role: Runtime environment used to execute the Python version of COSLIR (Windows)
- name: NumPy 1.14
  role: Only basic dependency required by the Python version of COSLIR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:40b9bdd2f43f804c0d6b05dee82091362d68148740404eed71ea1cc6c7a4d54d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  - Python 3.6.4
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# cell-state-transition-network-modeling

## Summary

Reconstruct directed, sparse gene regulatory networks (GRNs) driving cell fate transitions from time-stamped cross-sectional (TSCS) single-cell data using COSLIR, which relies only on first and second moments of samples between consecutive time points and avoids pseudo-time inference.

## When to use

Use when analyzing TSCS single-cell RT-PCR or RNA-seq data from two consecutive stages of a cell-state transition (e.g., ICM->EPI, ICM->PE) and needing a directed, weighted, sparse GRN without constructing pseudo-time.

## When NOT to use

- The data are a single cell stage/time point only, where the goal is the dense GRN maintaining the current state rather than the sparse mechanism driving a transition
- The data constitute a true longitudinal time series with linkage across time points rather than independently sampled, time-stamped cross-sectional batches
- Accurate pseudo-time trajectories already exist and a pseudo-time-based method (SINCERITIES, SCODE, SCRIBE, SINGE) is preferred for the analysis

## Inputs

- Time-stamped cross-sectional (TSCS) single-cell RT-PCR expression matrix (e.g., 442-cell mouse embryo dataset bundled in the COSLIR repo)
- TSCS single-cell RNA-seq expression data
- Definition of two consecutive time points / cell stages defining the transition (e.g., ICM and EPI, ICM and PE)
- Optional reference regulatory edges for benchmarking

## Outputs

- Directed sparse GRN with edge signs and weights for the cell-state transition
- Early precision values (e.g., at k=1,...,40) when benchmarked against reference edges, e.g., early_precision_results.csv
- Comparison curves against SINCERITIES, SCODE, SCRIBE, and SINGE

## How to apply

Load the TSCS single-cell expression matrix (e.g., the 442-cell mouse embryo single-cell RT-PCR data shipped in the COSLIR repository) using Python 3.6.4 with NumPy 1.14 (or MATLAB 9.4/R2018a). Select the two consecutive cell stages defining the transition of interest, then run COSLIR (Python version), which solves a covariance-restricted sparse linear regression model using only the first and second moments of samples across the two time points. Follow the repo's general workflow: solve the model multiple times after bootstrapping, then apply a clipping on the rescaled value and the confidence to obtain the final directed sparse GRN with signs and weights. Validate downstream by evaluating inferred edges against reference regulatory edges, e.g., computing BEELINE-style early precision at k=1,...,40 and comparing with pseudo-time-based competitors such as SINCERITIES, SCODE, SCRIBE, and SINGE. Check the DESCRIPTION files in the preferred version folder for usage details.

## Related tools

- **COSLIR (Python version)** (Primary inference engine: solves the covariance-restricted sparse linear regression on moments between two consecutive time points to output the directed sparse GRN) — https://github.com/Ge-lab-pku/COSLIR
- **COSLIR (MATLAB version)** (Alternative implementation of the same model, tested on MATLAB 9.4 (R2018a)) — https://github.com/Ge-lab-pku/COSLIR
- **Python 3.6.4** (Runtime environment used to execute the Python version of COSLIR (Windows))
- **NumPy 1.14** (Only basic dependency required by the Python version of COSLIR)

## Evaluation signals

- Inferred network is directed, sparse, and carries edge signs and weights, as expected for a transition-driving GRN rather than a dense state-maintaining network
- Early precision against reference edges (BEELINE-style, k=1,...,40) shows COSLIR competing favorably with SINCERITIES, SCODE, SCRIBE, and SINGE, reproducing the Fig. 3E-F benchmark pattern
- In simulations, the oracle case yields perfect accuracy; real-data results should be robust across bootstrapped runs
- Running time remains nearly independent of the number of cells; large deviations suggest misconfiguration
- Bootstrap-and-clipping workflow is followed: model solved multiple times after bootstrapping with clipping on rescaled values and confidence

## Limitations

- The released COSLIR is a beta version; the README warns that 'Some bugs may still remain, so please use COSLIR with caution at present'
- COSLIR assumes the transition GRN is sparse; it is not designed for dense GRNs maintaining a single cell state
- Requires TSCS data with two consecutive, identifiable time points; it cannot be applied without stage/time-point structure
- Performance claims rest on qPCR and RNA-seq datasets validated in the paper; behavior on other data modalities is unverified

## Evidence

- [abstract] COSLIR reconstructs GRNs using only the first and second moments of samples between two consecutive time points, bypassing single-cell temporal ordering: "utilizing only the first and second moments of samples between two consecutive time points"
- [readme] The general workflow is bootstrap-then-clip: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [methods] Tested runtime environment: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [methods] Repository location of usage details: "Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
- [abstract] Benchmark outcome on developmental datasets: "When applied to single-cell RT-PCR and RNAseq datasets in developmental biology, COSLIR competed favorably with existing methods."
