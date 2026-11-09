---
name: go-ontology-enrichment-interpretation
description: Use when you have paired transcriptome-only and hybrid ClonoCluster cluster
  assignments plus a count matrix, and need genes predicting cluster rearrangement
  and their GO CC/MF enrichment.
when_to_use_negative:
- You only have transcriptome-only cluster markers without paired hybrid cluster assignments
  — reorg-AUC requires comparing transcriptome-only vs hybrid cluster membership for
  the same cells.
- The marker list is empty or too small for a hypergeometric overrepresentation test
  to be meaningful after the reorg-AUC > 0.80 filter.
- The goal is marker fidelity assessment of hybrid clusters themselves (use Find_Markers_ROC
  per-cluster AUC analysis, not rearrangement ORA).
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Produces the hybrid α cluster assignments whose rearrangements relative to
    transcriptome-only clusters define reorganization markers
  repo: https://github.com/leeprichman/ClonoCluster
- name: ROCR
  role: ROC/AUC calculation backend for quantifying how well each gene's expression
    predicts cluster rearrangement via Find_Markers_ROC
  repo: https://github.com/leeprichman/ClonoCluster
- name: WebGestaltR
  role: Performs GO CC/MF overrepresentation analysis (hypergeometric test, BH FDR
    correction) on the reorganization marker list
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
- name: data.table
  role: R package dependency for fast handling of count matrices, AUC tables, and
    enrichment result tables
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:849aaaa0af623b09d2f01784f77410fbd53c39348cb57ca5ecbcb09cbfa19fcf
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# go-ontology-enrichment-interpretation

## Summary

Identify reorganization markers (genes whose expression predicts cell rearrangement between transcriptome-only and hybrid α clusters, reorg-AUC > 0.80) and run GO Cellular Compartment/Molecular Function overrepresentation analysis with WebGestaltR to reveal shared biology (translation/ribosome and extracellular matrix gene sets).

## When to use

Use when you have paired transcriptome-only and hybrid ClonoCluster cluster assignments plus a count matrix, and need genes predicting cluster rearrangement and their GO CC/MF enrichment.

## When NOT to use

- You only have transcriptome-only cluster markers without paired hybrid cluster assignments — reorg-AUC requires comparing transcriptome-only vs hybrid cluster membership for the same cells.
- The marker list is empty or too small for a hypergeometric overrepresentation test to be meaningful after the reorg-AUC > 0.80 filter.
- The goal is marker fidelity assessment of hybrid clusters themselves (use Find_Markers_ROC per-cluster AUC analysis, not rearrangement ORA).

## Inputs

- normalized/scaled gene-by-cell count matrix (e.g. from Data_genes/ in the ClonoCluster_paper release)
- transcriptome-only cluster assignments per cell
- ClonoCluster hybrid cluster assignments per cell at the chosen α value
- reorganization marker gene list (reorg-AUC > 0.80)

## Outputs

- reorg_markers.csv — list of reorganization markers with reorg-AUC > 0.80
- enrichment_table.csv — significant GO CC/MF terms (BH FDR < 0.05) from WebGestaltR ORA
- per-dataset WebGestaltR output files (e.g. *_rearrangment_ORA.txt)

## How to apply

Load the gene-by-cell count matrix and both cluster assignment tables (transcriptome-only and ClonoCluster hybrid clusters) from the upstream ClonoCluster run. For each gene, compute the reorg-AUC — how well expression predicts the cluster rearrangement between paired transcriptome and hybrid clusters — using the Find_Markers_ROC wrapper around ROCR for ROC/AUC calculation. Filter genes to those with reorg-AUC > 0.80 to define reorganization markers. Run WebGestaltR overrepresentation analysis (hypergeometric test) on the marker list against GO Cellular Compartment and Molecular Function ontologies with Benjamini-Hochberg FDR correction at FDR < 0.05. Interpret the top terms: across datasets in this paper the most widely shared enriched gene sets were translation-related (polysome, rRNA binding, structural constituent of ribosome) and extracellular matrix categories.

## Related tools

- **ClonoCluster** (Produces the hybrid α cluster assignments whose rearrangements relative to transcriptome-only clusters define reorganization markers) — https://github.com/leeprichman/ClonoCluster
- **ROCR** (ROC/AUC calculation backend for quantifying how well each gene's expression predicts cluster rearrangement via Find_Markers_ROC) — https://github.com/leeprichman/ClonoCluster
- **WebGestaltR** (Performs GO CC/MF overrepresentation analysis (hypergeometric test, BH FDR correction) on the reorganization marker list) — https://github.com/arjunrajlaboratory/ClonoCluster_paper
- **data.table** (R package dependency for fast handling of count matrices, AUC tables, and enrichment result tables) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
Rscript Paper/extractionScripts/Marker_analysis.R  # identifies cluster markers, reorganization markers (reorg-AUC > 0.80 via Find_Markers_ROC/ROCR), and runs WebGestaltR GO CC/MF ORA
```

## Evaluation signals

- enrichment_table.csv contains significant GO CC/MF terms at Benjamini-Hochberg FDR < 0.05
- Ribosomal/translation gene sets (polysome, rRNA binding, structural constituent of ribosome) and extracellular matrix gene sets appear among the top enriched categories, consistent with the paper's reported rearrangement-associated gene classes
- The reorganization marker list is reproducibly filtered at reorg-AUC > 0.80 from the Find_Markers_ROC/ROCR output (e.g. *_auc_barcodes.txt)
- ORA output schema matches WebGestaltR hypergeometric overrepresentation results (overlapping genes, expected counts, FDR columns)

## Limitations

- Further testing of the homogeneity of functional properties of cells within each cluster is required to determine which clustering methods most closely match biological distinctions, so enriched gene classes are suggestive rather than definitive.
- The paper did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate, so reorganization-marker enrichment may not generalize to all systems.
- Raw data must be obtained from the ClonoCluster_paper GitHub release (pixz-compressed tar) since it cannot be hosted directly on GitHub.

## Evidence

- [methods] Cluster marker analysis with `Find_Markers_ROC`: "Cluster marker analysis with `Find_Markers_ROC`"
- [readme] In addition to installing ClonoCluster, this workflow requires the following R packages from CRAN: [evidence span withheld — anchor status: external_doc]
- [abstract] The genes most strongly associated with the rearrangements in hybrid clusters were ribosomal function and extracellular matrix genes.: "The genes most strongly associated with the rearrangements in hybrid clusters were ribosomal function and extracellular matrix genes."
- [readme] *_auc_barcodes.txt - AUC to identify reorganization markers between a paired transcriptome cluster and a hybrid cluster: [evidence span withheld — anchor status: external_doc]
- [intro] These rearrangements had novel, potentially more biologically interpretable, cluster markers: "These rearrangements had novel, potentially more biologically interpretable, cluster markers, and were associated with expression of genes involved in extracellular matrix production and translation."
