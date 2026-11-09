---
name: hematopoietic-cell-type-annotation
description: Use when a single-cell hematopoiesis dataset (e.g., Weinreb et al. Kleind2ws
  murine HSC differentiation data) has ground-truth cell-type labels and you need
  to quantify cluster purity via per-cluster label entropy under different clustering
  schemes.
when_to_use_negative:
- The dataset lacks known cell-type labels or ground-truth annotations, so no label-distribution
  entropy can be computed
- The data are not hematopoietic/single-cell with clonal barcodes — entropy comparison
  across α levels presumes hybrid (clonal + transcriptome) clustering
- The goal is de novo marker discovery or gene set overrepresentation analysis (use
  Find_Markers_ROC / WebGestaltR workflow instead)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ClonoCluster
  role: generates hybrid cluster assignments across α values that are cross-tabulated
    against known hematopoietic cell-type labels
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: loads the count matrix and barcode table (fread) and reshapes label-by-cluster
    cross-tabulations
- name: cast_confusion
  role: ClonoCluster function producing barcode/cell-type to cluster concordance (confusion)
    plots used before entropy computation
  repo: https://github.com/leeprichman/ClonoCluster
- name: entropy
  role: R package dependency for computing Shannon entropy of the cell-type label
    distribution within clusters
- name: irlba_wrap
  role: computes the PCA (npc = 25) fed into clonocluster for the hematopoiesis dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:e7e56f3a62ae873967e5f229b72e6bbcd513a8a397fc65156219658afe5561e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - cast_confusion
  - data.table
  - entropy
  - irlba_wrap
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# hematopoietic-cell-type-annotation

## Summary

Annotate and evaluate clustering quality of single-cell hematopoiesis data by cross-tabulating known cell-type labels (basophils, erythroid/megakaryocyte progenitors, mast cells, dendritic cell subsets) against cluster assignments and computing Shannon entropy of the cell-type distribution per cluster. Used in the ClonoCluster paper to show that low-α hybrid clusters reduce entropy (increase purity) relative to transcriptome-only clustering in the in vitro hematopoiesis dataset (Figure S7).

## When to use

Use when a single-cell hematopoiesis dataset (e.g., Weinreb et al. Kleind2ws murine HSC differentiation data) has ground-truth cell-type labels and you need to quantify cluster purity via per-cluster label entropy under different clustering schemes.

## When NOT to use

- The dataset lacks known cell-type labels or ground-truth annotations, so no label-distribution entropy can be computed
- The data are not hematopoietic/single-cell with clonal barcodes — entropy comparison across α levels presumes hybrid (clonal + transcriptome) clustering
- The goal is de novo marker discovery or gene set overrepresentation analysis (use Find_Markers_ROC / WebGestaltR workflow instead)

## Inputs

- normalized/scaled single-cell count matrix (gene by cell, e.g. Data_genes/ table from ClonoCluster_paper)
- barcode assignment table of unique cell IDs and assigned barcodes (Data_barcodes/)
- known cell-type labels for hematopoietic cells (basophils, erythroid progenitors, lymphoid progenitors, mast cells, megakaryocytes, migratory dendritic cells, plasmacytoid dendritic cells)

## Outputs

- per-cluster Shannon entropy values of cell-type label distributions at α = 0 and low α
- per-cluster and mean entropy reduction statistics (e.g. entropy_comparison.csv)
- confusion/cross-tabulation matrices of cell types versus cluster assignments (cast_confusion output)
- entropy comparison plots of cell types across clustering methods (Figure S7-style, Heme_comp.R output)

## How to apply

Load the single-cell count matrix and barcode table with data.table::fread, ensuring cell IDs match, then convert with dt2m and compute PCA with irlba_wrap(npc = 25). Generate hybrid clusters with clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1), retaining assignments at α = 0 (transcriptome-only) and at a low α. Cross-tabulate the known cell-type labels against cluster assignments at each α using cast_confusion, then compute Shannon entropy of the label distribution within each cluster at both α values. Calculate per-cluster and mean entropy reduction (low α vs α = 0); a correct application shows every cluster at low α with entropy no greater than its α = 0 counterpart, consistent with Figure S7.

## Related tools

- **ClonoCluster** (generates hybrid cluster assignments across α values that are cross-tabulated against known hematopoietic cell-type labels) — https://github.com/leeprichman/ClonoCluster
- **data.table** (loads the count matrix and barcode table (fread) and reshapes label-by-cluster cross-tabulations)
- **cast_confusion** (ClonoCluster function producing barcode/cell-type to cluster concordance (confusion) plots used before entropy computation) — https://github.com/leeprichman/ClonoCluster
- **entropy** (R package dependency for computing Shannon entropy of the cell-type label distribution within clusters)
- **irlba_wrap** (computes the PCA (npc = 25) fed into clonocluster for the hematopoiesis dataset)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Every cluster's entropy at low α is ≤ its α = 0 (transcriptome-only) counterpart
- Entropy reduction is observed for the expected cell types: basophils, erythroid progenitors, lymphoid progenitors, mast cells, megakaryocytes, migratory and plasmacytoid dendritic cells (Figure S7)
- Cell IDs match 1:1 between the count matrix and barcode table before clustering
- Confusion matrices from cast_confusion show coherent cell-type enrichment in low-α clusters
- entropy_comparison.csv is written with finite, non-negative entropy values and reductions for all clusters

## Limitations

- Further testing of the homogeneity of functional properties of cells within each cluster is required to determine which clustering methods most closely match biological distinctions
- The entropy comparison depends on a chosen low-α value; appropriate relative weighting of transcriptome vs clone information may require case-by-case evaluation
- The analysis is grounded in the Weinreb et al. in vitro hematopoiesis dataset (Kleind2ws); conclusions may not transfer to systems with extrinsically determined cell fate

## Evidence

- [results] finding: [evidence span withheld — anchor status: not_found]
- [methods] clonocluster call: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [methods] irlba PCA step: "pca <- irlba_wrap(cm, npc = 25)"
- [readme] Heme_comp script: [evidence span withheld — anchor status: external_doc]
- [readme] hematopoiesis dataset: [evidence span withheld — anchor status: not_found]
