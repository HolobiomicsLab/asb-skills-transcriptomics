---
name: bootstrap-edge-confidence-thresholding
description: Use when inferred GRN edges need confidence scores before final network
  selection, and when precision/recall must be characterized as a function of an edge-confidence
  threshold rather than at a single arbitrary cutoff.
when_to_use_negative:
- The input is already a validated, single-point network estimate with no cell-level
  replicates to bootstrap (no resampling variability to estimate confidence from)
- The goal is to compare methods on a fixed network without threshold calibration
  — a single un-thresholded COSLIR run suffices
tools:
- name: COSLIR (Python version)
  role: Inference engine re-solved on each bootstrap resample to generate edge-stability
    counts; produces the rescaled values and confidences that are clipped
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Numerical backend for bootstrapping, moment computation, and threshold sweeping
    in the Python implementation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:c72f37a728a0d85fbac318a4cff8ba33ef89d8fdbf31f06c9ca680a74594456b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (Python version)
  - NumPy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bootstrap-edge-confidence-thresholding

## Summary

Assigns confidence scores to COSLIR-inferred GRN edges by repeatedly solving the covariance-restricted sparse linear regression model on bootstrapped resamples, then sweeping the confidence threshold to trace precision/recall trade-offs (e.g., Fig. 2D).

## When to use

Use when inferred GRN edges need confidence scores before final network selection, and when precision/recall must be characterized as a function of an edge-confidence threshold rather than at a single arbitrary cutoff.

## When NOT to use

- The input is already a validated, single-point network estimate with no cell-level replicates to bootstrap (no resampling variability to estimate confidence from)
- The goal is to compare methods on a fixed network without threshold calibration — a single un-thresholded COSLIR run suffices

## Inputs

- Simulated or measured single-cell expression data (first and second moments — mean and covariance — between two consecutive time points)
- Inferred directed, signed, weighted GRN edges from COSLIR
- Optional oracle (true) interaction matrix for computing precision/recall in simulations

## Outputs

- Per-edge confidence scores from bootstrap replicates
- Precision/recall values as a function of confidence threshold (Fig. 2D-style curve, e.g., saved as precision_recall_curves.png)
- Final clipped edge list with rescaled values and confidence scores

## How to apply

Run the COSLIR inference (Python/NumPy version, covariance-restricted sparse linear regression on first and second moments between consecutive time points) multiple times, each on a bootstrapped resample of the cells, per the README's stated workflow of solving the model repeatedly after bootstrapping. Score each inferred edge by how consistently it is recovered across bootstrap replicates, then apply the clipping to the rescaled edge value and the confidence. Sweep the confidence threshold across a range and record precision and recall of the retained edge set at each threshold against the oracle interaction matrix, producing a precision/recall-vs-threshold curve as in Fig. 2D. Decision point: choose the operating threshold from the curve's precision/recall balance for the downstream network rather than fixing it a priori.

## Related tools

- **COSLIR (Python version)** (Inference engine re-solved on each bootstrap resample to generate edge-stability counts; produces the rescaled values and confidences that are clipped) — github.com/Ge-lab-pku/COSLIR
- **NumPy** (Numerical backend for bootstrapping, moment computation, and threshold sweeping in the Python implementation)

## Evaluation signals

- In the oracle simulation, precision and recall at the optimal threshold should approach perfect accuracy as reported for COSLIR
- The precision/recall-vs-threshold curve should show the monotone trade-off expected as the confidence cutoff is varied (Fig. 2D trend)
- Every retained edge must carry both a rescaled value and a confidence score after the clipping step
- Bootstrap replicate runs should yield consistent edge sets (high-frequency edges) rather than unstable, replicate-specific edges

## Limitations

- COSLIR is a beta version and 'some bugs may still remain, so please use COSLIR with caution at present'
- Confidence estimates depend on bootstrap resampling of cells; sample size affects performance — precision and recall increased as the number of cells grew (Fig. 2C), so small samples may yield noisy confidence scores
- No changelog is available for the repository, so version-specific behavior of the bootstrap/clipping code cannot be tracked

## Evidence

- [readme] Bootstrap workflow: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [results] Sample-size effect on precision/recall: [evidence span withheld — anchor status: not_found]
- [methods] Bootstrap confidence scoring step: [evidence span withheld — anchor status: not_found]
- [abstract] Runtime independence supports repeated bootstrap solves: "Notably, its running time remained nearly independent of the number of cells."
