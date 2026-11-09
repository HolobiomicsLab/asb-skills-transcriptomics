---
name: kendall-rank-correlation-benchmark-comparison
description: Use when comparing pseudotime trajectory inference methods on a dataset
  whose true cell ordering (e.g., known developmental stages) is available and you
  need a rank-based ordering-accuracy metric per method.
when_to_use_negative:
- No ground-truth ordering or stage labels exist for the cells — Kendall tau against
  a reference ordering cannot be computed
- The goal is branch/bifurcation detection quality rather than linear ordering accuracy
  (use DTFLOW's RSKG branch detection evaluation instead)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: DTFLOW
  role: Primary pseudotime method being benchmarked; run with k=10, p=0.9, SVD solver,
    and z.get_dtflow_times(root_list)
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Provides the DPT pseudotime implementation (special case of RWR with p=1)
    used as the comparison baseline
- name: Monocle2
  role: R-based pseudotime method used as the second comparison baseline
- name: Python
  role: Implementation language for DTFLOW and the Kendall tau computation
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:49c014ab1e3588ab25d93bc2aa81d25e096f18a919710767c97982244eea0304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# kendall-rank-correlation-benchmark-comparison

## Summary

Benchmark pseudotime inference methods by computing the Kendall rank correlation coefficient between each method's inferred cell ordering and the known developmental stage ordering, as done for DTFLOW vs Scanpy (DPT) vs Monocle2 on the ME mouse-embryo RT-PCR dataset.

## When to use

Use when comparing pseudotime trajectory inference methods on a dataset whose true cell ordering (e.g., known developmental stages) is available and you need a rank-based ordering-accuracy metric per method.

## When NOT to use

- No ground-truth ordering or stage labels exist for the cells — Kendall tau against a reference ordering cannot be computed
- The goal is branch/bifurcation detection quality rather than linear ordering accuracy (use DTFLOW's RSKG branch detection evaluation instead)

## Inputs

- Pre-processed gene expression matrix (cells x genes) shared by all methods, e.g., the ME RT-PCR dataset with 438 cells and 48 genes
- Ground-truth time-stage labels / known developmental stage ordering per cell (time_stages)
- Pseudotime orderings from each method (DTFLOW dftimes, Scanpy DPT pseudotime, Monocle2 pseudotime)

## Outputs

- Kendall rank correlation coefficient (kendall_tau) per method
- CSV table (e.g., kendall_correlations.csv) with columns method and kendall_tau

## How to apply

Run each method (e.g., DTFLOW with k=10 nearest neighbors for the Markov transition matrix, RWR restart probability p=0.9, SVD-based BKFD embedding, and z.get_dtflow_times(root_list); Scanpy's DPT implementation, which is the p=1 special case of RWR; and Monocle2 in R) on the same pre-processed gene expression matrix. For each method, rank the cells by inferred pseudotime and compute scipy.stats.kendalltau between that ordering and the ground-truth stage ordering of all cells (e.g., 438 ME cells across seven stages from 1-cell zygote to 64-cell blastocyst). Higher tau indicates better ordering accuracy. Write results to a table with columns method and kendall_tau and compare against published reference values where available.

## Related tools

- **DTFLOW** (Primary pseudotime method being benchmarked; run with k=10, p=0.9, SVD solver, and z.get_dtflow_times(root_list)) — https://github.com/statway/DTFLOW
- **Scanpy** (Provides the DPT pseudotime implementation (special case of RWR with p=1) used as the comparison baseline)
- **Monocle2** (R-based pseudotime method used as the second comparison baseline)
- **Python** (Implementation language for DTFLOW and the Kendall tau computation) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', dim); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- Computed Kendall tau values should be close to the paper's reported values: 0.862 (DTFLOW), 0.796 (Scanpy), 0.761 (Monocle2) on the ME dataset
- DTFLOW's tau should exceed both baselines (relative ranking DTFLOW > Scanpy > Monocle2 on ME)
- All three methods must be run on the same pre-processed expression matrix (same cells and genes) so tau values are comparable
- Each tau must be computed over the full set of 438 ME cells against the same stage ordering

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; on MFG the reference taus differ (0.761/0.702/0.569), so ME values are not universal targets
- DTFLOW/Scanpy/Monocle2 ordering accuracy depends on correct root cell selection via root_list
- Scanpy's DPT is only the p=1 special case of RWR, so differences in tau partly reflect the restart probability rather than the framework

## Evidence

- [results] ME Kendall taus: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [methods] RWR vs DPT: "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
- [results] restart probability: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] k neighbors: "We test different values of k and find that the results are better if k=10"
- [results] ME dataset size: "the ME single-cell dataset with 48 genes and 438 single cells [42]"
