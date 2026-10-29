---
name: temporal-wave-clustering-of-differentially-regulated-genes
description: Use when cells have been reordered along a pseudo-temporal developmental
  trajectory and you need to identify how many distinct temporal waves of gene regulation
  exist and which genes (e.g. mitosis, DNA synthesis, MHC-II) occupy each wave.
when_to_use_negative:
- Cells have not been reordered along a developmental path — without pseudo-temporal
  ordering there is no axis on which to fit the GAMs (the skill presumes Mpath-derived
  cell ordering)
- Gene lists are not path-specific DEGs from end-stage contrasts — using genome-wide
  or unfiltered gene sets will not reproduce the defined wave clusters
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: R implementation used to fit GAMs of expression as a smooth function of pseudo-temporal
    position and cluster fitted gene dynamics into temporal waves on reordered cells
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Environment in which the Mpath GAM fitting and gene clustering workflow is
    executed
- name: Ingenuity Pathway Analysis (IPA)
  role: Pathway annotation of wave clusters to identify mitosis/DNA-synthesis gene
    sets and MHC class II antigen processing enrichment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:c6d214992fcae93fb9d15ac5f0312f84c4ac21832244a93faa083052fb3992ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Ingenuity Pathway Analysis (IPA)
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# temporal-wave-clustering-of-differentially-regulated-genes

## Summary

Fits GAMs of gene expression as a smooth function of pseudo-temporal position along reordered cells on a differentiation path (e.g. cDC2 and cDC1 paths from early CDP), then clusters fitted dynamics to recover sequential waves of gene regulation and annotate them with pathway information.

## When to use

Use when cells have been reordered along a pseudo-temporal developmental trajectory and you need to identify how many distinct temporal waves of gene regulation exist and which genes (e.g. mitosis, DNA synthesis, MHC-II) occupy each wave.

## When NOT to use

- Cells have not been reordered along a developmental path — without pseudo-temporal ordering there is no axis on which to fit the GAMs (the skill presumes Mpath-derived cell ordering)
- Gene lists are not path-specific DEGs from end-stage contrasts — using genome-wide or unfiltered gene sets will not reproduce the defined wave clusters

## Inputs

- Reordered (pseudo-temporally ordered) single cells along the cDC2 path
- Reordered single cells along the cDC1 path
- Path-specific differentially expressed gene lists (Supplementary Data 2: early CDP to late cDC2-committed preDC; Supplementary Data 3: early CDP to late cDC1-committed preDC)

## Outputs

- Gene-to-wave cluster assignments per path (temporal_wave_clusters.csv)
- Per-path cluster summaries: 7 wave clusters on cDC2, 6 on cDC1
- Pathway annotations per wave (IPA terms), including out-of-phase mitosis and DNA-synthesis waves and final MHC class II antigen processing wave

## How to apply

Load the reordered cells on each path together with the path-specific differentially expressed gene lists (Supplementary Data 2 for early CDP to late cDC2-committed preDC, Supplementary Data 3 for early CDP to late cDC1-committed preDC). For each differentially regulated gene, fit a generalized additive model (GAM) of expression as a smooth function of pseudo-temporal position using Mpath's R implementation. Cluster the fitted gene expression dynamics into temporal wave clusters per path — the expected result is 7 clusters on the cDC2 path and 6 on the cDC1 path. Annotate clusters with pathway terms via Ingenuity Pathway Analysis (IPA) to identify mitosis and DNA-synthesis gene sets, confirming they occupy two nearly out-of-phase waves that reflect the temporal coupling between cell cycle and cDC differentiation, and that the final wave on both paths is enriched for MHC class II antigen processing and presentation. Write gene-to-wave assignments and per-path summaries to temporal_wave_clusters.csv.

## Related tools

- **Mpath** (R implementation used to fit GAMs of expression as a smooth function of pseudo-temporal position and cluster fitted gene dynamics into temporal waves on reordered cells) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Environment in which the Mpath GAM fitting and gene clustering workflow is executed)
- **Ingenuity Pathway Analysis (IPA)** (Pathway annotation of wave clusters to identify mitosis/DNA-synthesis gene sets and MHC class II antigen processing enrichment)

## Examples

```
Rscript fit_gam_wave_clusters.R --path cDC2 --genes SupplementaryData2.csv --reordered_cells cdc2_reordered.csv --k 7 --out temporal_wave_clusters.csv
```

## Evaluation signals

- Cluster analysis recovers exactly 7 distinct wave clusters on the cDC2 path
- Cluster analysis recovers exactly 6 distinct wave clusters on the cDC1 path
- Mitosis and DNA-synthesis genes are assigned to two nearly out-of-phase temporal waves on both paths
- The final wave on both paths is enriched for MHC class II antigen processing and presentation terms
- Output file temporal_wave_clusters.csv contains a complete gene-to-wave-cluster assignment for every input differentially regulated gene

## Limitations

- The expected cluster counts (7 on cDC2, 6 on cDC1) are specific to the mouse cDC progenitor data set and its path-specific DEG lists; other data sets may require re-deriving cluster numbers
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for trajectory/fate prediction, so path definitions must already exist
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] phase-dependent waves of gene regulation: "its ordering of single cells discovered phase-dependent waves of gene regulation on the scale of DC development and subset differentiation continuum."
- [supplementary] Supplementary Data 2 cDC2 DEGs: "Supplementary Data 2 include genes that were differentially expressed along the development from early CDP to late cDC2-committed preDC."
- [supplementary] Supplementary Data 3 cDC1 DEGs: "Supplementary Data 3 include genes that were differentially expressed along the development from early CDP to late cDC1-committed preDC."
- [readme] Mpath R package documentation: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
