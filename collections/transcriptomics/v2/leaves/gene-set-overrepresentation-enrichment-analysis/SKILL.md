---
name: gene-set-overrepresentation-enrichment-analysis
description: Use when you have a filtered gene list (e.g. markers exceeding an AUC
  threshold) and need to test whether defined gene sets (GO CC/MF terms) are overrepresented
  at FDR < 0.05.
when_to_use_negative:
- The input is a full ranked transcriptome or all genes rather than a pre-filtered
  marker list — overrepresentation analysis requires a discrete gene list (e.g. reorg-AUC
  > 0.80 markers)
- The goal is standard cluster-marker identification or comparing marker fidelity
  between transcriptome-only and hybrid clusters, not characterizing which functional
  classes drive cluster rearrangements
- The clustering assignments have not yet been generated (upstream ClonoCluster run
  at a chosen α is required first)
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: WebGestaltR
  role: Performs hypergeometric overrepresentation analysis of reorganization markers
    against GO CC/MF ontologies with Benjamini-Hochberg FDR correction
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
- name: ClonoCluster
  role: Generates the hybrid α clusters whose rearrangement relative to transcriptome-only
    clusters defines the reorganization markers tested for enrichment
  repo: https://github.com/leeprichman/ClonoCluster
- name: ROCR
  role: ROC/AUC calculation (via the Find_Markers_ROC wrapper) used to compute reorg-AUC
    and select markers above 0.80 before enrichment testing
- name: data.table
  role: CRAN dependency used in the Marker_analysis.R script that identifies markers
    and performs the overrepresentation analysis
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:d77877904525831960dc9b68557e54854c4c1942d8a0bccb81697bec665c14e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/arjunrajlaboratory/ClonoCluster_paper
  tools:
  - ClonoCluster
  - ROCR
  - WebGestaltR
  - data.table
  - parallel
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-set-overrepresentation-enrichment-analysis

## Summary

Runs hypergeometric overrepresentation analysis (WebGestaltR) on a gene list of reorganization markers — genes whose expression (reorg-AUC > 0.80) predicts whether a cell switches from a transcriptome-only cluster to a hybrid ClonoCluster α cluster — against GO Cellular Compartment and Molecular Function ontologies, surfacing shared biology (ribosomal/translation and ECM gene sets) across datasets.

## When to use

Use when you have a filtered gene list (e.g. markers exceeding an AUC threshold) and need to test whether defined gene sets (GO CC/MF terms) are overrepresented at FDR < 0.05.

## When NOT to use

- The input is a full ranked transcriptome or all genes rather than a pre-filtered marker list — overrepresentation analysis requires a discrete gene list (e.g. reorg-AUC > 0.80 markers)
- The goal is standard cluster-marker identification or comparing marker fidelity between transcriptome-only and hybrid clusters, not characterizing which functional classes drive cluster rearrangements
- The clustering assignments have not yet been generated (upstream ClonoCluster run at a chosen α is required first)

## Inputs

- Gene-by-cell normalized/scaled count matrix (e.g. Data_genes/ from ClonoCluster_paper raw data)
- Paired cluster assignments: transcriptome-only clusters and ClonoCluster hybrid clusters (*_cluster_assignments.txt)
- Per-gene ROC-derived AUC values for reorganization markers (*_auc_barcodes.txt)
- Marker list filtered at reorg-AUC > 0.80 (reorg_markers.csv)

## Outputs

- reorg_markers.csv — genes passing the reorg-AUC > 0.80 threshold
- enrichment_table.csv / *_rearrangment_ORA.txt — significant GO CC/MF terms with Benjamini-Hochberg FDR < 0.05
- Gene-set enrichment heatmaps (reorg_hm/, from geneset_hm.R)

## How to apply

Load the normalized/scaled gene-by-cell count matrix and the paired cluster assignments (transcriptome-only clusters and hybrid ClonoCluster clusters at a given α). For each gene compute the reorg-AUC quantifying how well expression predicts rearrangement between a transcriptome cluster and a hybrid cluster pair, using the Find_Markers_ROC wrapper around ROCR for ROC/AUC calculation. Filter to reorganization markers at reorg-AUC > 0.80 and write the marker list (reorg_markers.csv). Run WebGestaltR overrepresentation analysis (hypergeometric test) on the marker list against GO Cellular Compartment and Molecular Function ontologies with Benjamini-Hochberg FDR correction at FDR < 0.05, writing the results (enrichment_table.csv / *_rearrangment_ORA.txt). Repeat per dataset (9 replicates from six sources) and compare enriched categories; the paper's validated result is that translation (polysome, rRNA binding, structural constituent of ribosome) and extracellular matrix gene sets are the most widely shared enriched classes.

## Related tools

- **WebGestaltR** (Performs hypergeometric overrepresentation analysis of reorganization markers against GO CC/MF ontologies with Benjamini-Hochberg FDR correction) — https://github.com/arjunrajlaboratory/ClonoCluster_paper
- **ClonoCluster** (Generates the hybrid α clusters whose rearrangement relative to transcriptome-only clusters defines the reorganization markers tested for enrichment) — https://github.com/leeprichman/ClonoCluster
- **ROCR** (ROC/AUC calculation (via the Find_Markers_ROC wrapper) used to compute reorg-AUC and select markers above 0.80 before enrichment testing)
- **data.table** (CRAN dependency used in the Marker_analysis.R script that identifies markers and performs the overrepresentation analysis) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
res <- WebGestaltR(enrichMethod="ORA", organism="hsapiens", enrichDatabase=c("geneontology_Cellular_Component","geneontology_Molecular_Function"), interestGene=reorg_markers, fdrThr=0.05)
```

## Evaluation signals

- enrichment_table.csv contains significant GO CC/MF terms at FDR < 0.05 (Benjamini-Hochberg corrected hypergeometric test)
- Ribosomal/translation gene sets (polysome, rRNA binding, structural constituent of ribosome) and extracellular matrix sets appear among top enriched categories
- Marker list input to WebGestaltR contains only genes with reorg-AUC > 0.80 between transcriptome-only and hybrid cluster pairs
- Enrichment results are reproducible across the 9 replicates from six datasets, with translation and ECM the most widely shared gene sets
- Output files follow the paper's manifest convention: *_rearrangment_ORA.txt generated by Marker_analysis.R alongside *_auc_barcodes.txt

## Limitations

- The paper notes further testing of the homogeneity of functional properties of cells within each cluster is required to determine which clustering methods most closely match biological distinctions, so enrichment categories are suggestive rather than definitive
- The study did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate, so rearrangement enrichment may not transfer to systems without clonal influence on clustering
- Whether the appropriate relative weighting of transcriptome vs clone information differs between endpoint and transitional systems remains unresolved and requires case-by-case evaluation, affecting which α-derived marker lists should be enriched

## Evidence

- [results] Overrepresentation analysis of reorganization markers showed translation/ribosomal and ECM gene sets among top enriched categories: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: genes most strongly associated with rearrangements were ribosomal function and extracellular matrix genes: "The genes most strongly associated with the rearrangements in hybrid clusters were ribosomal function and extracellular matrix genes."
- [readme] Marker_analysis.R performs gene set overrepresentation analysis: [evidence span withheld — anchor status: external_doc]
- [methods] Cluster marker analysis uses Find_Markers_ROC: "* Cluster marker analysis with `Find_Markers_ROC`"
