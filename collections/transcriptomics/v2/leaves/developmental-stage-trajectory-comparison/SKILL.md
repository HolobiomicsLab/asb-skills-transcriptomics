---
name: developmental-stage-trajectory-comparison
description: Use when a single-cell dataset carries known developmental stage/time
  labels and the goal is to validate that a low-dimensional embedding recovers them
  chronologically or to check which method resolves an intermediate state.
when_to_use_negative:
- The dataset lacks per-cell stage or time labels, so no chronological-order recovery
  can be validated
- The goal is branch/sub-branch detection (RSKG) rather than stage-recovering visualization
- The data is not single-cell trajectory data but, e.g., bulk or already-embedded
  features where diffusion-propagation manifold learning is inapplicable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: 'Python package implementing the BKFD embedding: transition matrix, RWR diffusion,
    SVD dimension reduction, and stage-colored plotting'
  repo: https://github.com/statway/DTFLOW
- name: Monocle2
  role: Baseline trajectory method whose embedding of the same cells is compared against
    the BKFD embedding to check ICM resolution
- name: Python
  role: Implementation platform for DTFLOW; used with pandas for data loading and
    matplotlib for plotting
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:c15173dcb6139e9edced6da97bc5ca9ad8847a75f3df9c8064879a9b04510a8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# developmental-stage-trajectory-comparison

## Summary

Reproduce the BKFD (Bhattacharyya kernel feature decomposition) 2-D/3-D embedding of a labelled single-cell developmental dataset to verify that all known stages are recovered in correct chronological order, and compare against alternative trajectory methods (Scanpy, Monocle2) to detect intermediate states. Demonstrated on the mouse embryo (ME) RT-PCR dataset with seven stages (1-cell to 64-cell) and the ICM intermediate state.

## When to use

Use when a single-cell dataset carries known developmental stage/time labels and the goal is to validate that a low-dimensional embedding recovers them chronologically or to check which method resolves an intermediate state.

## When NOT to use

- The dataset lacks per-cell stage or time labels, so no chronological-order recovery can be validated
- The goal is branch/sub-branch detection (RSKG) rather than stage-recovering visualization
- The data is not single-cell trajectory data but, e.g., bulk or already-embedded features where diffusion-propagation manifold learning is inapplicable

## Inputs

- Gene expression matrix X (N cells x D genes) with per-cell developmental stage labels, e.g., the ME high-throughput RT-PCR dataset (438 cells, 48 genes, stages 1-cell through 64-cell)
- Stage/time label vector (time_stages) for each cell
- Parameters: number of nearest neighbors k=10, restart probability p=0.9, embedding dimension d=2 (or 3)

## Outputs

- Low-dimensional BKFD embedding Y (N cells x d) computed via SVD of the log Bhattacharyya kernel
- 2-D/3-D scatter plot of the embedding colored by developmental stages (z.plotY(color='stages'))
- Chronological ordering quality metric (Kendall rank correlation between embedding-ordered and true stage order)
- Comparison panel showing whether the ICM intermediate stage is resolved by each method

## How to apply

Load the labelled expression matrix (e.g., the ME dataset: 438 cells x 48 genes from high-throughput RT-PCR) into a DTFLOW object along with the stage labels. Build the k-nearest-neighbor Markov transition matrix with k=10 (the best value found in the paper), then apply random walk with restart at p=0.9 to obtain a diffusion distribution vector per cell. Form the Bhattacharyya kernel from inner products of the distribution vectors, take the log, and run SVD keeping d=2 (or 3) singular values to get the BKFD embedding Y. Plot Y colored by stage labels with matplotlib and, for the ME data, check that the inner cell mass (ICM) intermediate stage is visibly separated — the paper's finding is that among DTFLOW, Scanpy, and Monocle2, only DTFLOW resolves the ICM state.

## Related tools

- **DTFLOW** (Python package implementing the BKFD embedding: transition matrix, RWR diffusion, SVD dimension reduction, and stage-colored plotting) — https://github.com/statway/DTFLOW
- **Monocle2** (Baseline trajectory method whose embedding of the same cells is compared against the BKFD embedding to check ICM resolution)
- **Python** (Implementation platform for DTFLOW; used with pandas for data loading and matplotlib for plotting) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', dim=2); z.plotY(size=(10,10), color='stages')
```

## Evaluation signals

- Kendall rank correlation between the embedding-based cell ordering and true stage order should reach ~0.862 for DTFLOW on the ME dataset (vs 0.761 for Monocle2 and 0.796 for Scanpy)
- Cells in the 2-D plot must appear in the correct chronological order across all seven ME stages (1-cell zygote to 64-cell blastocyst)
- Only the DTFLOW/BKFD embedding should visibly resolve the ICM intermediate stage, per the paper's finding
- The SVD-based reduction must retain d=2 singular values of the log Bhattacharyya kernel matrix logG
- Reuse k=10 and p=0.9; if results degrade, check that these defaults were not changed

## Limitations

- Parameter values (Gaussian kernel, number of neighbors k, restart probability) may vary from dataset to dataset and are not standardized
- BKFD uses the same restart probability for all nodes, which may limit the effectiveness of the random walk
- Expressing each cell by a distribution vector in a better way remains an open challenge

## Evidence

- [results] Kendall coefficients ME: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [results] ME dataset scale: "the ME single-cell dataset with 48 genes and 438 single cells [42]"
- [results] restart probability: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] neighbor count: "We test different values of k and find that the results are better if k=10"
- [readme] README SVD step: "dimension reduction with SVD, and plot the result of dimension reduction"
- [readme] transition matrix: "M= z.get_transition_matrix(k)"
