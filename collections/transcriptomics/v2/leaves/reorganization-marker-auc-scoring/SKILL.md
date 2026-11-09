---
name: reorganization-marker-auc-scoring
description: Use when you have paired transcriptome-only and ClonoCluster hybrid cluster
  assignments plus a count matrix, and need genes whose expression predicts cell reassignment
  (reorganization markers) between cluster pairs for downstream GO enrichment.
when_to_use_negative:
- You only have one clustering scheme with no paired transcriptome-only vs hybrid
  α assignments — use standard cluster marker analysis with Find_Markers_ROC instead
- No clone barcode data exist, so hybrid clusters cannot be generated and rearrangement
  between clusterings is undefined
- The goal is cluster-level marker identification at a single clustering level (covered
  by *_auc_linclust.txt AUCs), not genes predicting switches between clusterings
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Produces the hybrid α cluster assignments that the transcriptome-only clusters
    are rearranged against; installed via devtools::install_github("leeprichman/ClonoCluster")
  repo: https://github.com/leeprichman/ClonoCluster
- name: ROCR
  role: ROC/AUC computation backend behind the Find_Markers_ROC wrapper used to score
    each gene's reorg-AUC
- name: WebGestaltR
  role: Hypergeometric GO CC/MF overrepresentation analysis on the reorg-AUC > 0.80
    marker list
- name: data.table
  role: R package used in Marker_analysis.R to handle the count matrices, cluster
    assignments, and AUC result tables
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:e37f70be9eabd74a7024b092184b136de643bd02b2c35385e695d9600c4c095a
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reorganization-marker-auc-scoring

## Summary

Scores every gene by reorg-AUC — how well its expression predicts whether a cell switches affiliation from a transcriptome-only cluster to a particular ClonoCluster hybrid α cluster — using the Find_Markers_ROC wrapper around ROCR, then retains genes with reorg-AUC > 0.80 as reorganization markers.

## When to use

Use when you have paired transcriptome-only and ClonoCluster hybrid cluster assignments plus a count matrix, and need genes whose expression predicts cell reassignment (reorganization markers) between cluster pairs for downstream GO enrichment.

## When NOT to use

- You only have one clustering scheme with no paired transcriptome-only vs hybrid α assignments — use standard cluster marker analysis with Find_Markers_ROC instead
- No clone barcode data exist, so hybrid clusters cannot be generated and rearrangement between clusterings is undefined
- The goal is cluster-level marker identification at a single clustering level (covered by *_auc_linclust.txt AUCs), not genes predicting switches between clusterings

## Inputs

- Normalized/scaled gene-by-cell count matrix (e.g. retrieved from a Seurat object)
- Transcriptome-only cluster assignments per cell
- ClonoCluster hybrid cluster assignments at a chosen α (*_cluster_assignments.txt)

## Outputs

- Per-gene reorg-AUC table for each paired transcriptome/hybrid cluster combination (*_auc_barcodes.txt)
- reorg_markers.csv — list of reorganization markers with reorg-AUC > 0.80
- enrichment_table.csv — WebGestaltR GO CC/MF overrepresentation results (FDR < 0.05), e.g. *_rearrangment_ORA.txt

## How to apply

Load the gene-by-cell count matrix and both cluster assignment tables (transcriptome-only and ClonoCluster hybrid clusters at a chosen α, e.g. from Long_alluvial.R outputs like *_cluster_assignments.txt). For each paired transcriptome-cluster/hybrid-cluster combination, call the Find_Markers_ROC wrapper (built on ROCR) to compute, per gene, the AUC of expression as a classifier of the cluster rearrangement; these values are what Marker_analysis.R writes to *_auc_barcodes.txt. Threshold genes at reorg-AUC > 0.80 to define reorganization markers, then feed that gene list into WebGestaltR hypergeometric overrepresentation analysis against GO Cellular Compartment and Molecular Function ontologies with Benjamini-Hochberg FDR < 0.05. The rationale is that genes predicting rearrangement between transcriptome and hybrid clusters reveal 'hidden' factors — in the paper these were ribosomal/translation and extracellular-matrix genes — that standard transcriptome-only marker analysis would miss.

## Related tools

- **ClonoCluster** (Produces the hybrid α cluster assignments that the transcriptome-only clusters are rearranged against; installed via devtools::install_github("leeprichman/ClonoCluster")) — https://github.com/leeprichman/ClonoCluster
- **ROCR** (ROC/AUC computation backend behind the Find_Markers_ROC wrapper used to score each gene's reorg-AUC)
- **WebGestaltR** (Hypergeometric GO CC/MF overrepresentation analysis on the reorg-AUC > 0.80 marker list)
- **data.table** (R package used in Marker_analysis.R to handle the count matrices, cluster assignments, and AUC result tables) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
auc <- Find_Markers_ROC(cm, clust, ident = "hybrid_cluster"); reorg_markers <- auc[auc > 0.80]; write.csv(reorg_markers, "reorg_markers.csv")
```

## Evaluation signals

- Every gene in the marker list has reorg-AUC strictly > 0.80, and the list is written to reorg_markers.csv
- enrichment_table.csv contains GO CC/MF terms at FDR < 0.05 with ribosomal/translation (polysome, rRNA binding, structural constituent of ribosome) and extracellular matrix sets among the top categories
- Per-sample reorg-AUC tables (e.g. *_auc_barcodes.txt) exist for each transcriptome/hybrid cluster pair in the dataset
- WebGestaltR output (*_rearrangment_ORA.txt) is populated for each sample analyzed, matching the paper's Figure 3 heatmap inputs

## Limitations

- The paper did not determine which experimental systems have intrinsic determinants of cell fate, so no reliable rule exists for when reorganization markers will be biologically informative
- Further testing of homogeneity of functional properties of cells within clusters is required to confirm that hybrid-cluster rearrangements match biological distinctions
- The approach assumes the ClonoCluster α value has already been validated (via marker fidelity or ground truth); a poorly chosen α yields uninterpretable rearrangements

## Evidence

- [methods] Cluster marker analysis with Find_Markers_ROC: "Cluster marker analysis with `Find_Markers_ROC`"
- [abstract] The genes most strongly associated with the rearrangements in hybrid clusters were ribosomal function and extracellular matrix genes.: "The genes most strongly associated with the rearrangements in hybrid clusters were ribosomal function and extracellular matrix genes."
- [readme] auc_barcodes AUC identifies reorganization markers: [evidence span withheld — anchor status: external_doc]
- [readme] WebGestaltR ORA of reorganization markers: [evidence span withheld — anchor status: external_doc]
