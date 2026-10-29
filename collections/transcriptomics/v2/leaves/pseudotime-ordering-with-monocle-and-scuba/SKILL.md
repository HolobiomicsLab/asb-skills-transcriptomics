---
name: pseudotime-ordering-with-monocle-and-scuba
description: Decision support for choosing a trajectory-inference method when a scRNA-seq
  progenitor dataset is suspected to contain a branching event and single-path pseudotime
  methods (SCUBA, monocle) may miss it.
when_to_use_negative:
- The dataset is known to follow a single linear differentiation path with no branching,
  where monocle's single-path pseudotime suffices
- Development starts from multiple independent progenitor cell types — Mpath does
  not yet support this scenario
- Highly heterogeneous single-cell datasets in which specific subset information is
  unknown — Mpath depends on signature genes of FACS-sorted terminal subsets
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0085
tools:
- name: Mpath
  role: Reference neighborhood-based trajectory method used to confirm the preDC_c
    (cDC1) / preDC_d (cDC2) branch on identical input
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: SCUBA
  role: Baseline trajectory method run on the DC progenitor matrix to test whether
    it resolves the cDC1/cDC2 branching event
- name: R
  role: Implementation environment for monocle, diffusion map, PCA, MST construction
    and Mpath comparisons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:ab8e64b64eadaeb12e3a71e1c7ac4d92edd8b3fada246b023de8f960dab8971c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  - SCUBA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pseudotime-ordering-with-monocle-and-scuba

## Summary

Benchmark SCUBA, monocle, diffusion map, PCA and conventional MST trajectory inference on a single-cell progenitor dataset and compare their ability to detect branching against Mpath's neighborhood-based trajectories.

## When to use

Decision support for choosing a trajectory-inference method when a scRNA-seq progenitor dataset is suspected to contain a branching event and single-path pseudotime methods (SCUBA, monocle) may miss it.

## When NOT to use

- The dataset is known to follow a single linear differentiation path with no branching, where monocle's single-path pseudotime suffices
- Development starts from multiple independent progenitor cell types — Mpath does not yet support this scenario
- Highly heterogeneous single-cell datasets in which specific subset information is unknown — Mpath depends on signature genes of FACS-sorted terminal subsets

## Inputs

- QC-passed single-cell expression matrix of mouse DC progenitors (CDP to preDC stages)
- Differentially expressed gene (DEG) signature lists for mature cDC1 and cDC2 (Supplementary Data 1)
- Cell labels designating landmark subsets (preDC_a, preDC_b, preDC_c, preDC_d)

## Outputs

- trajectory_method_comparison.csv tabulating per-method branch detection and placement of preDC_c and preDC_d
- Per-method trajectory structures (SCUBA clusters, monocle pseudotime ordering, diffusion map/PCA embeddings, MST, Mpath trajectory)

## How to apply

Load the QC-passed DC progenitor expression matrix and the mature cDC1/cDC2 DEG signature lists (Supplementary Data 1) into R. Run SCUBA and monocle on the same matrix and record whether each detects a branching event separating cDC1-committed (preDC_c) and cDC2-committed (preDC_d) preDC subsets; monocle's single-path pseudotime is expected to collapse the branch. Embed cells with diffusion map and PCA and visually check for cDC1/cDC2 preDC branching. Construct a conventional minimum spanning tree on Euclidean distances and record the branch placement of preDC_c and preDC_d — in the benchmark MST placed the committed preDCs on the wrong branch. Run Mpath on the identical input as the reference method, using neighborhood-based cell state transitions (count of neighbouring cells) and signature genes from end-stage contrasts, then tabulate per-method outcomes (branch detected yes/no, preDC_c/preDC_d placement) into trajectory_method_comparison.csv.

## Related tools

- **Mpath** (Reference neighborhood-based trajectory method used to confirm the preDC_c (cDC1) / preDC_d (cDC2) branch on identical input) — https://github.com/JinmiaoChenLab/Mpath
- **SCUBA** (Baseline trajectory method run on the DC progenitor matrix to test whether it resolves the cDC1/cDC2 branching event)
- **R** (Implementation environment for monocle, diffusion map, PCA, MST construction and Mpath comparisons)

## Examples

```
Rscript -e 'library(Mpath); traj <- Mpath(expr=dc_progenitor_matrix, sig.genes=read.delim("SupplementaryData1_cDC1_cDC2_DEGs.txt"))'
```

## Evaluation signals

- Mpath, and only Mpath, faithfully recapitulates the branching of cDC1- and cDC2-committed preDCs on the benchmark dataset
- Conventional MST should be checked against the known failure mode: cDC1- and cDC2-committed preDCs (preDC_c, preDC_d) placed on the wrong branch
- monocle's single-path pseudotime ordering is expected to miss the cDC1/cDC2 preDC branch — deviation from this is a red flag
- trajectory_method_comparison.csv contains a row per method (SCUBA, monocle, diffusion map, PCA, MST, Mpath) with branch-detected yes/no and preDC placement fields populated
- Mpath trajectory spans macrophage/DC progenitors through common DC progenitor to pre-dendritic cells, matching the published cDC development order

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, so it cannot be applied without defined terminal states
- Mpath does not yet support development processes starting from multiple independent progenitor cell types
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown
- Benchmarked utility demonstrated only on two datasets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [results] Mpath outperformed SCUBA, monocle, diffusion map, PCA and MST: [evidence span withheld — anchor status: not_found]
- [results] MST misplacement: [evidence span withheld — anchor status: not_found]
- [abstract] Mpath algorithm principle: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [abstract] Branching event finding: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Mpath fate-prediction dependency: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [readme] README: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
