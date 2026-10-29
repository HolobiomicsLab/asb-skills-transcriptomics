---
name: gam-based-pseudotime-expression-modeling
description: Use when cells have been ordered along a developmental trajectory (e.g.,
  early CDP to late cDC2- or cDC1-committed preDC) and you need to identify distinct
  temporal waves of gene regulation from path-specific differentially expressed gene
  lists.
when_to_use_negative:
- Cells have not yet been reordered along a developmental trajectory — Mpath trajectory
  construction must precede pseudo-temporal GAM fitting
- The dataset starts from multiple independent progenitor cell types, a case Mpath
  does not yet support (planned future extension)
- The analysis is a simple end-stage contrast between subsets rather than temporal
  dynamics along a path — ANOVA with post-hoc Tukey's test suffices
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Mpath
  role: R implementation used to fit GAMs of expression as a smooth function of pseudo-temporal
    position and cluster fitted gene dynamics into wave clusters
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Environment in which Mpath is run to load reordered cells and DEG lists, fit
    GAMs, and write temporal_wave_clusters.csv
- name: Ingenuity Pathway Analysis (IPA)
  role: Pathway annotation of wave clusters to identify mitosis, DNA-synthesis, and
    MHC class II gene sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:8efbd5792e0386ac13c2f4be4d65139309ac19da993d3bbc00b1929579abd2a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Ingenuity Pathway Analysis (IPA)
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gam-based-pseudotime-expression-modeling

## Summary

Fits generalized additive models (GAMs) of gene expression as a smooth function of pseudo-temporal position along reordered single-cell developmental trajectories (cDC2 and cDC1 paths), then clusters fitted dynamics into temporal waves of gene regulation.

## When to use

Use when cells have been ordered along a developmental trajectory (e.g., early CDP to late cDC2- or cDC1-committed preDC) and you need to identify distinct temporal waves of gene regulation from path-specific differentially expressed gene lists.

## When NOT to use

- Cells have not yet been reordered along a developmental trajectory — Mpath trajectory construction must precede pseudo-temporal GAM fitting
- The dataset starts from multiple independent progenitor cell types, a case Mpath does not yet support (planned future extension)
- The analysis is a simple end-stage contrast between subsets rather than temporal dynamics along a path — ANOVA with post-hoc Tukey's test suffices

## Inputs

- Reordered single cells along the cDC2 developmental path (early CDP to late cDC2-committed preDC)
- Reordered single cells along the cDC1 developmental path (early CDP to late cDC1-committed preDC)
- Path-specific differentially expressed gene lists (Supplementary Data 2 and Supplementary Data 3)

## Outputs

- temporal_wave_clusters.csv with gene-to-wave cluster assignments and per-path cluster summaries
- Per-path counts of distinct temporal wave clusters (7 on the cDC2 path, 6 on the cDC1 path)
- IPA pathway annotations of wave clusters (e.g., MHC class II antigen processing and presentation, mitosis, DNA synthesis)

## How to apply

Load the reordered cells on the cDC2 and cDC1 paths together with the path-specific DEG lists (Supplementary Data 2 for early CDP to late cDC2-committed preDC and Supplementary Data 3 for early CDP to late cDC1-committed preDC) into R. For each differentially regulated gene, fit a GAM of expression as a smooth function of pseudo-temporal position using Mpath's R implementation, then cluster the fitted expression dynamics into temporal wave clusters per path. Annotate clusters with Ingenuity Pathway Analysis (IPA) terms to identify biological processes such as mitosis and DNA synthesis, and check whether these occupy out-of-phase waves reflecting temporal coupling between cell cycle and cDC differentiation. Write gene-to-wave cluster assignments and per-path summaries to temporal_wave_clusters.csv.

## Related tools

- **Mpath** (R implementation used to fit GAMs of expression as a smooth function of pseudo-temporal position and cluster fitted gene dynamics into wave clusters) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Environment in which Mpath is run to load reordered cells and DEG lists, fit GAMs, and write temporal_wave_clusters.csv)
- **Ingenuity Pathway Analysis (IPA)** (Pathway annotation of wave clusters to identify mitosis, DNA-synthesis, and MHC class II gene sets)

## Examples

```
Rscript gam_wave_clustering.R --path cDC2 --degs Supplementary_Data_2.csv --reordered_cells cdc2_reordered.csv --out temporal_wave_clusters.csv
```

## Evaluation signals

- Cluster analysis recovers exactly 7 distinct wave clusters on the cDC2 path and 6 on the cDC1 path
- Mitosis and DNA-synthesis gene sets are assigned to two nearly out-of-phase waves, reflecting temporal coupling between cell cycle and cDC differentiation
- The final wave on both paths is enriched for MHC class II antigen processing and presentation
- temporal_wave_clusters.csv contains a valid gene-to-cluster assignment for every gene in the Supplementary Data 2 and 3 DEG lists

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, so a priori subset knowledge is required
- Demonstrated utility limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts
- Mpath does not yet support development processes starting from multiple independent progenitor cell types or highly heterogeneous datasets where specific subset information is unknown

## Evidence

- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation.: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] Phase-dependent waves via cell ordering: "its ordering of single cells discovered phase-dependent waves of gene regulation on the scale of DC development and subset differentiation continuum."
- [supplementary] Cluster analysis of DEGs: "Cluster analysis of the DEGS identified 5 distinct clusters as shown in Supplementary Figure 2a."
- [readme] Mpath algorithm definition: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
