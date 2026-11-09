---
name: single-cell-module-scoring-ssgsea
description: Use when you have a single-cell expression matrix and defined gene signatures
  (meta-modules or cell-cycle sets) and need per-cell module scores plus dominant-module
  calls to annotate clusters from an unsupervised clustering such as RCC.
when_to_use_negative:
- Input is bulk transcriptomic data where subtype discovery (rather than per-cell
  module scoring) is the goal — use RCC directly for bulk subtyping
- No predefined gene signatures are available — ssGSEA requires curated gene sets
  and cannot de novo discover cellular-state modules
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: R package whose ssgsea() function performs per-cell ssGSEA module scoring,
    with clusterAnnotation() and trackingPlot() for cluster-level dominant-module
    aggregation and concordance visualization
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: Runtime environment; RCC is a user-friendly R package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:133720ae0f99fb4a32959e77313c899ea2c9cce9cb4de1de53966f12cec1f245
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - mclust
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-module-scoring-ssgsea

## Summary

Score every single cell against curated gene-set signatures (e.g. the six Neftel GBM meta-modules MES1/MES2/AC/OPC/NPC1/NPC2 plus G1S/G2M cell-cycle signatures) using ssGSEA, then assign each cell its dominant module and aggregate scores per cluster to annotate clusters biologically.

## When to use

Use when you have a single-cell expression matrix and defined gene signatures (meta-modules or cell-cycle sets) and need per-cell module scores plus dominant-module calls to annotate clusters from an unsupervised clustering such as RCC.

## When NOT to use

- Input is bulk transcriptomic data where subtype discovery (rather than per-cell module scoring) is the goal — use RCC directly for bulk subtyping
- No predefined gene signatures are available — ssGSEA requires curated gene sets and cannot de novo discover cellular-state modules

## Inputs

- Single-cell expression matrix formatted as RCC exprsFile (e.g. GSE131928 from GEO)
- Cell metadata / annotation file (infoFile)
- Gene-set definitions: meta-module gene sets MES1, MES2, AC, OPC, NPC1, NPC2 and G1S/G2M cell-cycle signatures
- Cluster assignments per cell (cluster file from RCC_clus output)

## Outputs

- Per-cell ssGSEA scores for each gene set
- Per-cell dominant module assignment (highest ssGSEA score)
- Per-cluster dominant module via clusterAnnotation
- Concordance plot of cell module vs cluster module (Fig 9C style) via trackingPlot

## How to apply

Format the single-cell expression matrix (e.g. the GSE131928 Neftel dataset from GEO) as the exprsFile input alongside an infoFile of cell metadata. Score each cell against the gene sets (MES1, MES2, AC, OPC, NPC1, NPC2 and the G1S/G2M signatures) by running ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir) from the RCC R package. Assign each cell its dominant module as the gene set with the highest ssGSEA score. Aggregate per-cell scores within each cluster and pass cluster assignments to clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir) to determine the dominant module per cluster. Cross-tabulate per-cell module assignments against the dominant module of the cell's cluster and render the concordance plot with trackingPlot(exprsFile, clusterFile, geneFile, attribute, outputDir); the analysis is correct when clusters dominated by MES signatures are enriched for MES-assigned cells, NPC/OPC-dominated clusters for NPC/OPC-assigned cells, and cell-cycle clusters separate G1S from G2M cells.

## Related tools

- **RCC** (R package whose ssgsea() function performs per-cell ssGSEA module scoring, with clusterAnnotation() and trackingPlot() for cluster-level dominant-module aggregation and concordance visualization) — github.com/MSCTR/RecursiveConsensusClustering
- **R** (Runtime environment; RCC is a user-friendly R package)

## Examples

```
ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)
```

## Evaluation signals

- Every cell receives an ssGSEA score for each of the gene sets (MES1, MES2, AC, OPC, NPC1, NPC2, G1S, G2M) with no missing values
- Each cell's dominant module corresponds to the maximum ssGSEA score across the scored gene sets
- Concordance plot shows clusters dominated by MES signatures enriched for MES-assigned cells and NPC/OPC-dominated clusters enriched for NPC/OPC-assigned cells, matching the Fig 9C pattern
- Cell-cycle-dominated clusters cleanly separate G1S from G2M cells
- Per-cluster dominant module from clusterAnnotation is consistent with the aggregate of its constituent cells' dominant modules

## Limitations

- RCC's computational time is a limitation and is dependent on the underlying base algorithm (consensus clustering with k-means), which affects upstream clustering before module scoring
- Analysis of large integrated single-cell datasets with millions of cells is still a next step not yet demonstrated; combine with scanorama and geometric sketching for such data
- Correctness of the concordance pattern depends on the quality/relevance of the supplied gene-set signatures to the dataset

## Evidence

- [methods] ssgsea workflow step: "ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)"
- [methods] cluster annotation step: "clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)"
- [methods] tracking plot step: "trackingPlot(exprsFile, clusterFile, geneFile, attribute, outputDir)"
- [discussion] single-cell support: "Also, it works equally well for both bulk and single-cell data."
- [readme] package installation: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
