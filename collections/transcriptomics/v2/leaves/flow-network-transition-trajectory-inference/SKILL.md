---
name: flow-network-transition-trajectory-inference
description: Use when single-cell RNA-seq count data must be resolved into distinct
  lineage trajectories toward rare or underrepresented cell fates, especially when
  a branching method like Monocle 2 collapses two lineages onto one branch.
when_to_use_negative:
- 'Monocle 2/DDRTree alone suffices: only use this skill when trajectories between
  two chosen cell states (e.g., toward rare megakaryocyte/erythrocyte fates) are needed,
  since Monocle 2 ''is not designed to reconstruct trajectories between any two chosen
  cell states'''
- CellRouter was tested only on Unix-based systems/Mac and not on Windows, so avoid
  on Windows environments
- Analysis goal is differential expression or clustering only, with no cell-state
  transition or lineage ordering question
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CellRouter
  role: 'Primary platform: flow-network reconstruction of multi-state trajectories
    and subpopulation identification via community detection'
  repo: https://github.com/edroaldo/cellrouter
- name: Monocle 2
  role: Benchmark comparator run with default DDRTree branching to assess whether
    lineages collapse onto a single branch
- name: R
  role: Implementation environment; CellRouter class sourced in R with required packages
    (reshape, pheatmap, tsne, igraph, ggplot2, mclust, Rtsne, cccd)
  repo: https://github.com/edroaldo/cellrouter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:7686349c0679054e057812aed63b5b3c753fc860f4214cb7b03045cf0d363cc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Monocle 2
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# flow-network-transition-trajectory-inference

## Summary

Reconstructs high-resolution multi-state cell-state transition trajectories between subpopulations (e.g., HSPC progenitors to megakaryocyte vs. erythrocyte fates) using CellRouter's flow network algorithm, which requires no a priori knowledge of the number of branches — unlike Monocle 2's DDRTree branching.

## When to use

Use when single-cell RNA-seq count data must be resolved into distinct lineage trajectories toward rare or underrepresented cell fates, especially when a branching method like Monocle 2 collapses two lineages onto one branch.

## When NOT to use

- Monocle 2/DDRTree alone suffices: only use this skill when trajectories between two chosen cell states (e.g., toward rare megakaryocyte/erythrocyte fates) are needed, since Monocle 2 'is not designed to reconstruct trajectories between any two chosen cell states'
- CellRouter was tested only on Unix-based systems/Mac and not on Windows, so avoid on Windows environments
- Analysis goal is differential expression or clustering only, with no cell-state transition or lineage ordering question

## Inputs

- Raw count or UMI single-cell RNA-seq matrix (e.g., GEO GSE70245 wild-type mouse myeloid progenitor data)
- Cell annotation/metadata table used for user-defined clusters or sorted-population labels
- User-specified source and target subpopulation IDs (e.g., progenitor subpopulations to subpopulations 19 and 5)

## Outputs

- Subpopulation assignments from community detection (e.g., subpopulation 19 megakaryocyte, subpopulation 5 erythrocyte)
- Multi-state transition trajectories from progenitor subpopulations to each target subpopulation
- Side-by-side lineage comparison table (lineage_comparison.csv) per subpopulation per tool
- Two-panel trajectory comparison figure (lineage_comparison.png) for expert review

## How to apply

Load the raw count matrix and cell annotations (e.g., GSE70245 wild-type mouse scRNA-seq) into R, source the CellRouter class, and set the 'libdir' variable to the folder containing CellRouter's Java libraries. First learn the structure of the data and identify subpopulations via community detection (igraph) and clustering (mclust), confirming subpopulation identity (e.g., subpopulation 19 = megakaryocyte, subpopulation 5 = erythrocyte). Then reconstruct high-resolution multi-state trajectories with CellRouter's flow network algorithm from chosen progenitor subpopulations to the target subpopulations; because CellRouter is transition-centered and subpopulation-aware, trajectories between any two chosen states (including rare types) can be requested without specifying branch count. For benchmarking, run Monocle 2 on the same input matrix with its default DDRTree branching and record which trajectory/branch each tool assigns per subpopulation. CellRouter's key rationale: Monocle 2 is not designed to reconstruct trajectories between any two chosen cell states, so it can place two distinct lineages on the same branch while CellRouter separates them.

## Related tools

- **CellRouter** (Primary platform: flow-network reconstruction of multi-state trajectories and subpopulation identification via community detection) — https://github.com/edroaldo/cellrouter
- **Monocle 2** (Benchmark comparator run with default DDRTree branching to assess whether lineages collapse onto a single branch)
- **R** (Implementation environment; CellRouter class sourced in R with required packages (reshape, pheatmap, tsne, igraph, ggplot2, mclust, Rtsne, cccd)) — https://github.com/edroaldo/cellrouter

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- CellRouter resolves distinct trajectories from progenitors to subpopulation 19 (megakaryocyte) and subpopulation 5 (erythrocyte), whereas Monocle 2 assigns both lineages to the same branch
- Subpopulation detection via community detection reproduces the expected subpopulations from the count matrix before trajectory inference
- Trajectory reconstruction succeeds without pre-specifying the number of cell fates or branches (no a priori trajectory structure)
- Multi-state trajectories show divergence of single-cell transcriptomes along each trajectory progression
- Labeled lineage_comparison.png panels pass expert_review of lineage identity

## Limitations

- Existing trajectory algorithms (Waterfall, Monocle 1 linear; Wishbone single branch point; DPT requiring a priori branch number) are less robust toward intermediate or transitory cell states — the reason this skill exists, but also a caution about relying on those alternatives
- Monocle 2 comparison baseline cannot reconstruct transitions between arbitrary chosen cell states, limiting symmetric benchmarking of rare-fate trajectories
- CellRouter was tested only in Unix-based systems/Mac; not tested on Windows
- Functional characterization of trajectory-identified genes for generating engraftable HSC-like cells is beyond the scope of the manuscript

## Evidence

- [intro] flow network algorithm: "CellRouter relaxes the requirement of identifying branching points during cell-fate transitions and implements a flow network algorithm to flexibly reconstruct multi-state transition trajectories"
- [intro] no a priori structure: "CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches"
- [intro] Monocle 2 limitation: "Monocle 2 addresses many of these challenges but is not designed to reconstruct trajectories between any two chosen cell states"
- [methods] subpopulation identification: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [methods] high-resolution trajectories: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [readme] setup: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
