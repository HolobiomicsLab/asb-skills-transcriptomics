---
name: early-precision-curve-evaluation
description: Use when comparing GRN inference methods on time-stamped cross-sectional
  (TSCS) single-cell data where a reference regulatory network (e.g. BEELINE hESC/mESC
  ground truth) exists and performance at the top of the ranked edge list matters
  more than full-network accuracy.
when_to_use_negative:
- The data lack a reference/ground-truth regulatory network, so early precision cannot
  be computed against known edges
- The goal is to evaluate GRNs that maintain a single cell stage rather than transition
  mechanisms — the article notes most alternative methods infer networks for a single
  stage, while early precision here targets sparse transition GRNs between consecutive
  time points
- The method requires pseudo-time trajectories as input (e.g. pseudo-time-dependent
  baselines evaluated on trajectories COSLIR does not use); the evaluation design
  must instead score networks inferred from the same TSCS time-point pairs
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: COSLIR (Python version)
  role: Produces the bootstrapped regression coefficient matrix and first/second moments
    from which rescaled interaction scores and the ranked edge list are derived for
    early-precision evaluation
  repo: github.com/Ge-lab-pku/COSLIR
- name: Python 3.6.4 + NumPy 1.14
  role: Runtime for computing early precision over ranked edges and writing result
    CSVs
- name: SINCERITIES / SCODE / SCRIBE / SINGE
  role: Baseline GRN inference methods whose inferred networks on the same consecutive
    time-point pairs are evaluated under the identical early-precision protocol
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:11481f660c0cbacb36a227b4c363d09a020e3370f57c76b68c5258f7137c0958
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  - Python 3.6.4 + NumPy 1.14
  - SINCERITIES / SCODE / SCRIBE / SINGE
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# early-precision-curve-evaluation

## Summary

Evaluates inferred gene regulatory networks (GRNs) by computing early precision against cell-type-specific or reference ground-truth edges, as used in the COSLIR paper to benchmark COSLIR against SINCERITIES, SCODE, SCRIBE, and SINGE on TSCS single-cell qPCR and RNA-seq data.

## When to use

Use when comparing GRN inference methods on time-stamped cross-sectional (TSCS) single-cell data where a reference regulatory network (e.g. BEELINE hESC/mESC ground truth) exists and performance at the top of the ranked edge list matters more than full-network accuracy.

## When NOT to use

- The data lack a reference/ground-truth regulatory network, so early precision cannot be computed against known edges
- The goal is to evaluate GRNs that maintain a single cell stage rather than transition mechanisms — the article notes most alternative methods infer networks for a single stage, while early precision here targets sparse transition GRNs between consecutive time points
- The method requires pseudo-time trajectories as input (e.g. pseudo-time-dependent baselines evaluated on trajectories COSLIR does not use); the evaluation design must instead score networks inferred from the same TSCS time-point pairs

## Inputs

- Ranked list of inferred directed regulatory interactions with edge scores (e.g. rescaled values from bootstrapped COSLIR regression coefficients and first-moment vectors mu1, mu2)
- Ground-truth regulatory network (e.g. BEELINE cell-type-specific ground truth for hESC and mESC, or reference edges for the ICM->EPI / ICM->PE mouse embryo RT-PCR transitions)
- Inferred networks from comparison methods (SINCERITIES, SCODE, SCRIBE, SINGE) on the same consecutive time-point pairs

## Outputs

- Early-precision values at each k (k = 1,...,40) for every method and dataset (early_precision_results.csv)
- Early-precision curves comparing methods (e.g. fig4ab_early_precision_curves.png for hESC and mESC)
- Ranked interaction table with regulator, target, and rescaled-value columns (ranked_interactions.csv)

## How to apply

First rank all candidate directed interactions by an edge score — for COSLIR, the rescaled value (|Â|)_ij · |μ̂1|_j / |(μ̂1)_i − (μ̂2)_i| aggregated across bootstrap replicates — in descending order. Then, for each ranked edge list, sweep k over the top-ranked edges (k = 1,2,...,40 in the BEELINE-style setup used for the 442-cell mouse embryo RT-PCR ICM->EPI and ICM->PE transitions) and compute, at each k, the fraction of the top-k inferred edges that are present in the ground-truth network; plotting these values yields the early-precision curve. Apply the same ranking-and-sweep procedure to every competing method's inferred network on the same consecutive time-point pairs so curves are directly comparable, e.g. COSLIR versus SINCERITIES, SCODE, SCRIBE, and SINGE on the hESC and mESC scRNA-seq datasets (Fig. 4A-B). The rationale is that transition GRNs are sparse, so recovery of the few true regulatory edges at small k is the discriminative metric; report the curves per dataset and per transition (e.g. fig4ab_early_precision_curves.png, early_precision_results.csv).

## Related tools

- **COSLIR (Python version)** (Produces the bootstrapped regression coefficient matrix and first/second moments from which rescaled interaction scores and the ranked edge list are derived for early-precision evaluation) — github.com/Ge-lab-pku/COSLIR
- **Python 3.6.4 + NumPy 1.14** (Runtime for computing early precision over ranked edges and writing result CSVs)
- **SINCERITIES / SCODE / SCRIBE / SINGE** (Baseline GRN inference methods whose inferred networks on the same consecutive time-point pairs are evaluated under the identical early-precision protocol)

## Examples

```
import numpy as np; rescaled = np.abs(A) * np.abs(mu1) / np.abs(mu1 - mu2); ranked = np.dstack(np.unravel_index(np.argsort(-rescaled, axis=None), rescaled.shape))
```

## Evaluation signals

- ranked_interactions.csv parses cleanly with NumPy/pandas, contains one row per non-zero bootstrapped interaction with finite rescaled values
- Early-precision values are computed at k = 1,2,...,40 against the reference regulatory edges in a BEELINE-style setup
- On hESC and mESC, COSLIR's early-precision values rank on par with the best-performing baseline method on both datasets, matching the qualitative pattern of Fig. 4A-B
- On the RT-PCR mouse embryo ICM->EPI and ICM->PE transitions, the curves reproduce the Fig. 3E-F benchmark pattern in which COSLIR competed favorably with the pseudo-time-reliant methods
- Top-ranked rows in the ranked table correspond to the highest rescaled scores available for downstream early-precision evaluation

## Limitations

- Evaluation requires trusted ground-truth networks; conclusions are limited to datasets where such references exist (BEELINE hESC/mESC, mouse embryo reference edges)
- COSLIR itself is a beta version — the README states some bugs may still remain and users should apply it with caution
- Early precision only assesses the top of the ranked edge list and presumes sparse transition GRNs; it is not a whole-network accuracy metric

## Evidence

- [abstract] When applied to single-cell RT-PCR and RNAseq datasets in developmental biology, COSLIR competed favorably with existing methods.: "When applied to single-cell RT-PCR and RNAseq datasets in developmental biology, COSLIR competed favorably with existing methods."
- [abstract] COSLIR reconstructs GRNs from first and second moments: "utilizing only the first and second moments of samples between two consecutive time points"
- [intro] GRNs that drive cell state transitions are sparse: "GRNs that drive cell state transitions are considered sparse"
- [readme] README: usage details in DESCRIPTION files: "Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
- [readme] README: bootstrapping and clipping workflow: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
