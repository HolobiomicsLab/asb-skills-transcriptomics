---
name: median-auc-benchmark-comparison
description: Use when you have ClonoCluster cluster assignments across a range of
  α values plus a count matrix and need to verify that hybrid clustering did not degrade
  top-marker sensitivity/specificity (AUC) relative to transcriptome-only clustering.
when_to_use_negative:
- Input is not hybrid cluster assignments spanning α levels — e.g. a single transcriptome-only
  Seurat/scanpy clustering with no clone-weighted variants to compare
- No barcode/clone information exists for the cells, so there is no α > 0 clustering
  to benchmark against the α = 0 baseline
- The goal is functional validation of cluster homogeneity rather than marker fidelity
  — the paper notes further testing of homogeneity of functional properties is still
  required to determine which clustering methods match biological distinctions
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ClonoCluster
  role: Generates the hybrid cluster assignments across α = seq(0, 1, by = 0.1) that
    are benchmarked for marker fidelity
  repo: https://github.com/leeprichman/ClonoCluster
- name: Find_Markers_ROC
  role: ClonoCluster package function that computes ROCR-based ROC AUC of top cluster
    markers used in the fidelity comparison
  repo: https://github.com/leeprichman/ClonoCluster
- name: ROCR
  role: R package underlying ROC AUC computation of the top marker gene for each cluster
- name: pheatmap
  role: Renders the marker-fidelity heatmap of top cluster marker AUC at each α level
- name: data.table
  role: Loads the count matrix and barcode tables via fread and handles the cluster-assignment
    tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:097f5f4021519ef309631ada8f8d30cf3e8bc789f3cbc0afe839665dcfe0ee63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - Find_Markers_ROC
  - R (version >=)
  - ROCR
  - data.table
  - pheatmap
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# median-auc-benchmark-comparison

## Summary

Benchmarks whether hybrid clusters produced by ClonoCluster at each α level retain marker fidelity by computing ROC AUC of each cluster's top marker (Find_Markers_ROC/ROCR) and comparing the per-α median top-marker AUC to the transcriptome-only (α = 0) baseline across datasets.

## When to use

Use when you have ClonoCluster cluster assignments across a range of α values plus a count matrix and need to verify that hybrid clustering did not degrade top-marker sensitivity/specificity (AUC) relative to transcriptome-only clustering.

## When NOT to use

- Input is not hybrid cluster assignments spanning α levels — e.g. a single transcriptome-only Seurat/scanpy clustering with no clone-weighted variants to compare
- No barcode/clone information exists for the cells, so there is no α > 0 clustering to benchmark against the α = 0 baseline
- The goal is functional validation of cluster homogeneity rather than marker fidelity — the paper notes further testing of homogeneity of functional properties is still required to determine which clustering methods match biological distinctions

## Inputs

- Normalized/scaled count matrix in gene-by-cell format (e.g. mycountmatrix.tsv, loadable via data.table::fread)
- Barcode table mapping unique cell IDs to assigned barcodes (mybarcodetable.tsv)
- Cluster-assignment table from clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) covering all α levels

## Outputs

- Per-cluster top-marker ROC AUC values at every α level (marker_fidelity_auc.tsv)
- Per-α median top-marker AUC per dataset
- Marker fidelity heatmap (pheatmap) of top cluster marker AUC across α levels

## How to apply

Load the normalized count matrix (e.g. mycountmatrix.tsv) and barcode table (mybarcodetable.tsv) with data.table::fread, confirming cell IDs match exactly, and load the cluster-assignment table produced by clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1). For each dataset and each α level (α = 0, low α ≈ 0.1–0.3, high α ≈ 0.6–0.9, α = 1), run Find_Markers_ROC(dl, cm), which uses ROCR to compute the ROC AUC of the top marker gene for each cluster. Compute the median top-marker AUC per α level, write per-cluster AUCs and per-α medians to marker_fidelity_auc.tsv, and render a marker-fidelity heatmap with pheatmap. Validate that all AUCs fall within [0.5, 1] and that median AUCs at hybrid α values are not significantly different from the α = 0 baseline for most datasets. Rationale: the paper found markers of hybrid clusters were different but had equivalent fidelity, with exceptions in extrinsically determined systems — the cardiomyocyte-directed iPSC dataset (Jiang et al.) dropped to 0.69 and 0.79 versus 0.94 at high α and clonal levels, and low-dose vemurafenib samples also showed reduced fidelity.

## Related tools

- **ClonoCluster** (Generates the hybrid cluster assignments across α = seq(0, 1, by = 0.1) that are benchmarked for marker fidelity) — https://github.com/leeprichman/ClonoCluster
- **Find_Markers_ROC** (ClonoCluster package function that computes ROCR-based ROC AUC of top cluster markers used in the fidelity comparison) — https://github.com/leeprichman/ClonoCluster
- **ROCR** (R package underlying ROC AUC computation of the top marker gene for each cluster)
- **pheatmap** (Renders the marker-fidelity heatmap of top cluster marker AUC at each α level)
- **data.table** (Loads the count matrix and barcode tables via fread and handles the cluster-assignment tables)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- All computed AUC values fall within [0.5, 1]
- Median top-marker AUC at hybrid α values is not significantly different from the transcriptome-only (α = 0) baseline for most datasets
- Known exceptions reproduce: decreased fidelity for cardiomyocyte-directed iPSCs (0.69 and 0.79 versus 0.94) at high α and clonal levels, and for low-dose vemurafenib samples
- Cell IDs in the barcode table exactly match those in the count matrix before analysis
- marker_fidelity_auc.tsv contains one row per cluster per α level with per-α medians derivable from it

## Limitations

- Extrinsically determined systems (cardiomyocyte-directed iPSCs from Jiang et al., low-dose vemurafenib) show decreased marker fidelity at high α, so fidelity preservation is not universal
- Marker fidelity is not equivalent to functional homogeneity; further testing of functional homogeneity within clusters is required to determine which clustering methods match biological distinctions
- The study did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information
- Appropriate relative weighting of transcriptome vs clone information may differ between endpoint and transitional systems and requires case-by-case evaluation

## Evidence

- [abstract] Markers of hybrid clusters were different from transcriptome-only cluster markers but had equivalent fidelity: "The markers of these hybrid clusters were different but had equivalent fidelity to transcriptome-only clusters."
- [methods] Find_Markers_ROC computes ROC-based cluster marker AUCs: "Cluster marker analysis with `Find_Markers_ROC`"
- [methods] ClonoCluster call with alpha grid and beta/res parameters: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [discussion] Cardiomyocyte-directed iPSC exception with reduced fidelity at high alpha: "in the case of cardiomyocyte-directed iPSCs from Jiang et al.,18 in which we know that cell fate is largely determined by extrinsic factors"
- [readme] Paper repo includes AUC heatmaps across alpha levels: [evidence span withheld — anchor status: external_doc]
- [discussion] Marker fidelity analysis confirms alpha choice: "Analysis of marker fidelity or comparison to ground truth data can be used to confirm that the α value produces hybrid clusters that are biologically meaningful"
