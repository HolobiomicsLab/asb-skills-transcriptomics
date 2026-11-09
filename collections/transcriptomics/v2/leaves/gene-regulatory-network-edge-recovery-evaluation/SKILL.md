---
name: gene-regulatory-network-edge-recovery-evaluation
description: Use when a reconstructed GRN (directed signed edges/weights) must be
  scored against a known ground-truth network — e.g., oracle-case simulations with
  known true mean and covariance matrices — to quantify edge recovery via precision
  and recall.
when_to_use_negative:
- Input is real-world single-cell RT-PCR or RNA-seq TSCS data without a known ground-truth
  network — precision/recall against ground truth cannot be computed; the article
  only claims 'robust performance in real-world scenarios'
- The goal is inference of networks maintaining a single cell state rather than mechanisms
  driving cell-state transitions — the article notes most alternative methods infer
  the network that 'maintains the current cell stage'
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (COvariance restricted Sparse LInear Regression)
  role: Method whose optimization recovers the sparse transition matrix A_t from supplied
    first and second moments; the recovered matrix is the object evaluated
  repo: github.com/Ge-lab-pku/COSLIR
- name: COSLIR (Python version)
  role: Python implementation used to run the oracle-case recovery; usage details
    in the DESCRIPTION files of the python folder
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy 1.14
  role: Only basic dependency used to load true mean/covariance arrays and run COSLIR
    (tested on v1.14)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:16f1b2b44f7886338cbaf635df0e59045a45b65ae7038331417cf289165cd962
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (COvariance restricted Sparse LInear Regression)
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-regulatory-network-edge-recovery-evaluation

## Summary

Evaluate whether an inferred directed, signed, sparse gene regulatory network (e.g., the COSLIR transition matrix A_t) recovers a ground-truth network by computing precision and recall over directed signed edges, as validated in COSLIR's oracle-case simulations where the true mean and covariance matrices are supplied.

## When to use

Use when a reconstructed GRN (directed signed edges/weights) must be scored against a known ground-truth network — e.g., oracle-case simulations with known true mean and covariance matrices — to quantify edge recovery via precision and recall.

## When NOT to use

- Input is real-world single-cell RT-PCR or RNA-seq TSCS data without a known ground-truth network — precision/recall against ground truth cannot be computed; the article only claims 'robust performance in real-world scenarios'
- The goal is inference of networks maintaining a single cell state rather than mechanisms driving cell-state transitions — the article notes most alternative methods infer the network that 'maintains the current cell stage'

## Inputs

- True mean vector for the first time point (NumPy array)
- True covariance matrices between two consecutive time points (NumPy arrays)
- Ground-truth sparse interaction matrix A_t with directed signed edges
- COSLIR Python implementation from github.com/Ge-lab-pku/COSLIR (DESCRIPTION file usage details)

## Outputs

- Recovered sparse transition matrix A_t (directed, signed, weighted edges)
- precision_recall_oracle.csv containing precision and recall values over directed signed edges

## How to apply

Load the oracle-case simulation inputs — the true mean vector and the true covariance matrices between two consecutive time points — as NumPy arrays, following the usage details in the DESCRIPTION files of the Python version of COSLIR. Execute COSLIR (COvariance restricted Sparse LInear Regression), tested on Python v3.6.4 with NumPy v1.14, to recover the sparse transition matrix A_t from the supplied first and second moments; COSLIR's general workflow solves the model multiple times after bootstrapping and applies a clipping on the rescaled value and the confidence. Compare the recovered sparse A_t against the ground-truth network, computing precision and recall over the directed signed edges. Write the metrics to precision_recall_oracle.csv and check that both precision and recall approach the perfect accuracy reported for the oracle case. The rationale is that COSLIR reconstructs GRNs using only first and second moments between consecutive time points, so in the oracle case recovery should be near-exact, providing a clean upper-bound benchmark.

## Related tools

- **COSLIR (COvariance restricted Sparse LInear Regression)** (Method whose optimization recovers the sparse transition matrix A_t from supplied first and second moments; the recovered matrix is the object evaluated) — github.com/Ge-lab-pku/COSLIR
- **COSLIR (Python version)** (Python implementation used to run the oracle-case recovery; usage details in the DESCRIPTION files of the python folder) — github.com/Ge-lab-pku/COSLIR
- **NumPy 1.14** (Only basic dependency used to load true mean/covariance arrays and run COSLIR (tested on v1.14))

## Examples

```
import numpy as np; from coslir import coslir; A_t = coslir(mean_t1, cov_t1, cov_t2)
```

## Evaluation signals

- precision_recall_oracle.csv is written and both precision and recall approach the perfect accuracy reported for the oracle case
- Recovered sparse A_t matches ground truth over directed signed edges (near-exact recovery, with precision and recall 'closely mirroring the ground-truth values')
- COSLIR runs under its tested environment (Python v3.6.4, Windows, NumPy v1.14) using only NumPy as a dependency
- Recovery relies only on first and second moments between two consecutive time points — no pseudo-time inference inputs are used
- Runtime remains nearly independent of the number of cells, consistent with the reported scaling property

## Limitations

- The distributed COSLIR is a beta version: 'Some bugs may still remain, so please use COSLIR with caution at present'
- Perfect accuracy is established only for the oracle case with known true mean and covariance; real-world scenarios are described as robust but not perfect
- Single-cell omics experiments sacrifice individual cells per assay, so only time-stamped cross-sectional (TSCS) data — not true longitudinal time series — can be generated

## Evidence

- [abstract] oracle-case perfect recovery: "Simulations validated COSLIR's perfect accuracy in the oracle case and demonstrated its robust performance in real-world scenarios."
- [abstract] moments-only model: [evidence span withheld — anchor status: not_found]
- [methods] usage via DESCRIPTION: "Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
- [methods] tested environment: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [readme] bootstrapping and clipping workflow: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [readme] beta-version caution: "Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present."
