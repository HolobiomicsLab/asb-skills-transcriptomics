---
name: t-cell-naive-effector-subtype-annotation
description: Use when annotating clusters of a Seurat-processed PBMC 3k (or similar
  immune) scRNA-seq object where closely related T-cell subtypes (naïve, memory, effector)
  score nearly equally under positive-marker-only annotation and negative markers
  are needed to resolve them.
when_to_use_negative:
- When the target cell types have no curated negative markers in the ScType database
  — negative-marker evidence is the key discriminator here
- When the input is already an annotated cell-type label table rather than a raw/scaled
  scRNA-seq expression matrix
- When a supervised reference-based classifier is acceptable and a labelled training
  dataset exists (ScType is unsupervised and reference-free)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ScType
  role: Computes per-cluster cell-type scores from positive (gs) and negative (gs2)
    marker gene sets; the negative-marker run is what resolves naïve vs effector T
    cells
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Loads Read10X data, normalizes (LogNormalize, 10000), selects 2000 vst variable
    features, scales data, runs PCA and FindNeighbors/FindClusters (dims 1:10, resolution
    0.8) to produce clusters for scoring
- name: dplyr
  role: Aggregates es.max scores per cluster by row sums and selects the top-scoring
    cell type per cluster with top_n(n = 1, wt = scores)
- name: R
  role: Open-source language in which the ScType pipeline is implemented and executed
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:67d559fb556445850beebdb4d29206d2eff393d84bafdac6e00fc2983d30ac7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - dplyr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# t-cell-naive-effector-subtype-annotation

## Summary

Annotates T-cell subtypes (naïve vs effector/memory) in PBMC scRNA-seq clusters with ScType by supplying both positive and negative marker gene sets, using negative markers such as CCR7 and SELL for effector T cells to suppress false effector assignments in CCR7/SELL-expressing clusters.

## When to use

Use when annotating clusters of a Seurat-processed PBMC 3k (or similar immune) scRNA-seq object where closely related T-cell subtypes (naïve, memory, effector) score nearly equally under positive-marker-only annotation and negative markers are needed to resolve them.

## When NOT to use

- When the target cell types have no curated negative markers in the ScType database — negative-marker evidence is the key discriminator here
- When the input is already an annotated cell-type label table rather than a raw/scaled scRNA-seq expression matrix
- When a supervised reference-based classifier is acceptable and a labelled training dataset exists (ScType is unsupervised and reference-free)

## Inputs

- PBMC 3k filtered gene-barcode matrices from the 10x Genomics Dataset Repository (Read10X input)
- Seurat object with scaled expression matrix (LogNormalize, scale.factor = 10000; PCA dims 1:10; resolution 0.8)
- ScTypeDB_full.xlsx marker database restricted to the 'Immune system' tissue type, with positive (gs) and negative (gs2) marker sets including CCR7 and SELL as negative markers for effector T cells

## Outputs

- Per-cluster T-cell subtype labels (naïve, memory, effector T cells, or 'Unknown' for scores < ncells/4)
- Per-cluster ScType scores under both positive+negative and positive-only scoring modes
- CSV table of per-cluster scores and labels for the two scoring runs

## How to apply

Load the PBMC 3k matrix from the 10x Genomics repository into Seurat, normalize with LogNormalize (scale.factor = 10000), select 2000 variable features (vst), scale all genes, run PCA, and cluster with FindNeighbors/FindClusters (dims = 1:10, resolution = 0.8). Prepare positive and negative gene sets with gene_sets_prepare from ScTypeDB_full.xlsx restricted to the 'Immune system' tissue, then extract the scaled expression matrix and run sctype_score with both gs (positive) and gs2 (negative) sets, explicitly including CCR7 and SELL as negative markers for effector T-cell types. Aggregate es.max scores per cluster by row sums, pick the top cell type via top_n(n = 1, wt = scores), and label clusters scoring below ncells/4 as 'Unknown'. Rerun sctype_score with gs2 = NULL (positive markers only) and compare: negative markers should suppress the effector assignment for CCR7/SELL-expressing clusters, whereas positive-only scoring misassigns naïve T cells as effector T cells with almost equal scores (4 vs 7). Write per-cluster scores and labels to a CSV table for verification.

## Related tools

- **ScType** (Computes per-cluster cell-type scores from positive (gs) and negative (gs2) marker gene sets; the negative-marker run is what resolves naïve vs effector T cells) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Loads Read10X data, normalizes (LogNormalize, 10000), selects 2000 vst variable features, scales data, runs PCA and FindNeighbors/FindClusters (dims 1:10, resolution 0.8) to produce clusters for scoring)
- **dplyr** (Aggregates es.max scores per cluster by row sums and selects the top-scoring cell type per cluster with top_n(n = 1, wt = scores))
- **R** (Open-source language in which the ScType pipeline is implemented and executed) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- With gs2 including CCR7/SELL as negative markers, CCR7/SELL-expressing clusters are labeled naïve/memory T cells, not effector T cells
- With gs2 = NULL (positive markers only), naïve T-cell clusters are misassigned as effector T cells with almost equal ScType scores (4 for naïve vs 7 for effector)
- Clusters whose top ScType score falls below ncells/4 are set to 'Unknown'
- Per-cluster scores and labels from both runs are present in the exported CSV table and differ between gs2-present and gs2-NULL runs for the T-cell clusters

## Limitations

- Negative markers are still relatively few compared to positive markers, limiting annotation; results are expected to improve as more negative markers are curated
- ScType uses z-score transformation, which may be suboptimal for single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version, which can affect negative-marker evidence

## Evidence

- [intro] Negative markers suppress effector assignment: "the expression of negative marker genes, which provide evidence against a cell being of a particular type, should also be incorporated into the cell-type identification process"
- [methods] Marker preparation functions: "source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [readme] Positive+negative scoring call: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [readme] Immune-system gene set preparation: "gs_list <- gene_sets_prepare("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/ScTypeDB_short.xlsx", "Immune system")"
