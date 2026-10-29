---
name: lineage-path-pseudotime-reordering
description: Use when single-cell data have been mapped onto a multi-branching trajectory
  and you need to order cells along a specific branch and identify temporally phased
  waves of gene regulation along that branch.
when_to_use_negative:
- Trajectory structure has not yet been established — Mpath derives multi-branching
  trajectories via neighborhood-based cell state transitions before reordering, so
  this skill assumes a path already exists
- Developmental process starts from multiple independent progenitor cell types or
  subset information is unknown — the article notes Mpath does not yet support these
  settings
- Data lack signature genes of differentiated (end-stage) cell types, since Mpath
  fate prediction depends on end-stage contrasts
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: R implementation used to fit GAMs of gene expression as a smooth function
    of pseudo-temporal position and cluster fitted dynamics into temporal wave clusters
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Environment in which the Mpath package is executed for GAM fitting and clustering
- name: Ingenuity Pathway Analysis (IPA)
  role: Used to annotate wave clusters with pathway terms to identify mitosis and
    DNA-synthesis gene sets and MHC class II antigen processing enrichment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:ebef9badcfdc28733342f0fa71c7dd8df7e9b69228a7e6001dd38743006b6a73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Ingenuity Pathway Analysis (IPA)
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lineage-path-pseudotime-reordering

## Summary

Reorders individual cells along a lineage-specific developmental path (e.g., early CDP to late cDC2- or cDC1-committed preDC) and fits gene expression as a smooth function of pseudo-temporal position using GAMs (Mpath R implementation) to uncover sequential waves of gene regulation and temporal coupling between cell cycle and differentiation.

## When to use

Use when single-cell data have been mapped onto a multi-branching trajectory and you need to order cells along a specific branch and identify temporally phased waves of gene regulation along that branch.

## When NOT to use

- Trajectory structure has not yet been established — Mpath derives multi-branching trajectories via neighborhood-based cell state transitions before reordering, so this skill assumes a path already exists
- Developmental process starts from multiple independent progenitor cell types or subset information is unknown — the article notes Mpath does not yet support these settings
- Data lack signature genes of differentiated (end-stage) cell types, since Mpath fate prediction depends on end-stage contrasts

## Inputs

- Reordered (pseudo-temporally ordered) single cells along the cDC2 developmental path
- Reordered (pseudo-temporally ordered) single cells along the cDC1 developmental path
- Path-specific differentially expressed gene lists (Supplementary Data 2 for cDC2 path; Supplementary Data 3 for cDC1 path)

## Outputs

- Gene-to-wave cluster assignments per path (temporal_wave_clusters.csv)
- Per-path cluster summaries (7 wave clusters on cDC2; 6 on cDC1)
- Pathway annotations of wave clusters (e.g., mitosis, DNA synthesis, MHC class II antigen processing and presentation)

## How to apply

First obtain the reordered cells on the lineage-specific path (e.g., cDC2 or cDC1) together with that path's differentially expressed gene list (Supplementary Data 2 for early CDP to late cDC2-committed preDC; Supplementary Data 3 for early CDP to late cDC1-committed preDC). For each differentially regulated gene, fit a generalized additive model (GAM) of expression as a smooth function of pseudo-temporal position along the reordered cells using Mpath's R implementation. Cluster the fitted gene expression dynamics into temporal wave clusters per path and annotate clusters with pathway information (e.g., Ingenuity Pathway Analysis terms) to identify mitosis and DNA-synthesis gene sets and confirm their waves are out of phase. Validation requires recovering exactly 7 wave clusters on the cDC2 path and 6 on the cDC1 path, with mitosis/synthesis genes assigned to temporally out-of-phase waves, and writing gene-to-wave assignments to temporal_wave_clusters.csv.

## Related tools

- **Mpath** (R implementation used to fit GAMs of gene expression as a smooth function of pseudo-temporal position and cluster fitted dynamics into temporal wave clusters) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Environment in which the Mpath package is executed for GAM fitting and clustering)
- **Ingenuity Pathway Analysis (IPA)** (Used to annotate wave clusters with pathway terms to identify mitosis and DNA-synthesis gene sets and MHC class II antigen processing enrichment)

## Examples

```
Rscript -e 'library(Mpath); waves <- clusterGeneDynamics(gam.fit, path="cDC2"); write.csv(waves, "temporal_wave_clusters.csv")'
```

## Evaluation signals

- Cluster analysis on the reordered cDC2 path recovers exactly 7 distinct temporal wave clusters
- Cluster analysis on the reordered cDC1 path recovers exactly 6 distinct temporal wave clusters
- Mitosis and DNA-synthesis genes are assigned to two nearly out-of-phase waves, reflecting temporal coupling between cell cycle and cDC differentiation
- The final wave on both paths is enriched for MHC class II antigen processing and presentation
- temporal_wave_clusters.csv contains gene-to-wave cluster assignments for every differentially regulated gene on both paths

## Limitations

- Mpath does not yet support developmental processes starting from multiple independent progenitor cell types
- Mpath is not yet extended to highly heterogeneous single-cell data sets in which specific subset information is unknown
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction

## Evidence

- [results] GAM-based differential gene expression on pseudo-temporally ordered cells: [evidence span withheld — anchor status: not_found]
- [abstract] Reordering reveals sequential waves and cell cycle coupling: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [supplementary] Mpath is an R package: "Mpath reference manual provides detailed documentation of Mpath R package."
- [supplementary] Pathway terms defined by IPA: "as defined by Ingenuity Pathway Analysis (IPA)"
- [discussion] Switch from proliferation to differentiation: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation."
