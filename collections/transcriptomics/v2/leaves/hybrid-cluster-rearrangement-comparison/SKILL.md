---
name: hybrid-cluster-rearrangement-comparison
description: Use when you already have paired transcriptome-only and ClonoCluster
  hybrid cluster assignments plus a count matrix, and need to find genes driving cluster
  rearrangements and test them for GO enrichment.
when_to_use_negative:
- You only need standard hybrid-cluster marker identification for a single clustering
  level (use Find_Markers_ROC cluster-marker analysis directly, not the rearrangement
  comparison)
- Your cells lack clone barcode information, so no hybrid ClonoCluster assignments
  exist to compare against transcriptome-only clusters
- Alpha tuning, cluster sizing, or Warp Factor visualization is the goal — those are
  upstream/downstream steps, not rearrangement-marker analysis
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Provides the hybrid cluster assignments (via tunable alpha weighting of transcriptome
    and clone edge weights) that are compared against transcriptome-only clusters
    to find rearrangements
  repo: https://github.com/leeprichman/ClonoCluster
- name: ROCR
  role: ROC/AUC engine behind the Find_Markers_ROC wrapper used to compute reorg-AUC
    for each gene's prediction of cluster rearrangement
- name: WebGestaltR
  role: Runs hypergeometric GO CC/MF overrepresentation analysis on the filtered reorganization
    marker list with BH FDR correction
- name: data.table
  role: R package dependency of the ClonoCluster_paper analysis scripts (Marker_analysis.R)
    that process count matrices and marker tables
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:206e7d5356be734181ffa390bf240eb5cfb68414052a32f597c21f002c17101e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - ROCR
  - WebGestaltR
  - data.table
  - parallel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hybrid-cluster-rearrangement-comparison

## Summary

Identifies reorganization markers — genes whose expression predicts when a cell switches affiliation from a transcriptome-only cluster to a particular hybrid ClonoCluster cluster — by computing reorg-AUC values with Find_Markers_ROC/ROCR, filtering at reorg-AUC > 0.80, and running WebGestaltR GO overrepresentation analysis on the marker list.

## When to use

Use when you already have paired transcriptome-only and ClonoCluster hybrid cluster assignments plus a count matrix, and need to find genes driving cluster rearrangements and test them for GO enrichment.

## When NOT to use

- You only need standard hybrid-cluster marker identification for a single clustering level (use Find_Markers_ROC cluster-marker analysis directly, not the rearrangement comparison)
- Your cells lack clone barcode information, so no hybrid ClonoCluster assignments exist to compare against transcriptome-only clusters
- Alpha tuning, cluster sizing, or Warp Factor visualization is the goal — those are upstream/downstream steps, not rearrangement-marker analysis

## Inputs

- Normalized or scaled gene-by-cell count matrix (e.g. from Data_genes/, retrievable from a Seurat object)
- Hybrid cluster assignments for a range of alpha values (*_cluster_assignments.txt from Long_alluvia.R)
- Transcriptome-only cluster assignments for pairing against hybrid clusters
- Table of unique cell IDs and assigned barcodes (from the upstream ClonoCluster run)

## Outputs

- reorg_markers.csv — list of reorganization markers passing reorg-AUC > 0.80 (paper equivalent: *_auc_barcodes.txt)
- enrichment_table.csv — WebGestaltR GO CC/MF overrepresentation results (paper equivalent: *_rearrangment_ORA.txt)
- Optional heatmap of enrichment results across datasets (geneset_hm.R, Figure 3 reorg_hm)

## How to apply

Load the normalized/scaled count matrix and the hybrid cluster assignments (transcriptome-only and ClonoCluster hybrid clusters generated at a chosen alpha) from the upstream ClonoCluster run. For each gene, compute the reorg-AUC quantifying how well expression predicts cluster rearrangement between each paired transcriptome cluster and hybrid cluster, using the Find_Markers_ROC wrapper around ROCR for ROC/AUC calculation. Filter genes to reorganization markers at reorg-AUC > 0.80 and write the marker list (e.g. reorg_markers.csv, corresponding to *_auc_barcodes.txt in the paper repo). Run WebGestaltR overrepresentation analysis (hypergeometric test) on the marker list against GO Cellular Compartment (CC) and Molecular Function (MF) ontologies with Benjamini-Hochberg FDR correction at FDR < 0.05. Rationale: genes most strongly associated with rearrangements in hybrid clusters are expected to be translation/ribosomal and extracellular matrix genes, which this enrichment step confirms.

## Related tools

- **ClonoCluster** (Provides the hybrid cluster assignments (via tunable alpha weighting of transcriptome and clone edge weights) that are compared against transcriptome-only clusters to find rearrangements) — https://github.com/leeprichman/ClonoCluster
- **ROCR** (ROC/AUC engine behind the Find_Markers_ROC wrapper used to compute reorg-AUC for each gene's prediction of cluster rearrangement)
- **WebGestaltR** (Runs hypergeometric GO CC/MF overrepresentation analysis on the filtered reorganization marker list with BH FDR correction)
- **data.table** (R package dependency of the ClonoCluster_paper analysis scripts (Marker_analysis.R) that process count matrices and marker tables) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
setwd("ClonoCluster_paper/"); source("Paper/extractionScripts/Marker_analysis.R")
```

## Evaluation signals

- All retained reorganization markers have reorg-AUC > 0.80
- reorg_markers.csv is non-empty and gene IDs match the count matrix row names
- enrichment_table.csv contains significant GO CC/MF terms at BH FDR < 0.05
- Ribosomal/translation terms (polysome, rRNA binding, structural constituent of ribosome) and extracellular matrix gene sets appear among the top enriched categories, consistent with the paper's findings
- Per-sample outputs follow the paper's naming scheme: *_auc_barcodes.txt and *_rearrangment_ORA.txt regenerated by Marker_analysis.R

## Limitations

- Further testing of the homogeneity of functional properties of cells within each cluster is required to determine which clustering methods most closely match biological distinctions
- Findings of rearrangement-associated gene classes were observed across the paper's six datasets but a reliable method to identify which experimental systems benefit from clonal information was not established
- Raw data must be obtained from the GitHub release (BarCluster_raw_data.tpxz, decompressed with pixz) since it is not hosted directly in the repository

## Evidence

- [abstract] Genes most strongly associated with rearrangements are ribosomal/ECM: "The genes most strongly associated with the rearrangements in hybrid clusters were ribosomal function and extracellular matrix genes."
- [methods] Find_Markers_ROC used for cluster marker analysis: "* Cluster marker analysis with `Find_Markers_ROC`"
- [readme] README defines reorganization-marker AUC output file: [evidence span withheld — anchor status: external_doc]
- [readme] README identifies Marker_analysis.R as the script for reorganization markers and overrepresentation analysis: [evidence span withheld — anchor status: external_doc]
- [intro] Rearrangements had biologically interpretable markers tied to ECM and translation: "These rearrangements had novel, potentially more biologically interpretable, cluster markers, and were associated with expression of genes involved in extracellular matrix production and translation."
