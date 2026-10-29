---
name: lineage-cluster-annotation-mapping
description: Use when a pseudotime trajectory (e.g., DTFLOW on the 2730-cell MMP MARS-seq
  dataset with 3451 informative genes) has been computed and detected sub-branches
  must be annotated against known progenitor clusters and cross-checked against Scanpy/Monocle2
  branch calls.
when_to_use_negative:
- The dataset has no independently known cluster or lineage labels to map branches
  onto, making the annotation step unverifiable
- The goal is pseudotime ordering itself rather than branch detection and lineage
  annotation (use DTFLOW's pseudotime steps alone)
- Input data is raw counts that have not been pre-processed — Scanpy pre-processing
  (e.g., removal of corrupted gene names) must be done first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: DTFLOW
  role: Performs RSKG branch detection by reverse-searching the kNN graph in decreasing
    pseudotime order (z.get_branches routine) to produce the sub-branch labels being
    annotated
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Provides pre-processing (removal of the 10 corrupted gene names) and an independent
    branch/root-cell assignment used as a comparison baseline
- name: Monocle2
  role: Supplies a second independent branch-assignment baseline for the comparison
    against RSKG results
- name: Python
  role: Implementation language; DTFLOW is distributed as Python source code
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:9692a4b860984728746469d20dde64bd47625180c437929e4511a2eda75be614
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lineage-cluster-annotation-mapping

## Summary

Maps pseudotime branch labels produced by DTFLOW's RSKG branch detection onto known lineage clusters (e.g., the 19 erythroid/CMP/DC/GMP/lymphoid progenitor clusters of the MMP MARS-seq dataset) and compares them with branch assignments from Scanpy and Monocle2 to validate that the inferred branching structure is developmentally reasonable.

## When to use

Use when a pseudotime trajectory (e.g., DTFLOW on the 2730-cell MMP MARS-seq dataset with 3451 informative genes) has been computed and detected sub-branches must be annotated against known progenitor clusters and cross-checked against Scanpy/Monocle2 branch calls.

## When NOT to use

- The dataset has no independently known cluster or lineage labels to map branches onto, making the annotation step unverifiable
- The goal is pseudotime ordering itself rather than branch detection and lineage annotation (use DTFLOW's pseudotime steps alone)
- Input data is raw counts that have not been pre-processed — Scanpy pre-processing (e.g., removal of corrupted gene names) must be done first

## Inputs

- Gene expression matrix from the MMP MARS-seq dataset (2730 cells × 3451 informative genes, after removal of 10 genes with corrupted names via Scanpy pre-processing)
- Per-cell pseudotime values from DTFLOW (z.get_dtflow_times)
- RSKG branch assignments from DTFLOW (z.get_branches(Num))
- Scanpy branch assignments and root-cell identification
- Monocle2 branch assignments
- Known lineage cluster labels (19 MMP progenitor clusters: erythroid 1-6, CMP 7-10, DC 11, GMP 12-18, lymphoid 19)

## Outputs

- Per-cell branch labels mapped to lineage clusters
- Method-comparison summary (RSKG vs. Scanpy vs. Monocle2) written to branch_comparison.csv

## How to apply

After computing pseudotime via diffusion (Gaussian-kernel Markov transition matrix with k=10 nearest neighbors, random walk with restart at p=0.9), run DTFLOW's RSKG reverse search on the kNN graph in decreasing pseudotime order, merging or splitting candidate groups using a minimum sub-branch size threshold Num. Map each detected RSKG sub-branch to the known biological clusters — for MMP, the 19 clusters spanning erythroid (1-6), CMP (7-10), DC (11), GMP (12-18), and lymphoid (19) — and check that the branch structure reproduces the erythroid/CMP/GMP/DC split shown in the paper's MMP figures. Compare RSKG branch assignments against Scanpy and Monocle2 branch calls; judge validity by whether the root cell lands in a developmentally sensible position (e.g., Scanpy identified four sub-branches on MMP, matching DTFLOW, but its root cell was not in the initial group, which is developmentally unreasonable). Write per-cell branch labels and the method-comparison summary to an output file such as branch_comparison.csv.

## Related tools

- **DTFLOW** (Performs RSKG branch detection by reverse-searching the kNN graph in decreasing pseudotime order (z.get_branches routine) to produce the sub-branch labels being annotated) — https://github.com/statway/DTFLOW
- **Scanpy** (Provides pre-processing (removal of the 10 corrupted gene names) and an independent branch/root-cell assignment used as a comparison baseline)
- **Monocle2** (Supplies a second independent branch-assignment baseline for the comparison against RSKG results)
- **Python** (Implementation language; DTFLOW is distributed as Python source code) — https://github.com/statway/DTFLOW

## Examples

```
z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list); branch = z.get_branches(Num); z.plotY(size=(10,10), color='branches')
```

## Evaluation signals

- RSKG branches reproduce the lineage structure (erythroid/CMP/GMP/DC split) shown in the paper's MMP figures
- Branch calls align with the Monocle2 and Scanpy assignments reported for the MMP dataset
- Number of detected sub-branches matches expectation (Scanpy and DTFLOW both identified four sub-branches on MMP)
- The identified root cell lies in the initial/earliest developmental group (Scanpy's misplaced root was flagged as developmentally unreasonable)
- branch_comparison.csv contains per-cell branch labels for all cells and a coherent method-comparison summary

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- The minimum sub-branch size Num (minimum number of cells required for forming one sub-branch) must be chosen by the user and affects merging/splitting of candidate groups
- Alternative methods such as Scanpy may detect the correct number of branches but misplace the root cell, so branch-count agreement alone is not sufficient validation

## Evidence

- [abstract] a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation: "a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation"
- [methods] 10 genes with corrupted names removed: "10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy."
- [methods] Rskg For Branch Detection filter step: "## Method > Rskg For Branch Detection"
- [readme] branch detection API: "branch = z.get_branches(Num) # 1-11 12-112"
- [readme] Num is minimum sub-branch size: "here Num is the minimum number of cells required for forming one sub-branch."
