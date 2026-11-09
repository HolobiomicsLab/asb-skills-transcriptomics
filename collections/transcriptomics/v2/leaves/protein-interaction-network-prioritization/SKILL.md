---
name: protein-interaction-network-prioritization
description: Use when you have cell-cluster-annotated scRNA-seq data (human or mouse)
  and need to identify, score, and rank ligand-receptor interactions between sender
  and receiver populations, as in SCALA's ligand-receptor analysis mode.
when_to_use_negative:
- The input dataset is from an organism other than human or mouse — SCALA and nichenetr
  currently support only human and mouse analysis
- Clusters are not yet annotated / no sender-receiver population pair has been defined
  — ligand-receptor scoring requires annotated cell populations as input
- The goal is basic exploratory single-cell analysis (QC, normalization, clustering)
  rather than intercellular communication inference between defined populations
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: nichenetr
  role: Core engine of the ligand-receptor analysis layer; supplies the prior model
    integrating ligand-receptor interactions, signaling pathways, and TF regulation
    to compute interaction scores
  repo: https://github.com/saeyslab/nichenetr
- name: SCALA
  role: Web application embedding the nichenetr-based ligand-receptor module with
    interactive selection of sender/receiver clusters and downloadable outputs
  repo: https://github.com/PavlopoulosLab/SCALA
- name: Seurat
  role: Provides graph-based Louvain clustering in PCA space that generates the annotated
    cluster labels consumed by the ligand-receptor analysis
- name: R
  role: Implementation language for SCALA and nichenetr; analysis is scripted/executed
    in R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:82e3fcf8bb6724bb67b98de7c19adcc699bc9ee60cbfc8035c05970005d4e72b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/saeyslab/nichenetr
  tools:
  - R
  - SCALA
  - Seurat
  - nichenetr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# protein-interaction-network-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs SCALA's ligand-receptor analysis layer, which employs the nichenetr (NicheNet) R package to score interactions between annotated cell populations and rank ligand-receptor pairs by a 'prior interaction potential' score derived from a protein-protein interaction model.

## When to use

Use when you have cell-cluster-annotated scRNA-seq data (human or mouse) and need to identify, score, and rank ligand-receptor interactions between sender and receiver populations, as in SCALA's ligand-receptor analysis mode.

## When NOT to use

- The input dataset is from an organism other than human or mouse — SCALA and nichenetr currently support only human and mouse analysis
- Clusters are not yet annotated / no sender-receiver population pair has been defined — ligand-receptor scoring requires annotated cell populations as input
- The goal is basic exploratory single-cell analysis (QC, normalization, clustering) rather than intercellular communication inference between defined populations

## Inputs

- Processed scRNA-seq dataset with Louvain cluster annotations (e.g., Seurat object on PBMC3k)
- User-selected sender cluster (ligand-expressing) and receiver cluster (receptor-expressing)
- Cell type-annotated gene expression data from human or mouse (single-cell or sorted bulk)
- Overexpressed gene lists computed per cluster (nichenetr requirement)

## Outputs

- lr_interactions_scores.csv — ranked table of ligand-receptor interactions with a 'prior interaction potential' score per interaction
- Heatmap of 'bona fide' ligand-receptor interactions (analogous to SCALA Fig. 2F)
- Shared and specific interaction counts between cluster pairs (e.g., 157 sublining-lining and 152 intermediate-lining interactions in the hTNFtg use case)

## How to apply

Load the processed scRNA-seq dataset with Louvain cluster annotations (e.g., the PBMC3k dataset clustered via Seurat graph-based clustering in PCA space). For a user-selected sender (ligand-expressing) and receiver (receptor-expressing) cluster pair — e.g., cluster 0 as sender and cluster 2 as receiver — compute overexpressed genes in each cluster as required by nichenetr, and restrict analysis to ligands/receptors expressed in more than 10% of cells in the clusters of interest. Construct the protein-protein interaction (PPI) model to obtain the 'prior interaction potential' score, then rank reported ligand-receptor interactions between the two clusters by this score. Emit a downloadable table (lr_interactions_scores.csv) containing the score for every interaction, and render 'bona fide' interactions as a heatmap analogous to SCALA's Fig. 2F. NicheNet goes beyond expression-only ligand-receptor linking by integrating a prior model over ligand-to-target signaling paths, so ranking reflects predicted regulatory influence on receiver cells rather than mere co-expression.

## Related tools

- **nichenetr** (Core engine of the ligand-receptor analysis layer; supplies the prior model integrating ligand-receptor interactions, signaling pathways, and TF regulation to compute interaction scores) — https://github.com/saeyslab/nichenetr
- **SCALA** (Web application embedding the nichenetr-based ligand-receptor module with interactive selection of sender/receiver clusters and downloadable outputs) — https://github.com/PavlopoulosLab/SCALA
- **Seurat** (Provides graph-based Louvain clustering in PCA space that generates the annotated cluster labels consumed by the ligand-receptor analysis)
- **R** (Implementation language for SCALA and nichenetr; analysis is scripted/executed in R)

## Examples

```
if(!requireNamespace("devtools", quietly=TRUE)){install.packages("devtools")}; devtools::install_github("saeyslab/nichenetr")
```

## Evaluation signals

- lr_interactions_scores.csv exists and parses as valid CSV
- Every reported interaction has a 'prior interaction potential' score field
- All scores are non-negative and the table is sorted so top-ranked entries correspond to the strongest predicted interactions
- Interaction counts match the article's reported use case values (e.g., 157 sublining-lining and 152 intermediate-lining interactions in hTNFtg)
- Bona fide interaction heatmap is comparable to the reported cluster 0–2 interaction heatmap in SCALA's Fig. 2F

## Limitations

- Only human and mouse datasets are currently supported for ligand-receptor analysis
- The online SCALA version restricts dataset size to < 2 GB with at most 2 CPU threads; for datasets > 50,000 cells some steps cannot be completed online and the desktop version is recommended
- NicheNet prioritization depends on the quality of the prior knowledge model (ligand-receptor, signaling, and TF-regulation databases); it predicts candidate interactions, not experimentally confirmed ones

## Evidence

- [readme] NicheNet uses human or mouse gene expression data of interacting cells as input and combines this with a prior model that integrates existing knowledge on ligand-to-target signaling paths.: [evidence span withheld — anchor status: external_doc]
