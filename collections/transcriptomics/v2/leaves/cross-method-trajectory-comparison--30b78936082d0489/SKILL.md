---
name: cross-method-trajectory-comparison
description: Use when a single-cell pseudotime/branching result from one method (e.g.,
  DTFLOW RSKG) needs to be validated or contextualized against branch assignments
  from alternative tools such as Scanpy or Monocle2.
when_to_use_negative:
- The dataset has only a single developmental path with no branching events, so cross-method
  branch comparison has no informative signal
- No trusted reference branching structure or independent method output exists, making
  agreement checks unfalsifiable
- Input is raw sequencing data rather than a pre-processed gene expression matrix
  with pseudotime and branch labels available
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DTFLOW
  role: 'Primary method: computes Markov transition matrix, RWR diffusion matrix,
    pseudotime, and RSKG branches (get_branches) for the reference branching result'
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: 'Comparison method: provides pre-processing (corrupted-gene removal) and an
    independent branch/root-cell assignment to contrast with RSKG'
- name: Monocle2
  role: 'Comparison method: supplies an additional independent branch assignment against
    which RSKG branches are aligned'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:61e5bdd10a402800e490dc82d642b33a7107a57580493a0eff57ff6a59461a5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-method-trajectory-comparison

## Summary

Reproduces and cross-validates multi-branch trajectory detection by comparing DTFLOW's RSKG branch assignments against branches from Scanpy and Monocle2 on the same single-cell dataset (e.g., the MMP MARS-seq myeloid progenitor data with 2730 cells and 3451 informative genes), exposing method-specific artifacts such as developmentally unreasonable root-cell placement.

## When to use

Use when a single-cell pseudotime/branching result from one method (e.g., DTFLOW RSKG) needs to be validated or contextualized against branch assignments from alternative tools such as Scanpy or Monocle2.

## When NOT to use

- The dataset has only a single developmental path with no branching events, so cross-method branch comparison has no informative signal
- No trusted reference branching structure or independent method output exists, making agreement checks unfalsifiable
- Input is raw sequencing data rather than a pre-processed gene expression matrix with pseudotime and branch labels available

## Inputs

- Pre-processed gene expression matrix (N cells x D genes), e.g., MMP MARS-seq matrix with 2730 cells and 3451 informative genes after removal of 10 corrupted-name genes
- Pseudotime ordering per cell from the primary method (DTFLOW diffusion/RWR pseudotime)
- Branch assignments from comparison methods (Scanpy and Monocle2) on the same cells
- Known biological cluster labels for validation (e.g., 19 MMP progenitor clusters)

## Outputs

- Per-cell branch labels from each method (e.g., branch_comparison.csv)
- Method-comparison summary (branch counts, root-cell placement, lineage correspondence)

## How to apply

Load the target dataset (e.g., MMP MARS-seq: 2730 cells, 3451 informative genes, with the 10 genes with corrupted names removed via Scanpy pre-processing) and run the primary method's pipeline: build the k-nearest-neighbor graph (k=10, the paper's optimal value) with the Gaussian-kernel Markov transition matrix, compute the diffusion matrix by random walk with restart (restart probability p=0.9), derive pseudotime, then run RSKG branch detection (z.get_branches(Num)) by reverse-searching the kNN graph in decreasing pseudotime order with a minimum sub-branch size Num. In parallel, obtain branch assignments from comparison methods (Scanpy and Monocle2) on the same pre-processed matrix. Align the per-cell branch labels across methods and compare them against known biology — for MMP, the 19 known progenitor clusters (erythroid 1-6, CMP 7-10, DC 11, GMP 12-18, lymphoid 19) and the paper's erythroid/CMP/GMP/DC split. Write per-cell labels and the comparison summary to a file such as branch_comparison.csv. Judge agreement not just by branch count but by whether root-cell placement and lineage structure are developmentally reasonable; e.g., Scanpy found the same four sub-branches as DTFLOW on MMP, but its root cell fell outside the initial group.

## Related tools

- **DTFLOW** (Primary method: computes Markov transition matrix, RWR diffusion matrix, pseudotime, and RSKG branches (get_branches) for the reference branching result) — https://github.com/statway/DTFLOW
- **Scanpy** (Comparison method: provides pre-processing (corrupted-gene removal) and an independent branch/root-cell assignment to contrast with RSKG)
- **Monocle2** (Comparison method: supplies an additional independent branch assignment against which RSKG branches are aligned)

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list); branch = z.get_branches(Num)
```

## Evaluation signals

- RSKG on MMP should reproduce the four sub-branches reported for DTFLOW's branching result
- Detected branches should align with the known lineage structure (erythroid/CMP/GMP/DC split) across the 19 known progenitor clusters
- Root cell of each method must sit in a developmentally reasonable position (e.g., the initial group); a root outside the initial group, as Scanpy produced on MMP, is flagged as an artifact
- Per-cell branch labels and comparison summary are complete for all 2730 cells in branch_comparison.csv
- Branch correspondence to Monocle2 and Scanpy assignments matches those reported in the paper's MMP figures

## Limitations

- Parameter values (Gaussian kernel, number of nearest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- DTFLOW's software versions and environment are not reported in the paper (only hardware: Lenovo ThinkPad P53, 2.6 GHz CPU), so exact reproduction may need trial and error
- Agreement across methods is necessary but not sufficient: identical branch counts can coexist with a developmentally unreasonable root-cell placement, so comparisons must include root position, not just branch topology

## Evidence

- [results] On the MMP dataset, Scanpy was also able to identify four sub-branches (matching DTFLOW's branching result), but the root cell identified by Scanpy was not located in the initial group, which is developmentally unreasonable.: [evidence span withheld — anchor status: not_found]
- [methods] The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes: "The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes"
- [methods] 10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy.: "10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy."
- [abstract] a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation: "a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation"
- [readme] here Num is the minimum number of cells required for forming one sub-branch.: "here Num is the minimum number of cells required for forming one sub-branch."
