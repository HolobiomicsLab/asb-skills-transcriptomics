---
name: cell-cell-communication-network-analysis
description: Use when you have clustered scRNA-seq cell groups and need to infer significant
  ligand-receptor communication between them, quantify pathway-level signaling, identify
  dominant senders/receivers, and group coordinated outgoing/incoming patterns.
when_to_use_negative:
- Input scRNA-seq data has not yet been clustered into cell groups — cell clustering
  is a pre-requisite for communication analysis with CellChat
- Spatially resolved data in CellChat v1/v2 where spatial constraints have not been
  introduced or tested — use SpatialCellChat (v3) instead
- Systems requiring ground-truth-validated interaction calls — systematic evaluation
  of predicted networks is challenged by lack of ground truth
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: 'Primary R package: infers communication probabilities, computes centrality
    metrics, performs NMF pattern recognition and pathway similarity clustering'
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated ligand-receptor-cofactor database (2,021 validated interactions
    from KEGG) consulted via identifyOverExpressedInteractions
- name: Seurat
  role: 'Preprocessing: QC, subclustering of cell states (Louvain) and annotation
    before CellChat object creation'
- name: UMAP
  role: Embeds the smoothed signaling-pathway similarity matrix via netEmbedding to
    obtain the pathway manifold
  repo: https://github.com/lmcinnes/umap
- name: circlize
  role: Chord-diagram visualization of predicted signaling networks (netVisual_aggregate)
  repo: https://github.com/jokergoo/circlize
- name: ComplexHeatmap
  role: Heatmap visualization of communication networks
  repo: https://github.com/jokergoo/ComplexHeatmap
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:952355651a792045d08adf746b378ecfa6b9a66f8fc68bd0d62c9fb269c75946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - ComplexHeatmap
  - Seurat
  - UMAP
  - circlize
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cell-cell-communication-network-analysis

## Summary

Infers, quantifies, and analyzes intercellular ligand-receptor communication networks from single-cell RNA-seq data using CellChat, which models communication probability via law-of-mass-action/Hill dynamics against the manually curated CellChatDB, then assigns sender/receiver/mediator/influencer roles and patterns via network centrality, NMF, and manifold learning.

## When to use

Use when you have clustered scRNA-seq cell groups and need to infer significant ligand-receptor communication between them, quantify pathway-level signaling, identify dominant senders/receivers, and group coordinated outgoing/incoming patterns.

## When NOT to use

- Input scRNA-seq data has not yet been clustered into cell groups — cell clustering is a pre-requisite for communication analysis with CellChat
- Spatially resolved data in CellChat v1/v2 where spatial constraints have not been introduced or tested — use SpatialCellChat (v3) instead
- Systems requiring ground-truth-validated interaction calls — systematic evaluation of predicted networks is challenged by lack of ground truth

## Inputs

- Clustered scRNA-seq count matrix (Seurat object or dense/sparse matrix) with cell-group metadata
- CellChatDB ligand-receptor/cofactor interaction database (mouse or human)
- Pre-inferred CellChat communication probability network (output of computeCommunProb) for pattern and similarity analysis
- Optionally spatial coordinates/factors for spatially proximal communication (CellChat v3/SpatialCellChat)

## Outputs

- Significant ligand-receptor pair list with communication probabilities at p<0.05
- Pathway-level aggregated communication network (computeCommunProbPathway)
- Cell-group centrality scores (out-degree, in-degree, flow betweenness, information centrality) classifying sender/receiver/mediator/influencer roles
- W/H NMF pattern matrices and k-means signaling-pathway group assignments with UMAP embedding coordinates (e.g. pattern_analysis_results.rds)
- Circle/heatmap/alluvial/Chord visualizations (netVisual_aggregate via circlize, ComplexHeatmap)

## How to apply

Create a CellChat object from the Seurat count matrix and metadata with createCellChat, run identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05) and identifyOverExpressedInteractions against CellChatDB (2,021 validated interactions including heteromeric complexes and cofactors), then compute communication probabilities with computeCommunProb using the law-of-mass-action model with ensemble-averaged expression as the trimean of quartiles and a permutation test (M=100 label shuffles, p<0.05). Aggregate to pathway level with computeCommunProbPathway, filter with filterCommunication, and compute centrality metrics (out-degree, in-degree, flow betweenness, information centrality) with netAnalysis_computeCentrality to classify dominant senders, receivers, mediators, and influencers. To identify communication patterns, apply non-negative matrix factorization via identifyCommunicationPatterns separately for outgoing and incoming signaling, selecting pattern numbers with computeEigengap on the Laplacian of the consensus matrix pruned by zeroing elements < 0.3, and zeroing W/H elements < 0.5 for pattern-contribution summaries. For pathway-level grouping, compute functional similarity (network Jaccard overlap via computeNetSimilarity) and structural similarity, build and smooth an SNN similarity network, assign pathway groups with netClustering (k-means), and embed with netEmbedding (UMAP).

## Related tools

- **CellChat** (Primary R package: infers communication probabilities, computes centrality metrics, performs NMF pattern recognition and pathway similarity clustering) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated ligand-receptor-cofactor database (2,021 validated interactions from KEGG) consulted via identifyOverExpressedInteractions)
- **Seurat** (Preprocessing: QC, subclustering of cell states (Louvain) and annotation before CellChat object creation)
- **UMAP** (Embeds the smoothed signaling-pathway similarity matrix via netEmbedding to obtain the pathway manifold) — https://github.com/lmcinnes/umap
- **circlize** (Chord-diagram visualization of predicted signaling networks (netVisual_aggregate)) — https://github.com/jokergoo/circlize
- **ComplexHeatmap** (Heatmap visualization of communication networks) — https://github.com/jokergoo/ComplexHeatmap

## Examples

```
devtools::install_github("jinworks/CellChat"); cellchat <- createCellChat(object = data, meta = meta, group.by = "labels"); cellchat <- netAnalysis_computeCentrality(cellchat)
```

## Evaluation signals

- Significant interactions pass the permutation test with M=100 label shuffles at p<0.05
- Centrality-derived hub roles match expected biology (e.g. placode cells secrete via 44 and receive via 19 significant ligand-receptor pairs in E14.5 embryonic skin)
- Pathway-level aggregation recovers expected pathway families (e.g. WNT, TGFbeta, SEMA3, PTN, PDGF, MK, FGF placode-to-DC pairs)
- Functional similarity grouping reproduces the published number of pathway groups (four groups in the wound skin dataset)
- Recovered outgoing/incoming pattern counts match published pattern numbers after consensus-matrix pruning (<0.3 zeroed) and W/H thresholding (<0.5 zeroed)

## Limitations

- Communication probability represents interaction strength only and is not exactly a probability
- mRNA levels approximate protein levels, so comprehensive ligand-receptor modeling is challenged by lack of single-cell proteomics
- Hill function parameters Kh and n lack explicit biological connections and determining biologically meaningful mass-action parameters is challenging
- Systematic evaluation of predicted networks is difficult due to lack of ground truth, and overlap-based comparison assumes similar methods generate similar results regardless of accuracy

## Evidence

- [intro] CellChat infers cell-state specific signaling communications using mass action models: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models, along with differential expression analysis and statistical tests on cell groups,"
- [discussion] Significant interactions via permutation test: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Centrality metrics for sender/receiver roles: "we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and"
- [methods] createCellChat signature: "createCellChat(object, meta, group.by, datatype, coordinates, spatial.factors, assay, do.sparse)"
- [discussion] NMF pattern discovery: "CellChat uses an unsupervised learning method non-negative matrix factorization that has been successfully applied in pattern recognition"
