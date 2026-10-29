---
name: sparse-linear-regression-matrix-recovery
description: Use when you have TSCS single-cell expression data (qPCR or RNA-seq)
  at two or more consecutive time points and need to reconstruct a directed, signed,
  sparse GRN driving a cell-state transition without constructing pseudo-time trajectories.
when_to_use_negative:
- You have true longitudinal time-series data with linkage across time points for
  the same cells, rather than time-stamped cross-sectional (TSCS) data
- The goal is to infer a network that maintains the current cell stage rather than
  the mechanism driving a cell-state transition
- Only a single cell stage/time point is available, so no two consecutive time-point
  moments can be supplied
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (COvariance restricted Sparse LInear Regression)
  role: 'Core method: covariance-restricted sparse linear regression that recovers
    the sparse transition matrix A_t from first and second moments'
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: COSLIR (Python version)
  role: Python implementation used to run COSLIR; consult its DESCRIPTION file for
    usage details
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: COSLIR (MATLAB version)
  role: Alternative MATLAB implementation of COSLIR for running the same recovery
    workflow
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Only basic dependency (tested on v1.14) used to load moment inputs and hold
    the recovered matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:f50953cd95b7675525b3750ce098511e5cb9c52128acb6f5ba88675b0653e5bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (COvariance restricted Sparse LInear Regression)
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# sparse-linear-regression-matrix-recovery

## Summary

Recover a sparse directed interaction matrix (e.g., a gene regulatory network transition matrix A_t) from time-stamped cross-sectional single-cell data using COSLIR (COvariance restricted Sparse LInear Regression), which relies exclusively on the first and second moments of samples between two consecutive time points, bypassing pseudo-time inference.

## When to use

Use when you have TSCS single-cell expression data (qPCR or RNA-seq) at two or more consecutive time points and need to reconstruct a directed, signed, sparse GRN driving a cell-state transition without constructing pseudo-time trajectories.

## When NOT to use

- You have true longitudinal time-series data with linkage across time points for the same cells, rather than time-stamped cross-sectional (TSCS) data
- The goal is to infer a network that maintains the current cell stage rather than the mechanism driving a cell-state transition
- Only a single cell stage/time point is available, so no two consecutive time-point moments can be supplied

## Inputs

- First and second moments (mean vector and covariance matrix) of single-cell expression samples at two consecutive time points (NumPy arrays); in oracle-case simulations, the true mean and covariance matrices
- Ground-truth sparse interaction matrix A_t (for validation in simulation settings)

## Outputs

- Recovered sparse directed signed transition matrix A_t (GRN)
- Precision and recall over directed signed edges (e.g., written to precision_recall_oracle.csv)

## How to apply

Load the sample moments (the first/second moments, or in the oracle case the true mean and covariance matrices) between two consecutive time points as NumPy arrays, following the usage details in the DESCRIPTION files of the Python or MATLAB version of COSLIR. Execute COSLIR, a covariance-restricted sparse linear regression, to recover the sparse transition matrix A_t from the supplied moments. Apply the general workflow of solving the model multiple times after bootstrapping, then clipping on the rescaled value and the confidence to binarize/sign the edges. Compare the recovered sparse matrix against a ground-truth network by computing precision and recall over directed signed edges (oracle-case simulations show almost exact recovery), and write metrics such as precision_recall_oracle.csv.

## Related tools

- **COSLIR (COvariance restricted Sparse LInear Regression)** (Core method: covariance-restricted sparse linear regression that recovers the sparse transition matrix A_t from first and second moments) — https://github.com/Ge-lab-pku/COSLIR
- **COSLIR (Python version)** (Python implementation used to run COSLIR; consult its DESCRIPTION file for usage details) — https://github.com/Ge-lab-pku/COSLIR
- **COSLIR (MATLAB version)** (Alternative MATLAB implementation of COSLIR for running the same recovery workflow) — https://github.com/Ge-lab-pku/COSLIR
- **NumPy** (Only basic dependency (tested on v1.14) used to load moment inputs and hold the recovered matrix)

## Examples

```
# in the python/ folder of Ge-lab-pku/COSLIR (see DESCRIPTION for details): import numpy as np; from coslir import coslir; A_t = coslir(mean_t0, mean_t1, cov_t0, cov_t1)
```

## Evaluation signals

- In the oracle case (true mean and covariance supplied), precision and recall of the recovered A_t should closely mirror the ground truth, approaching the perfect accuracy reported for simulations
- Precision/recall computed over directed signed edges should be written to precision_recall_oracle.csv and inspected against expected perfect-accuracy values
- The recovered A_t should be sparse, consistent with the premise that GRNs driving cell state transitions are sparse
- Running time should remain nearly independent of the number of cells; a strong scaling with cell count suggests incorrect usage
- Bootstrapped solutions, after clipping on the rescaled value and confidence, should yield stable edges across resampling runs

## Limitations

- The released COSLIR is a beta version in which some bugs may still remain, so results should be used with caution
- Performance was validated in the oracle case with perfect accuracy and robustly in real qPCR and RNA-seq datasets, but real-world scenarios lack ground truth for exact verification
- COSLIR uses only first and second moments, so it cannot exploit higher-order distributional structure or true longitudinal linkage across time points

## Evidence

- [abstract] COSLIR uses only first and second moments: "utilizing only the first and second moments of samples between two consecutive time points"
- [abstract] oracle-case perfect accuracy: "Simulations validated COSLIR's perfect accuracy in the oracle case and demonstrated its robust performance in real-world scenarios."
- [readme] bootstrap-and-clip workflow: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [methods] check DESCRIPTION files for usage: "Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
- [methods] Python/NumPy test environment: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [readme] beta version caveat: "Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present."
