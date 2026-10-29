---
name: covariance-moment-based-network-inference
description: Use when you have time-stamped cross-sectional single-cell expression
  data (qPCR or RNA-seq) from two consecutive time points and need to infer a sparse,
  directed, signed transition matrix A_t without constructing pseudo-time trajectories.
when_to_use_negative:
- Data comes from a single cell stage/time point only — most alternative methods for
  TSCS data infer networks maintaining the current cell state rather than mechanisms
  driving state transitions, and COSLIR requires two consecutive time points
- Only raw single-cell counts without reliable estimates of the first and second moments
  are available, or a true longitudinal time series of the same individual cells is
  expected (single-cell omics sacrifices cells per assay, so only TSCS data can be
  generated)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (Python version)
  role: Python implementation used to solve the covariance-restricted sparse linear
    regression and recover the sparse transition matrix A_t from supplied first and
    second moments
  repo: github.com/Ge-lab-pku/COSLIR
- name: COSLIR (MATLAB version)
  role: Alternative MATLAB implementation of the same model, tested on version 9.4
    (R2018a)
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy 1.14
  role: Only basic dependency; used to load the mean vector and covariance matrices
    and run the solver
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:504da4472aa34f03a63be94b9c191ae27ef4cb75405ef879b6e16b2c52b881f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# covariance-moment-based-network-inference

## Summary

Reconstruct directed, signed gene regulatory networks (GRNs) driving cell-state transitions from time-stamped cross-sectional (TSCS) single-cell data using COSLIR, which requires only the first and second moments (mean and covariance) of samples from two consecutive time points, bypassing pseudo-time inference.

## When to use

Use when you have time-stamped cross-sectional single-cell expression data (qPCR or RNA-seq) from two consecutive time points and need to infer a sparse, directed, signed transition matrix A_t without constructing pseudo-time trajectories.

## When NOT to use

- Data comes from a single cell stage/time point only — most alternative methods for TSCS data infer networks maintaining the current cell state rather than mechanisms driving state transitions, and COSLIR requires two consecutive time points
- Only raw single-cell counts without reliable estimates of the first and second moments are available, or a true longitudinal time series of the same individual cells is expected (single-cell omics sacrifices cells per assay, so only TSCS data can be generated)

## Inputs

- Mean vector (first moment) of gene expression at time point t, as a NumPy array
- Covariance matrices (second moments) of gene expression between two consecutive time points t and t+1, as NumPy arrays
- Optionally, ground-truth sparse interaction matrix for precision/recall evaluation (oracle case)

## Outputs

- Sparse directed signed transition matrix A_t encoding the GRN driving the cell-state transition
- Precision and recall values over directed signed edges (e.g., written to precision_recall_oracle.csv in validation runs)

## How to apply

Load the first and second moments (mean vector and covariance matrices) of samples between two consecutive time points as NumPy arrays, following the usage details in the DESCRIPTION files of the Python version of COSLIR (tested on Python v3.6.4 with NumPy v1.14; a MATLAB 9.4/R2018a version also exists). Solve the covariance-restricted sparse linear regression model to recover the sparse transition matrix A_t encoding directed signed edges. Following the README, the general workflow is to solve the model multiple times after bootstrapping and then clip on the rescaled value and the confidence to stabilize edge selection. For validation, compare the recovered A_t against a ground-truth network by computing precision and recall over the directed signed edges; in the oracle case (true mean and covariance supplied) both metrics should approach perfect accuracy.

## Related tools

- **COSLIR (Python version)** (Python implementation used to solve the covariance-restricted sparse linear regression and recover the sparse transition matrix A_t from supplied first and second moments) — github.com/Ge-lab-pku/COSLIR
- **COSLIR (MATLAB version)** (Alternative MATLAB implementation of the same model, tested on version 9.4 (R2018a)) — github.com/Ge-lab-pku/COSLIR
- **NumPy 1.14** (Only basic dependency; used to load the mean vector and covariance matrices and run the solver)

## Examples

```
import numpy as np; import coslir  # per python/DESCRIPTION; A_t = coslir.fit(mean_t, cov_t, cov_t1)  # recover sparse transition matrix from first/second moments
```

## Evaluation signals

- In the oracle case (true mean and covariance supplied), precision and recall over directed signed edges should closely mirror the ground truth, approaching perfect accuracy as reported in Fig. 2A-style simulations
- Recovered A_t should be sparse, consistent with the expectation that GRNs driving cell state transitions are sparse rather than dense
- Running time should remain nearly independent of the number of cells, since COSLIR uses only first and second moments
- Bootstrapped solutions should converge after the clipping step on rescaled value and confidence (per the README workflow)
- On real qPCR and RNA-seq developmental datasets, performance should compete favorably with existing GRN inference methods

## Limitations

- This is a beta version of COSLIR and some bugs may still remain; the README advises using it with caution and welcomes bug reports
- Performance depends on the quality of the first and second moments estimated from TSCS data; it avoids sensitivity to pseudo-time construction but does not eliminate all estimation error in real-world (non-oracle) scenarios
- Tested environments are narrow: Python v3.6.4 (Windows) with NumPy v1.14, or MATLAB 9.4 (R2018a)

## Evidence

- [abstract] COSLIR reconstructs GRNs using only the first and second moments of samples between two consecutive time points, bypassing single-cell temporal ordering: "utilizing only the first and second moments of samples between two consecutive time points"
- [abstract] Simulations validated COSLIR's perfect accuracy in the oracle case and demonstrated its robust performance in real-world scenarios.: "Simulations validated COSLIR's perfect accuracy in the oracle case and demonstrated its robust performance in real-world scenarios."
- [methods] Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage: "Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
- [readme] COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14).: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [readme] The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence.: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [readme] Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present.: "Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present."
