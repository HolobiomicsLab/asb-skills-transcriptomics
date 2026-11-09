---
name: transition-specific-gene-expression-scoring
description: Use when single-cell RNA-seq data contain annotated subpopulations and
  you need transition-specific ranked lists of overexpression or knockdown candidates
  for a defined start-to-target cell-state transition, such as reprogramming B-cells
  or monocytes/DCs to HSPCs.
when_to_use_negative:
- When you only need a single linear pseudotemporal ordering without transition-specific
  regulator ranking — simpler tools (Waterfall, Monocle 1) or diffusion pseudotime
  may suffice
- When the input is not single-cell data with identifiable subpopulations (e.g., bulk
  RNA-seq or an already-aggregated feature table)
- When the goal is functional validation of engraftable HSC-like cells — the article
  explicitly defers thorough characterization of each identified gene's potential
  to future work
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellRouter
  role: 'Core platform: reconstructs multi-state flow-network trajectories between
    chosen subpopulations and scores transition-specific gene expression and regulators
    using the integrated GRN'
  repo: https://github.com/edroaldo/cellrouter
- name: igraph
  role: Community detection used with mclust to identify subpopulations that define
    trajectory endpoints
- name: mclust
  role: Model-based clustering supporting subpopulation identification prior to trajectory
    reconstruction
- name: Rtsne
  role: Dimensionality reduction used in subpopulation structure learning
- name: R
  role: Implementation environment; CellRouter class is sourced in R with libdir set
    to the Java libraries folder
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:a940c9e6e1bb78823027c5fc49f86245e72d761cfff43858e60ae2cccce903eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - Rtsne
  - cccd
  - ggplot2
  - igraph
  - mclust
  - pheatmap
  - reshape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transition-specific-gene-expression-scoring

## Summary

Score genes and transcriptional regulators along CellRouter-reconstructed trajectories between defined subpopulations (e.g., B-cells subpop 10 and mono/DC subpop 17 toward HSPCs subpop 12 in GSE75478) to rank overexpression and knockdown candidates for cellular reprogramming. Rationale: transition-specific scoring integrates multi-state flow-network trajectories with gene regulatory networks to capture regulators driving or mediating specific cell-fate transitions.

## When to use

Use when single-cell RNA-seq data contain annotated subpopulations and you need transition-specific ranked lists of overexpression or knockdown candidates for a defined start-to-target cell-state transition, such as reprogramming B-cells or monocytes/DCs to HSPCs.

## When NOT to use

- When you only need a single linear pseudotemporal ordering without transition-specific regulator ranking — simpler tools (Waterfall, Monocle 1) or diffusion pseudotime may suffice
- When the input is not single-cell data with identifiable subpopulations (e.g., bulk RNA-seq or an already-aggregated feature table)
- When the goal is functional validation of engraftable HSC-like cells — the article explicitly defers thorough characterization of each identified gene's potential to future work

## Inputs

- Single-cell RNA-seq expression matrix (e.g., GSE75478)
- Subpopulation/cluster annotations for cells (e.g., subpopulation 10 = B-cells, 17 = mono/DCs, 12 = HSPCs)
- CellRouter class source file and Java libraries directory (libdir)
- List of published reprogramming factors used as a recovery benchmark (67 factors)

## Outputs

- reprogramming_candidates.csv containing ranked overexpression candidate lists and ranked knockdown candidate lists per lineage transition
- Transition-specific gene expression scores along each reconstructed trajectory
- Predicted knockdown regulators (DNMT1, H3F3A, JARID2, KMT2D for B-cell; DNMT1, H2AFZ, TRIM28, KMT2D, H2AFV for mono/DC to HSPC)

## How to apply

First, embed cells with STEMNET and identify subpopulations using CellRouter's combination of learning the structure of the data and community detection (igraph/mclust), then reconstruct the gene regulatory network from the single cells to define putative gene regulatory relationships. Run CellRouter flow-network trajectory reconstruction from the source subpopulation to the target subpopulation (subpopulation 10 B-cells or 17 monocyte/DCs to subpopulation 12 HSPCs in the GSE75478 example). Score genes along each transition trajectory to produce ranked overexpression candidates (targeting recovery of the published reprogramming factors) and ranked knockdown candidates. Validate by checking that 11 of the 67 published reprogramming factors are recovered among overexpression candidates and that the expected knockdown regulators appear: DNMT1, H3F3A, JARID2, KMT2D for the B-cell-to-HSPC transition and DNMT1, H2AFZ, TRIM28, KMT2D, H2AFV for the mono/DC-to-HSPC transition.

## Related tools

- **CellRouter** (Core platform: reconstructs multi-state flow-network trajectories between chosen subpopulations and scores transition-specific gene expression and regulators using the integrated GRN) — https://github.com/edroaldo/cellrouter
- **igraph** (Community detection used with mclust to identify subpopulations that define trajectory endpoints)
- **mclust** (Model-based clustering supporting subpopulation identification prior to trajectory reconstruction)
- **Rtsne** (Dimensionality reduction used in subpopulation structure learning)
- **R** (Implementation environment; CellRouter class is sourced in R with libdir set to the Java libraries folder)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- reprogramming_candidates.csv exists and contains both overexpression and knockdown ranked lists
- 11 of the 67 published reprogramming factors are recovered among the overexpression candidates
- B-cell knockdown list contains DNMT1, H3F3A, JARID2, and KMT2D
- Mono/DC knockdown list contains DNMT1, H2AFZ, TRIM28, KMT2D, and H2AFV
- Trajectories were reconstructed from the correct endpoints: subpopulation 10 (B-cells) and 17 (mono/DCs) to subpopulation 12 (HSPCs)

## Limitations

- Functional characterization of genes identified for generating engraftable HSC-like cells is deferred to future work and beyond the manuscript's scope
- CellRouter was tested only on Unix-based systems/Mac and not tested on Windows
- No dataset accession numbers, software versions, or parameter settings are provided within the discussion section
- Application to single-cell chromatin accessibility, DNA methylation, and mass cytometry data is only envisioned as future work, not demonstrated

## Evidence

- [abstract] CellRouter was applied to cellular reprogramming of monocytes and B-cells to HSPCs: "re-specification of cell identity by cellular reprogramming of monocytes and B-cells to HSPCs"
- [discussion] CellRouter addresses many of these challenges by integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression: "CellRouter addresses many of these challenges by integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression"
- [methods] source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir': "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [readme] CellRouter identifies complex cell-state transition trajectories by using flow networks to explore the subpopulation structure of multi-dimensional, single-cell omics data: "identifies complex cell-state transition trajectories by using flow networks to explore the subpopulation structure of multi-dimensional, single-cell omics data"
