---
name: diffusion-pseudotime-dpt-ordering
description: Use when single cells must be ordered along a differentiation continuum
  with a diffusion-based method (Scanpy DPT or RWR-based DTFLOW), especially when
  known developmental stage labels allow scoring by Kendall rank correlation.
when_to_use_negative:
- The goal is resolving multiple bifurcation events in a branching trajectory — plain
  DPT yields a linear ordering only; the article introduces RSKG (Reverse Searching
  on k-nearest neighbor graph) for multi-branching detection.
- 'A restart-augmented diffusion can be tuned instead: the paper reports ordering
  accuracy is better with p around 0.9, so p=1 DPT may be dominated by the general
  RWR formulation on the same data.'
- The input is not a cell-level gene expression matrix (e.g., bulk profiles or a metadata
  table) — the kNN Markov graph and per-cell diffusion distributions require an N
  x D matrix of single cells.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DTFLOW
  role: Reference implementation of the RWR/BKFD pseudotime pipeline of which DPT
    is the p=1 special case; get_transition_matrix(k) builds the kNN Markov matrix
    and get_diffusionRWR(p) runs the diffusion step
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Provides the DPT implementation used as the p=1 diffusion pseudotime baseline,
    scored Kendall tau 0.796 on the ME dataset
- name: Monocle2
  role: R-based pseudotime baseline in the same comparison, scored Kendall tau 0.761
    on the ME dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:181186ecfb95639479779955e8353f63c081582ac177ecb44ace3fae433bd5d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# diffusion-pseudotime-dpt-ordering

## Summary

Diffusion pseudotime (DPT) ordering of single cells along a differentiation trajectory, framed by the DTFLOW paper as the p=1 special case of random walk with restart (RWR); on the ME mouse-embryo RT-PCR dataset (438 cells, 48 genes) DPT reached Kendall tau 0.796 versus 0.862 for DTFLOW with p=0.9 and 0.761 for Monocle2.

## When to use

Use when single cells must be ordered along a differentiation continuum with a diffusion-based method (Scanpy DPT or RWR-based DTFLOW), especially when known developmental stage labels allow scoring by Kendall rank correlation.

## When NOT to use

- The goal is resolving multiple bifurcation events in a branching trajectory — plain DPT yields a linear ordering only; the article introduces RSKG (Reverse Searching on k-nearest neighbor graph) for multi-branching detection.
- A restart-augmented diffusion can be tuned instead: the paper reports ordering accuracy is better with p around 0.9, so p=1 DPT may be dominated by the general RWR formulation on the same data.
- The input is not a cell-level gene expression matrix (e.g., bulk profiles or a metadata table) — the kNN Markov graph and per-cell diffusion distributions require an N x D matrix of single cells.

## Inputs

- Pre-processed gene expression matrix X (N cells x D genes), e.g., the ME RT-PCR dataset with 438 cells and 48 genes
- Time-stage labels per cell (e.g., seven developmental stages, 1-cell zygote to 64-cell blastocyst) used to score the ordering
- Root cell specification (root_list) anchoring the pseudotime origin

## Outputs

- Per-cell pseudotime values (dftimes) defining a total ordering of all cells
- Kendall rank correlation coefficient (kendall_tau) per method, e.g., written to kendall_correlations.csv with columns method and kendall_tau

## How to apply

Pre-process the data into a gene expression matrix X (N cells x D genes), e.g., the guo2010 ME RT-PCR matrix of 438 cells and 48 genes spanning seven stages from 1-cell zygote to 64-cell blastocyst. Build a k-nearest-neighbor Markov transition matrix (the paper's sweep found k=10 best) and run diffusion propagation: DPT is the special case of RWR with p=1 (restart probability 1-p=0), whereas DTFLOW's improved variant restarts with p around 0.9. Extract per-cell pseudotime from a chosen root via z.get_dtflow_times(root_list) in the DTFLOW API, or run Scanpy's DPT implementation on the same preprocessed matrix. Score ordering accuracy as the Kendall rank correlation between inferred pseudotime and the known developmental stage ordering; on the ME dataset this gave 0.796 for Scanpy DPT, 0.862 for DTFLOW (p=0.9), and 0.761 for Monocle2. Decision point: if plain DPT underperforms, switch to the general RWR with p around 0.9, which the article reports yields better ordering accuracy.

## Related tools

- **DTFLOW** (Reference implementation of the RWR/BKFD pseudotime pipeline of which DPT is the p=1 special case; get_transition_matrix(k) builds the kNN Markov matrix and get_diffusionRWR(p) runs the diffusion step) — https://github.com/statway/DTFLOW
- **Scanpy** (Provides the DPT implementation used as the p=1 diffusion pseudotime baseline, scored Kendall tau 0.796 on the ME dataset)
- **Monocle2** (R-based pseudotime baseline in the same comparison, scored Kendall tau 0.761 on the ME dataset)

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(1); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- Kendall tau for the DPT ordering on the ME dataset should land near the reported 0.796 (Scanpy DPT), with DTFLOW (p=0.9) at 0.862 and Monocle2 at 0.761 as reference bounds; a large deviation signals a preprocessing or root-cell error.
- Diffusion configuration check: DPT requires p=1, i.e., restart probability 1-p=0 in z.get_diffusionRWR(p); any other p runs general RWR rather than DPT.
- The Markov transition matrix must use k=10 nearest neighbors, per the paper's parameter sweep.
- Output schema check: kendall_correlations.csv contains columns method and kendall_tau with one row each for DTFLOW, Scanpy, and Monocle2.
- The inferred ordering must cover all 438 ME cells exactly once, and Kendall tau must be computed against the seven developmental stage labels (1-cell zygote to 64-cell blastocyst).

## Limitations

- Optimal parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized in the paper.
- DPT fixes p=1, while the paper found ordering accuracy is better with p around 0.9, so plain DPT can underperform the general RWR on the same dataset.
- BKFD/RWR uses the same restart probability for all nodes, which may limit the effectiveness of the random walk (article discussion).
- No software versions are reported in the paper, so environment reproducibility must be pinned separately.

## Evidence

- [methods] DPT is the p=1 special case of the RWR algorithm used by DTFLOW: "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
- [results] Benchmark Kendall taus on the ME dataset: DTFLOW 0.862, Scanpy 0.796, Monocle2 0.761: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [results] Ordering accuracy is better when p is around 0.9, adopted throughout the paper: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [readme] In the DTFLOW API, 1-p is the restart probability of the RWR processes: "Here 1-p is the restart probability of RWR processes."
- [results] k=10 nearest neighbors gave the best results in the parameter sweep: "We test different values of k and find that the results are better if k=10"
- [discussion] Parameter values are not standardized across datasets: "the values of these parameters may vary from dataset to dataset."
