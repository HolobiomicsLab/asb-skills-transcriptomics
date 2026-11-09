---
name: interaction-heatmap-visualization
description: Use when a scRNA-seq dataset has annotated cell clusters (e.g., Seurat
  Louvain clusters) and the goal is to score, rank, and visualize bona fide ligand-receptor
  interactions between a sender and a receiver population.
when_to_use_negative:
- The input lacks cell-type/cluster annotations, so sender and receiver populations
  cannot be defined for scoring interactions.
- The goal is scATAC-seq-only regulatory analysis (e.g., GRN/chromVAR motif enrichment)
  rather than intercellular communication scoring.
- The dataset is from an organism other than human or mouse, since SCALA's analysis
  of only human and mouse datasets is currently supported.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Web/desktop application hosting the ligand-receptor analysis module that scores
    interactions and renders the bona fide interaction heatmap
  repo: https://github.com/PavlopoulosLab/SCALA
- name: nichenetr
  role: R implementation of the NicheNet method used by SCALA to compute overexpressed
    genes and rank ligand-receptor interactions via the prior interaction potential
  repo: https://github.com/saeyslab/nichenetr
- name: Seurat
  role: Provides graph-based clustering in PCA space that yields the Louvain cluster
    annotations used as input populations
- name: R
  role: Implementation language in which SCALA and its ligand-receptor layer are developed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:71f5e953020a1a61d49c905eaac835169f0ea34443e9c75b689f0141d0fdc881
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - R
  - SCALA
  - Seurat
  - nichenetr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Interaction Heatmap Visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders ligand-receptor interactions between annotated single-cell clusters as a scored, ranked heatmap, as implemented in SCALA's ligand-receptor layer built on the nichenetr (NicheNet) framework. Interactions are filtered by expression prevalence (percentage of expression > 10% in clusters of interest) and ranked by the 'prior interaction potential' score.

## When to use

Use when a scRNA-seq dataset has annotated cell clusters (e.g., Seurat Louvain clusters) and the goal is to score, rank, and visualize bona fide ligand-receptor interactions between a sender and a receiver population.

## When NOT to use

- The input lacks cell-type/cluster annotations, so sender and receiver populations cannot be defined for scoring interactions.
- The goal is scATAC-seq-only regulatory analysis (e.g., GRN/chromVAR motif enrichment) rather than intercellular communication scoring.
- The dataset is from an organism other than human or mouse, since SCALA's analysis of only human and mouse datasets is currently supported.

## Inputs

- Processed scRNA-seq dataset with cluster annotations (e.g., PBMC3k with Seurat Louvain clusters)
- User-selected ligand-expressing (sender) cluster and receptor-expressing (receiver) cluster
- Overexpressed gene lists per cluster computed within the nichenetr framework

## Outputs

- lr_interactions_scores.csv — ranked table of ligand-receptor interactions with a 'prior interaction potential' score per interaction
- Heatmap of bona fide ligand-receptor interactions between the selected clusters (analogous to SCALA Fig. 2F)

## How to apply

Load a processed scRNA-seq object with Louvain cluster annotations (e.g., the PBMC3k dataset clustered via Seurat graph-based clustering in PCA space). For the user-selected sender-receiver cluster pair (e.g., cluster 0 as ligand-expressing sender and cluster 2 as receptor-expressing receiver), compute overexpressed genes in each cluster as required by the nichenetr framework. Retain only ligands/receptors with a percentage of expression > 10% in the clusters of interest, then construct the protein-protein interaction model to obtain the 'prior interaction potential' score and rank reported interactions by it. Emit a downloadable table (lr_interactions_scores.csv) and render the 'bona fide' interactions as a heatmap analogous to SCALA's Fig. 2F in the hTNFtg synovial fibroblast use case.

## Related tools

- **SCALA** (Web/desktop application hosting the ligand-receptor analysis module that scores interactions and renders the bona fide interaction heatmap) — https://github.com/PavlopoulosLab/SCALA
- **nichenetr** (R implementation of the NicheNet method used by SCALA to compute overexpressed genes and rank ligand-receptor interactions via the prior interaction potential) — https://github.com/saeyslab/nichenetr
- **Seurat** (Provides graph-based clustering in PCA space that yields the Louvain cluster annotations used as input populations)
- **R** (Implementation language in which SCALA and its ligand-receptor layer are developed)

## Examples

```
devtools::install_github("saeyslab/nichenetr"); vignette("seurat_steps", package="nichenetr")
```

## Evaluation signals

- lr_interactions_scores.csv exists and parses as valid CSV with one row per reported ligand-receptor pair.
- Every reported interaction carries a 'prior interaction potential' score field.
- All scores are non-negative, and the table is sorted so top-ranked entries correspond to the strongest predicted interactions.
- Only ligands/receptors with percentage of expression > 10% in the clusters of interest appear in the output.
- The heatmap reproduces the reported interaction pattern for the selected pair (e.g., 157 sublining-lining and 152 intermediate-lining interactions in the hTNFtg use case).

## Limitations

- Only human and mouse datasets are currently supported in SCALA for this analysis mode.
- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session, and some steps cannot be completed online for datasets larger than 50,000 cells; prefer the desktop version for larger datasets.
- NicheNet's predictions depend on its prior knowledge model (ligand-receptor interactions, signaling pathways, TF regulation), so scores reflect prior-based potential rather than direct measurement.

## Evidence

- [other] focusing on ligands/receptors with a percentage of expression > 10% in the clusters of interest, from which it identifies shared and specific interactions: [evidence span withheld — anchor status: not_found]
- [readme] NicheNet uses human or mouse gene expression data of interacting cells as input and combines this with a prior model that integrates existing knowledge on ligand-to-target signaling paths.: [evidence span withheld — anchor status: external_doc]
- [readme] As output, NicheNet returns the ranking of ligands that best explain the CCC event of interest, as well as candidate target genes with high potential to be regulated by these ligands.: [evidence span withheld — anchor status: external_doc]
