---
name: cluster-overexpressed-gene-identification
description: Use when a scRNA-seq dataset has annotated cell clusters (e.g., Seurat
  Louvain clusters) and you need per-cluster overexpressed genes to feed NicheNet-style
  ligand-receptor interaction scoring between sender and receiver clusters.
when_to_use_negative:
- The input lacks cluster or cell-type annotations — NicheNet requires cell type-annotated
  expression data reflecting the cell-cell communication event of interest
- The dataset is from an organism other than human or mouse — SCALA currently supports
  only human and mouse datasets
- You need differential expression per condition rather than per-cluster overexpression
  for cross-cluster communication — use standard marker identification instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCALA
  role: Interactive pipeline that computes overexpressed genes per annotated cluster
    and applies the >10% expression filter before NicheNet-based ligand-receptor scoring
- name: nichenetr
  role: R implementation of the NicheNet method; requires overexpressed genes per
    sender/receiver cluster as input to its prior-model-based ligand activity analysis
  repo: https://github.com/saeyslab/nichenetr
- name: Seurat
  role: Provides the graph-based clustering in PCA space that produces the Louvain
    cluster annotations over which overexpressed genes are computed
- name: presto
  role: Fast differential expression / marker discovery used to identify cluster markers
    in single-cell data
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:50ef44503c49b9e0dbbe708e21d06ce79b0de4e34752f62686af9c9b0a653deb
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
  - presto
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Cluster overexpressed gene identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies genes overexpressed in annotated cell clusters (e.g., Louvain clusters from Seurat graph-based clustering in PCA space) as a prerequisite step for SCALA's NicheNet-based ligand-receptor analysis, which focuses on ligands/receptors expressed in >10% of cells in the clusters of interest.

## When to use

Use when a scRNA-seq dataset has annotated cell clusters (e.g., Seurat Louvain clusters) and you need per-cluster overexpressed genes to feed NicheNet-style ligand-receptor interaction scoring between sender and receiver clusters.

## When NOT to use

- The input lacks cluster or cell-type annotations — NicheNet requires cell type-annotated expression data reflecting the cell-cell communication event of interest
- The dataset is from an organism other than human or mouse — SCALA currently supports only human and mouse datasets
- You need differential expression per condition rather than per-cluster overexpression for cross-cluster communication — use standard marker identification instead

## Inputs

- Processed scRNA-seq expression matrix (e.g., PBMC3k dataset) with cell-level cluster annotations (Seurat Louvain clusters identified via graph-based clustering in PCA space)
- User-selected sender (ligand-expressing) and receiver (receptor-expressing) cluster identifiers
- Cell-type annotation labels (required by nichenetr: cell type-annotated expression data from human or mouse)

## Outputs

- Per-cluster lists of overexpressed genes (ligand candidates for sender, receptor candidates for receiver)
- Filtered ligand-receptor candidate pairs restricted to genes with >10% expression in clusters of interest
- lr_interactions_scores.csv table with 'prior interaction potential' score per interaction

## How to apply

Start from a processed scRNA-seq dataset (e.g., PBMC3k) with cluster annotations obtained via Seurat graph-based clustering in PCA space. For the user-selected sender cluster (e.g., cluster 0, ligand-expressing) and receiver cluster (e.g., cluster 2, receptor-expressing), compute overexpressed genes in each cluster as required by the nichenetr framework — typically via marker identification against all other clusters. Apply SCALA's expression-coverage filter: retain only ligands/receptors with a percentage of expression > 10% in the clusters of interest. These filtered overexpressed gene lists then define the candidate ligand and receptor pools for constructing the protein-protein interaction prior model and ranking ligand-receptor interactions. Rationale: NicheNet combines expression data with a prior model of ligand-to-target signaling, so restricting to genes detectably expressed in the relevant clusters removes spurious interactions and yields interpretable shared vs. specific interaction sets (e.g., 157 sublining-lining and 152 intermediate-lining interactions in the hTNFtg use case).

## Related tools

- **SCALA** (Interactive pipeline that computes overexpressed genes per annotated cluster and applies the >10% expression filter before NicheNet-based ligand-receptor scoring)
- **nichenetr** (R implementation of the NicheNet method; requires overexpressed genes per sender/receiver cluster as input to its prior-model-based ligand activity analysis) — https://github.com/saeyslab/nichenetr
- **Seurat** (Provides the graph-based clustering in PCA space that produces the Louvain cluster annotations over which overexpressed genes are computed)
- **presto** (Fast differential expression / marker discovery used to identify cluster markers in single-cell data) — https://github.com/immunogenomics/presto

## Examples

```
devtools::install_github("saeyslab/nichenetr"); nichenet_seuratobj_aggregate(seurat_obj, receiver = "cluster2", sender = "cluster0", condition_colname = NULL, expressed_pct = 0.10)
```

## Evaluation signals

- lr_interactions_scores.csv exists and is valid CSV
- Every reported interaction carries a 'prior interaction potential' score field
- All prior interaction potential scores are non-negative and sorted so top-ranked entries correspond to the strongest predicted interactions
- Only ligands/receptors with percentage of expression > 10% in the clusters of interest appear in the candidate pools
- Bona fide interactions rendered in a heatmap reproduce SCALA's reported cluster 0–2 interaction pattern (analogous to Fig. 2F)

## Limitations

- SCALA currently supports only human and mouse datasets for both scRNA-seq and scATAC-seq modalities
- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session, and cannot complete some steps for datasets larger than 50,000 cells
- NicheNet's predictions depend on the prior model integrating ligand-receptor interactions, signaling pathways, and TF regulation; results reflect prior knowledge coverage

## Evidence

- [results] SCALA's ligand-receptor analysis layer employs the nichenetR package, taking annotated cell clusters as input and focusing on ligands/receptors with a percentage of expression > 10% in the clusters of interest: "focusing on ligands/receptors with a percentage of expression > 10% in the clusters of interest"
- [results] compute overexpressed genes in each cluster as required by the nichenetR framework: [evidence span withheld — anchor status: not_found]
- [readme] As input to NicheNet, users must provide cell type-annotated expression data that reflects a cell-cell communication (CCC) event. The input can be single-cell or sorted bulk data from human or mouse.: [evidence span withheld — anchor status: external_doc]
- [methods] It is worth noting that in both modalities, the analysis of only human and mouse datasets is currently supported.: "It is worth noting that in both modalities, the analysis of only human and mouse datasets is currently supported."
- [results] Load the processed PBMC3k scRNA-seq dataset with its Louvain cluster annotations (identified via Seurat graph-based clustering in PCA space): [evidence span withheld — anchor status: not_found]
