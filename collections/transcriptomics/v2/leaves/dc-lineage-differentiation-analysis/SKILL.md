---
name: dc-lineage-differentiation-analysis
description: Use when single-cell RNA-seq data on dendritic cell progenitors must
  be ordered along cDC2 or cDC1 differentiation trajectories to identify how many
  distinct temporal gene regulation waves exist and which genes/pathways occupy each
  wave.
when_to_use_negative:
- The developmental process starts from multiple independent progenitor cell types
  — the article notes Mpath does not yet support this (planned future extension)
- The dataset is highly heterogeneous with no known subset information, since Mpath
  depends on signature genes of differentiated cell types and FACS-sorted terminal
  subsets for fate prediction
- The analysis target is a different tissue/system without path-specific DEG lists
  (the demonstrated utility is limited to mouse bone marrow-derived DC progenitors
  and primary human myoblasts)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Mpath
  role: R implementation used to derive the multi-branching cDC trajectories, reorder
    cells by pseudo-temporal position, fit GAMs of expression along the paths, and
    cluster fitted gene dynamics into temporal wave clusters
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Environment in which the Mpath package is run to load reordered cells, fit
    per-gene GAMs, and perform wave clustering
- name: Ingenuity Pathway Analysis (IPA)
  role: Used to annotate temporal wave clusters with pathway terms to identify mitosis/DNA-synthesis
    gene sets and MHC class II antigen processing enrichment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:cb49a6c6acad0e85d36c3869e2201d31d60f86e00bedcf22168083f3c958c530
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dc-lineage-differentiation-analysis

## Summary

Reproduces the sequential waves of gene regulation along mouse cDC developmental paths by pseudo-temporally reordering single cells with Mpath, fitting GAMs of expression versus pseudo-temporal position, and clustering fitted gene dynamics into temporal wave clusters on the cDC2 and cDC1 paths.

## When to use

Use when single-cell RNA-seq data on dendritic cell progenitors must be ordered along cDC2 or cDC1 differentiation trajectories to identify how many distinct temporal gene regulation waves exist and which genes/pathways occupy each wave.

## When NOT to use

- The developmental process starts from multiple independent progenitor cell types — the article notes Mpath does not yet support this (planned future extension)
- The dataset is highly heterogeneous with no known subset information, since Mpath depends on signature genes of differentiated cell types and FACS-sorted terminal subsets for fate prediction
- The analysis target is a different tissue/system without path-specific DEG lists (the demonstrated utility is limited to mouse bone marrow-derived DC progenitors and primary human myoblasts)

## Inputs

- Reordered (pseudo-temporally ordered) single cells on the cDC2 path
- Reordered single cells on the cDC1 path
- Path-specific differentially expressed gene lists: Supplementary Data 2 (early CDP to late cDC2-committed preDC) and Supplementary Data 3 (early CDP to late cDC1-committed preDC)

## Outputs

- temporal_wave_clusters.csv with gene-to-wave cluster assignments and per-path cluster summaries
- Verified 7 temporal wave clusters on the cDC2 path and 6 on the cDC1 path
- Pathway annotations per wave cluster (e.g. IPA terms) identifying mitosis, DNA-synthesis, and MHC class II gene sets

## How to apply

Load the reordered cells on the cDC2 and cDC1 paths together with the path-specific differentially expressed gene lists (Supplementary Data 2 for early CDP to late cDC2-committed preDC and Supplementary Data 3 for early CDP to late cDC1-committed preDC) into R. For each differentially regulated gene, fit a generalized additive model (GAM) of expression as a smooth function of pseudo-temporal position along the reordered path using Mpath's R implementation, then cluster the fitted expression dynamics into temporal wave clusters. Annotate each wave cluster with pathway terms (e.g. via Ingenuity Pathway Analysis) to locate mitosis and DNA-synthesis gene sets and confirm the temporal coupling between cell cycle and cDC differentiation. The expected result is seven distinct wave clusters on the cDC2 path and six on the cDC1 path, with mitosis and synthesis genes in two nearly out-of-phase waves and the final wave on both paths enriched for MHC class II antigen processing and presentation.

## Related tools

- **Mpath** (R implementation used to derive the multi-branching cDC trajectories, reorder cells by pseudo-temporal position, fit GAMs of expression along the paths, and cluster fitted gene dynamics into temporal wave clusters) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Environment in which the Mpath package is run to load reordered cells, fit per-gene GAMs, and perform wave clustering)
- **Ingenuity Pathway Analysis (IPA)** (Used to annotate temporal wave clusters with pathway terms to identify mitosis/DNA-synthesis gene sets and MHC class II antigen processing enrichment)

## Examples

```
library(Mpath); clusters <- Mpath::clusterGenes(expr, pseudotime, paths=list(cDC2=SD2_genes, cDC1=SD3_genes)); write.csv(clusters, 'temporal_wave_clusters.csv')
```

## Evaluation signals

- The GAM-based clustering recovers exactly 7 distinct wave clusters on the cDC2 path
- The GAM-based clustering recovers exactly 6 distinct wave clusters on the cDC1 path
- Mitosis and DNA-synthesis gene sets are assigned to two nearly out-of-phase temporal waves
- The final wave on both the cDC2 and cDC1 paths is enriched for MHC class II antigen processing and presentation
- temporal_wave_clusters.csv is produced with complete gene-to-wave assignments for all genes in the Supplementary Data 2 and Supplementary Data 3 lists

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction

## Evidence

- [abstract] GAM wave clustering on cDC paths: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [supplementary] Supplementary Data 2 content: "Supplementary Data 2 include genes that were differentially expressed along the development from early CDP to late cDC2-committed preDC."
- [supplementary] Supplementary Data 3 content: "Supplementary Data 3 include genes that were differentially expressed along the development from early CDP to late cDC1-committed preDC."
- [supplementary] IPA annotation of gene sets: "as defined by Ingenuity Pathway Analysis (IPA)"
- [abstract] Mpath algorithm description: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [readme] Mpath R package documentation: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
