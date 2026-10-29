---
name: co-dependency-index-differential-expression
description: Use when scRNA-seq cluster markers must be cluster-specific (binary)
  DEGs — e.g. to detect genes exclusive to one sub_cluster population — or when comparing
  CDI against standard DE methods like Wilcoxon on DEG counts and specificity.
when_to_use_negative:
- The analysis goal is detecting graded/continuous DEGs across populations rather
  than binary, cluster-exclusive (bDEG) markers — CDI is designed specifically for
  cluster-specific binary DEGs
- The dataset contains multiple biological replicates whose variation must be modeled
  — the CDI approach does not account for variation in biological replicates
- The full feature space must be scanned without preprocessing — CDI is computationally
  intensive, so unfiltered feature spaces (without getExpressedGenes() restriction)
  are impractical
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: Provides getExpressedGenes() for feature-space restriction and findCDIMarkers()
    for CDI/nCDI marker detection per gene-cluster pair
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework underlying the Seurat object input and DotPlot rendering of top
    CDI vs Wilcoxon markers
- name: ggbeeswarm
  role: Overlaid on sensitivity/specificity boxplots when benchmarking CDI markers
    against the Wilcoxon per-marker indices
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:4e5854f4a67b494651a4f0e43d2d748765853bda1ddce87a7141cfb20bcc1555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - getDEG
  - getExpressedGenes
  - ggbeeswarm
  - scMiko
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# co-dependency-index-differential-expression

## Summary

Applies the co-dependency index (CDI), implemented in scMiko's findCDIMarkers(), as a differential expression method for identifying binary differentially-expressed genes (bDEGs) — genes exclusively expressed in one cell population — within scRNA-seq data, scoring gene-cluster pairs by binomial co-occurrence enrichment.

## When to use

Use when scRNA-seq cluster markers must be cluster-specific (binary) DEGs — e.g. to detect genes exclusive to one sub_cluster population — or when comparing CDI against standard DE methods like Wilcoxon on DEG counts and specificity.

## When NOT to use

- The analysis goal is detecting graded/continuous DEGs across populations rather than binary, cluster-exclusive (bDEG) markers — CDI is designed specifically for cluster-specific binary DEGs
- The dataset contains multiple biological replicates whose variation must be modeled — the CDI approach does not account for variation in biological replicates
- The full feature space must be scanned without preprocessing — CDI is computationally intensive, so unfiltered feature spaces (without getExpressedGenes() restriction) are impractical

## Inputs

- Preprocessed Seurat object with author/cluster annotations (e.g. so_tyser2021_220621.rds, 1195 cells, sub_cluster metadata column)
- List of genes expressed within the dataset (from getExpressedGenes()) used to subset the feature space
- Cluster/identity metadata column (features.x, e.g. sub_cluster)

## Outputs

- Per gene-cluster pair normalized CDI (nCDI) scores, all values within [0,1]
- Per-cluster ranked top-marker table (e.g. cdi_markers.csv, top-ranked by nCDI)
- Significant DEG counts per cluster at FDR < 0.05 and Seurat DotPlots of top-50 markers
- Benchmarking comparison vs Wilcoxon (DEG counts, sensitivity/specificity boxplots with ggbeeswarm overlays)

## How to apply

Load the preprocessed Seurat object (e.g. the Tyser 2021 human gastrulation dataset, 1195 cells) and first restrict the feature space to genes expressed within the dataset using scMiko's getExpressedGenes(), because CDI is computationally intensive on the full feature space. Run findCDIMarkers(features.x = 'sub_cluster', features.y = expressed genes) to obtain normalized CDI (nCDI) scores per gene-cluster pair, where CDI derives from the binomial co-occurrence p-value of expressing cells within the cluster and nCDI is normalized to [0,1] per Methods Eq. 1-4. Rank genes within each cluster by nCDI, select top markers (e.g. top-50 by nCDI), and call significant DEGs at FDR < 0.05. For benchmarking, run Wilcoxon via getDEG(group_by = 'sub_cluster', return.all = TRUE), select top-50 by AUC, render Seurat DotPlots of both marker sets, and compare DEG counts and cluster specificity — the expected pattern is that CDI identifies fewer DEGs than Wilcoxon but with superior cluster specificity.

## Related tools

- **scMiko** (Provides getExpressedGenes() for feature-space restriction and findCDIMarkers() for CDI/nCDI marker detection per gene-cluster pair) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework underlying the Seurat object input and DotPlot rendering of top CDI vs Wilcoxon markers)
- **ggbeeswarm** (Overlaid on sensitivity/specificity boxplots when benchmarking CDI markers against the Wilcoxon per-marker indices)

## Examples

```
expr_gene <- getExpressedGenes(object = so.query); cdi_markers <- findCDIMarkers(object = so.query, features.x = "sub_cluster", features.y = expr_gene)
```

## Evaluation signals

- All nCDI values fall within [0,1] after normalization
- At 5% FDR, CDI identifies fewer significant DEGs than Wilcoxon across the tested datasets
- Top-50 CDI DEGs show higher cluster specificity than top-50 Wilcoxon DEGs (expected 0.95 vs 0.75, p = 5.4e-3, Fig. 2g,i pattern)
- Per-cluster top nCDI markers reproduce the cluster-specific expression pattern of the scMiko findCDIMarkers output / CDI Markers dot plot (Fig. 2j pattern)
- CDI markers match gene-cluster pairs to the Wilcoxon sensitivity/specificity table returned by getDEG(return.all = TRUE) without unmatched entries

## Limitations

- The CDI DE method does not account for variation in biological replicates, a known limitation it shares with existing scRNA-seq DE methods
- CDI is computationally intensive on the full feature space, requiring prior restriction to expressed genes via getExpressedGenes()
- Because CDI targets binary DEGs, it is inappropriate when cluster stability is uncertain — in single-replicate datasets it is difficult to determine whether clusters represent technical artifacts or real biology

## Evidence

- [intro] CDI as a DE method identifying bDEGs: "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [abstract] CDI in the scPipeline module list: "(i) co-dependency index (CDI)-based differential expression"
- [methods] Feature-space restriction rationale: "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [discussion] Biological replicate limitation: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception15."
- [readme] Package installation: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
