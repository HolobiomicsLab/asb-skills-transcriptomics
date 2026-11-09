---
name: cross-algorithm-lineage-comparison-visualization
description: Use when two trajectory-inference algorithms (e.g., CellRouter flow networks
  vs Monocle 2 DDRTree) must be compared on the same count matrix to test whether
  underrepresented or rare lineage bifurcations are resolved or collapsed.
when_to_use_negative:
- You need a trajectory between two chosen cell states, including transitions to or
  from rare cell types — Monocle 2's DDRTree is explicitly not designed for this;
  use CellRouter alone.
- The input is not a single-cell count matrix with cell annotations but a pseudotime
  vector, feature table, or already-completed trajectory object — a re-run comparison
  cannot be performed.
- A priori knowledge of the number of branches is required by the downstream analysis
  — CellRouter requires no a priori knowledge of trajectory structure, so forced branch
  counts defeat the comparison.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: CellRouter
  role: Reconstructs subpopulation-aware multi-state trajectories via flow networks
    from progenitor subpopulations to defined subpopulations (e.g., 19 and 5)
  repo: https://github.com/edroaldo/cellrouter
- name: Monocle 2
  role: Runs the default DDRTree branching trajectory on the same count matrix to
    obtain a second lineage/branch assignment for comparison
- name: R
  role: Analysis environment; both pipelines and the comparison table/figure generation
    are implemented in R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:5ddffc537da36efc9d6521ee85866a87a333cd79a278421fd41d56b762515e21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Monocle 2
  - R
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-algorithm-lineage-comparison-visualization

## Summary

A side-by-side lineage resolution comparison between CellRouter's subpopulation-aware flow-network trajectories and Monocle 2's default DDRTree branching, applied to the same single-cell RNA-seq matrix, with a comparison table and two-panel figure. It exposes when one algorithm resolves an underrepresented bifurcation (e.g., megakaryocyte/erythrocyte in Olsson GSE70245) that the other collapses onto a single branch.

## When to use

Use when two trajectory-inference algorithms (e.g., CellRouter flow networks vs Monocle 2 DDRTree) must be compared on the same count matrix to test whether underrepresented or rare lineage bifurcations are resolved or collapsed.

## When NOT to use

- You need a trajectory between two chosen cell states, including transitions to or from rare cell types — Monocle 2's DDRTree is explicitly not designed for this; use CellRouter alone.
- The input is not a single-cell count matrix with cell annotations but a pseudotime vector, feature table, or already-completed trajectory object — a re-run comparison cannot be performed.
- A priori knowledge of the number of branches is required by the downstream analysis — CellRouter requires no a priori knowledge of trajectory structure, so forced branch counts defeat the comparison.

## Inputs

- Single-cell RNA-seq raw count matrix with cell annotations (e.g., GSE70245 Olsson wild-type myeloid progenitor data)
- CellRouter subpopulation assignments (e.g., subpopulation 19 = megakaryocyte, subpopulation 5 = erythrocyte) from community detection
- Monocle 2 branch/lineage assignment per cell from the default DDRTree trajectory

## Outputs

- lineage_comparison.csv — per-subpopulation table of trajectory/branch assignments from each tool
- lineage_comparison.png — two-panel side-by-side figure of CellRouter vs Monocle 2 lineage assignments
- Validation conclusion stating whether the algorithms resolve distinct trajectories for the target subpopulations

## How to apply

Load the same wild-type scRNA-seq count matrix and cell annotations (e.g., GSE70245 Olsson dataset) once and run both algorithms on identical input. For CellRouter, learn the data structure and identify subpopulations with community detection (igraph/mclust), confirming specific subpopulations such as subpopulation 19 (megakaryocyte) and subpopulation 5 (erythrocyte), then reconstruct high-resolution multi-state trajectories from progenitor subpopulations to each target subpopulation using CellRouter's flow network algorithm. For Monocle 2, run the default DDRTree branching trajectory on the same matrix and record its lineage/branch assignment per cell. Per subpopulation, tabulate which trajectory/branch each tool assigns (lineage_comparison.csv) and render a two-panel figure (lineage_comparison.png). The key decision point is whether the two algorithms assign distinct or identical trajectories to the target subpopulations — the rationale is that Monocle 2 'is not designed to reconstruct trajectories between any two chosen cell states', so rare fates may collapse onto one branch, while CellRouter requires no a priori knowledge of trajectory structure.

## Related tools

- **CellRouter** (Reconstructs subpopulation-aware multi-state trajectories via flow networks from progenitor subpopulations to defined subpopulations (e.g., 19 and 5)) — https://github.com/edroaldo/cellrouter
- **Monocle 2** (Runs the default DDRTree branching trajectory on the same count matrix to obtain a second lineage/branch assignment for comparison)
- **R** (Analysis environment; both pipelines and the comparison table/figure generation are implemented in R)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'; sc <- CellRouter(dat=data.raw, annot=annotations); sc <- findSubpop(sc); sc <- findPaths(sc, c('S19','S5'))
```

## Evaluation signals

- CellRouter resolves distinct trajectories from progenitors to subpopulation 19 (megakaryocyte) and subpopulation 5 (erythrocyte) rather than a shared path.
- Monocle 2 assigns both target lineages to the same branch, which is the expected contrast on the Olsson bifurcation.
- lineage_comparison.csv contains a per-subpopulation row for every subpopulation analyzed, with both tools' trajectory/branch labels present.
- lineage_comparison.png contains exactly two labeled panels (one per algorithm) covering the same cell set.
- Subpopulation identities (19 = megakaryocyte, 5 = erythrocyte) from community detection are confirmed before trajectory reconstruction and submitted for expert_review of lineage identity.

## Limitations

- Monocle 2 addresses many trajectory challenges but is not designed to reconstruct trajectories between any two chosen cell states, which might include transitions from or to rare cell types.
- Monocle 1, Waterfall, Wishbone, and DPT are limited to linear trajectories, single branch points, or requiring a priori knowledge of the number of branches, so the comparison partner choice matters.
- CellRouter was tested in Unix-based systems only and is not tested on Windows; the CellRouter class must be sourced with the correct libdir path to the Java libraries.
- Current trajectory algorithms, including Monocle 2, are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states.

## Evidence

- [other] CellRouter resolved distinct HSPC differentiation trajectories to megakaryocytes or erythrocytes (subpopulations 19 and 5), whereas Monocle 2 assigned both lineages to the same branch.: [evidence span withheld — anchor status: not_found]
- [intro] Monocle 2 is not designed to reconstruct trajectories between any two chosen cell states: "Monocle 2 addresses many of these challenges but is not designed to reconstruct trajectories between any two chosen cell states"
- [methods] Subpopulations are identified by a combination of learning the structure of the data and community detection algorithms: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [intro] CellRouter implements a flow network algorithm to flexibly reconstruct multi-state transition trajectories: "CellRouter relaxes the requirement of identifying branching points during cell-fate transitions and implements a flow network algorithm to flexibly reconstruct multi-state transition trajectories"
- [intro] CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches: "CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches"
- [readme] R packages required: reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd: "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
