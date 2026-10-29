---
name: ligand-receptor-interaction-scoring
description: Use when a Seurat-processed scRNA-seq dataset has annotated cell clusters
  and the goal is to identify and rank ligands expressed by one cluster against receptors
  on another, filtering by >10% expression of ligands/receptors in the clusters of
  interest.
when_to_use_negative:
- The dataset is scATAC-seq only (no gene expression profiles), since NicheNet requires
  human or mouse gene expression data of interacting cells
- The organism is neither human nor mouse — SCALA supports analysis of only human
  and mouse datasets
- The clusters have no cell-type annotation and ligand/receiver populations cannot
  be defined (sender-agnostic prioritization would be needed instead)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Web/desktop application whose ligand-receptor analysis mode wraps nichenetr
    to score and rank interactions between annotated cell clusters and render bona
    fide interaction heatmaps
  repo: https://github.com/PavlopoulosLab/SCALA
- name: nichenetr
  role: R implementation of the NicheNet method providing the prior model (ligand-to-target
    signaling paths) and ligand-activity scoring used to compute interaction potential
    between sender and receiver populations
  repo: https://github.com/saeyslab/nichenetr
- name: Seurat
  role: Provides the upstream QC, normalization, PCA-based dimensionality reduction
    and Louvain clustering that yield the annotated clusters consumed as input by
    the ligand-receptor step
- name: R / Shiny / JavaScript
  role: Languages in which SCALA and its ligand-receptor analysis layer are implemented
    and served
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:e408d107afe04fd4751335a0a15eaf3ec61175925d25d0eb09c1cffe7e6cabc2
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
  - R / Shiny / JavaScript
  - SCALA
  - Seurat
  - nichenetr
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ligand-receptor-interaction-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores ligand-receptor interactions between annotated single-cell clusters using NicheNet (nichenetr) as implemented in SCALA, ranking cell-cell communication events by the 'prior interaction potential' derived from a ligand-to-target signaling prior model.

## When to use

Use when a Seurat-processed scRNA-seq dataset has annotated cell clusters and the goal is to identify and rank ligands expressed by one cluster against receptors on another, filtering by >10% expression of ligands/receptors in the clusters of interest.

## When NOT to use

- The dataset is scATAC-seq only (no gene expression profiles), since NicheNet requires human or mouse gene expression data of interacting cells
- The organism is neither human nor mouse — SCALA supports analysis of only human and mouse datasets
- The clusters have no cell-type annotation and ligand/receiver populations cannot be defined (sender-agnostic prioritization would be needed instead)

## Inputs

- Processed scRNA-seq dataset with annotated Louvain clusters (e.g., Seurat object from PBMC3k)
- User-selected sender cluster (ligand-expressing) and receiver cluster (receptor-expressing)
- NicheNet prior model integrating ligand-receptor interactions, signaling pathways, and TF regulation (human or mouse)

## Outputs

- lr_interactions_scores.csv — downloadable table of ligand-receptor interactions ranked by 'prior interaction potential' score
- Heatmap of 'bona fide' ligand-receptor interactions (analogous to SCALA Fig. 2F)
- Counts of shared and specific interactions between cluster pairs (e.g., 157 sublining-lining and 152 intermediate-lining in the hTNFtg case)

## How to apply

Load the processed scRNA-seq object (e.g., PBMC3k with Louvain clusters from Seurat graph-based clustering in PCA space) and select a sender cluster (ligand-expressing) and receiver cluster (receptor-expressing). Following SCALA's nichenetr-based layer, restrict ligands and receptors to those with a percentage of expression > 10% in the clusters of interest, then compute overexpressed genes per cluster as required by the nichenetR framework. Construct the protein-protein interaction prior model to obtain the 'prior interaction potential' score for each ligand-receptor pair, and rank all reported interactions between the two clusters by this score. Emit a downloadable interaction table (lr_interactions_scores.csv) including the score, and render 'bona fide' interactions as a heatmap analogous to SCALA's Fig. 2F; in the paper's hTNFtg use case this yielded 157 sublining-lining and 152 intermediate-lining interactions.

## Related tools

- **SCALA** (Web/desktop application whose ligand-receptor analysis mode wraps nichenetr to score and rank interactions between annotated cell clusters and render bona fide interaction heatmaps) — https://github.com/PavlopoulosLab/SCALA
- **nichenetr** (R implementation of the NicheNet method providing the prior model (ligand-to-target signaling paths) and ligand-activity scoring used to compute interaction potential between sender and receiver populations) — https://github.com/saeyslab/nichenetr
- **Seurat** (Provides the upstream QC, normalization, PCA-based dimensionality reduction and Louvain clustering that yield the annotated clusters consumed as input by the ligand-receptor step)
- **R / Shiny / JavaScript** (Languages in which SCALA and its ligand-receptor analysis layer are implemented and served)

## Examples

```
devtools::install_github("saeyslab/nichenetr"); nichenet_seuratobj_aggregate(seurat_obj, sender = "cluster_0", receiver = "cluster_2", expression_pct = 0.10)
```

## Evaluation signals

- lr_interactions_scores.csv exists and is valid CSV with a 'prior interaction potential' score field present for every reported interaction
- All 'prior interaction potential' scores are non-negative
- The table is sorted so top-ranked entries correspond to the strongest predicted interactions
- Bona fide interactions render as a heatmap matching the reported cluster 0–2 interaction pattern in SCALA's Fig. 2F
- In the hTNFtg use case, the expected interaction counts (~157 sublining-lining and 152 intermediate-lining interactions) are recovered

## Limitations

- Only human and mouse datasets are currently supported for ligand-receptor analysis (SCALA scope: 'the analysis of only human and mouse datasets is currently supported')
- The online SCALA version is restricted to datasets < 2 GB with no more than 2 CPU threads per session; for datasets > 50,000 cells some steps cannot be completed online and the desktop version is recommended
- NicheNet goes beyond ligand-receptor pairs by linking ligands to downstream target genes via signaling and TF-regulation priors, so its rankings reflect regulatory potential rather than only co-expression of ligand and receptor

## Evidence
