---
name: entropy-analysis-of-cluster-label-distributions
description: Use when hybrid clusters from ClonoCluster runs (alpha = 0 vs low alpha)
  must be compared against known cell-type ground-truth labels to demonstrate clonal
  information increases cluster purity.
when_to_use_negative:
- No known ground-truth cell-type labels are available for the dataset — entropy of
  label distributions is undefined without annotations
- The goal is to assess marker fidelity or cluster reorganization (use Find_Markers_ROC
  / Plot_alluvia_track or reorganization-marker AUC analyses instead)
- Only a single clustering level (one alpha value) exists — the skill requires paired
  alpha = 0 vs low-alpha assignments for a reduction comparison
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ClonoCluster
  role: generates hybrid cluster assignments across a range of alpha values (alpha
    = seq(0, 1, by = 0.1), beta = 0.1, res = 1) that are cross-tabulated with cell-type
    labels
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: loads the count matrix and barcode table with fread and provides dt2m for
    matrix conversion
- name: cast_confusion
  role: cross-tabulates known cell-type labels against cluster assignments at each
    alpha value prior to entropy computation
  repo: https://github.com/leeprichman/ClonoCluster
- name: entropy
  role: CRAN package used by the ClonoCluster_paper workflow to compute Shannon entropy
    of cell-type label distributions (Heme_comp.R)
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:dd394f73255fa89ce383bcb65dd6304edc62fd0cf8131b573649280b4934e4bf
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# entropy-analysis-of-cluster-label-distributions

## Summary

Quantifies Shannon entropy of known cell-type label distributions within clusters from ClonoCluster hybrid clustering at alpha = 0 (transcriptome-only) versus a low alpha, to verify that clonal information increases cluster purity (Figure S7, Weinreb et al. in vitro hematopoiesis data).

## When to use

Use when hybrid clusters from ClonoCluster runs (alpha = 0 vs low alpha) must be compared against known cell-type ground-truth labels to demonstrate clonal information increases cluster purity.

## When NOT to use

- No known ground-truth cell-type labels are available for the dataset — entropy of label distributions is undefined without annotations
- The goal is to assess marker fidelity or cluster reorganization (use Find_Markers_ROC / Plot_alluvia_track or reorganization-marker AUC analyses instead)
- Only a single clustering level (one alpha value) exists — the skill requires paired alpha = 0 vs low-alpha assignments for a reduction comparison

## Inputs

- gene-by-cell normalized/scaled count matrix for barcoded cells (e.g. Data_genes/ Kleind2ws table from ClonoCluster_paper release)
- barcode assignment table of unique cell IDs and their barcodes (Data_barcodes/)
- known cell-type labels for each cell (Weinreb et al. in vitro hematopoiesis annotations)

## Outputs

- per-cluster Shannon entropy of cell-type label distributions at alpha = 0 and low alpha
- per-cluster and mean entropy reduction values (e.g. entropy_comparison.csv)
- cell-type vs cluster confusion cross-tabulations from cast_confusion

## How to apply

Load the single-cell count matrix and barcode table with data.table::fread, ensuring cell IDs match between tables, then convert the count matrix with dt2m and compute PCA with irlba_wrap(cm, npc = 25). Generate hybrid clusters via clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) and retain assignments at alpha = 0 and at a low alpha value. Cross-tabulate known cell-type labels (basophils, erythroid progenitors, lymphoid progenitors, mast cells, megakaryocytes, migratory dendritic cells, plasmacytoid dendritic cells) against cluster assignments at each alpha using cast_confusion. Compute the Shannon entropy of the cell-type label distribution within each cluster (e.g. with the CRAN 'entropy' package) at both alpha values, then derive per-cluster and mean entropy reduction. The paper applies this to the Kleind2ws in vitro hematopoiesis dataset, where the Heme_comp.R script produced the entropy comparison in Figure S7.

## Related tools

- **ClonoCluster** (generates hybrid cluster assignments across a range of alpha values (alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) that are cross-tabulated with cell-type labels) — https://github.com/leeprichman/ClonoCluster
- **data.table** (loads the count matrix and barcode table with fread and provides dt2m for matrix conversion)
- **cast_confusion** (cross-tabulates known cell-type labels against cluster assignments at each alpha value prior to entropy computation) — https://github.com/leeprichman/ClonoCluster
- **entropy** (CRAN package used by the ClonoCluster_paper workflow to compute Shannon entropy of cell-type label distributions (Heme_comp.R)) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
cm %<>% dt2m; pca <- irlba_wrap(cm, npc = 25); clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Every cluster at low alpha shows a Shannon entropy no greater than its alpha = 0 counterpart, consistent with the low-alpha purity increase in Figure S7
- Entropy reduction is observed for the expected cell types: basophils, erythroid progenitors, lymphoid progenitors, mast cells, megakaryocytes, migratory dendritic cells, and plasmacytoid dendritic cells
- Cell IDs in the count matrix and barcode table match exactly before clustering (fread + ID check)
- cast_confusion cross-tabulations are generated at both alpha = 0 and low alpha, with row sums equal to total labeled cell counts
- Output file entropy_comparison.csv contains entropy values and reductions for all clusters with no missing alpha = 0 counterparts

## Limitations

- Reduced entropy shows increased label homogeneity but does not by itself establish functional homogeneity; the paper notes that further testing of homogeneity of functional properties of cells within each cluster is required
- The appropriate relative weighting of transcriptome vs clone information may differ between endpoint and transitional systems and requires case-by-case evaluation, so a fixed low alpha may not transfer across datasets
- In this study the entropy comparison was applied only to the in vitro hematopoiesis (Kleind2ws/Weinreb) dataset as supplementary analysis (Figure S7), not across all six datasets

## Evidence

- [methods] clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1): "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [methods] Barcode to cluster concordance plots with cast_confusion: "* Barcode to cluster concordance plots with `cast_confusion`"
- [readme] Heme_comp.R entropy of hematopoietic cell types from Weinreb et al.: [evidence span withheld — anchor status: external_doc]
- [readme] ClonoCluster_paper requires the entropy CRAN package: [evidence span withheld — anchor status: external_doc]
