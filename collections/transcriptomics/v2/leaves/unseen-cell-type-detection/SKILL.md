---
name: unseen-cell-type-detection
description: Use when a query scRNA-seq dataset (AnnData) contains cell types not
  represented in the reference used for MASI label transfer, e.g. query annotations
  exceed reference cell types (Oetjen et al. 16 types vs 10x PBMC 12 types).
when_to_use_negative:
- When every query cell type is present in the reference — standard MASI gene2cell
  + parallel annotation suffices without certainty filtering
- When input is raw counts or marker-gene expression has not yet been converted to
  a cell-type score matrix via masi.gene2cell
- When the reference itself is low quality or resolution-mismatched to the query —
  the article warns accuracy is dependent on the quality of reference data
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Computes cell-type scores via gene2cell, transfers labels via parallel, and
    provides cal_centainty for the certainty-score detection of unseen cell types
  repo: https://github.com/hayatlab/MASI
- name: SCCAF
  role: Combined with MASI to identify subtypes within major cell types when the query
    has unseen cell types not in the reference
- name: Scanpy
  role: Provides the AnnData object format and preprocessing (e.g. pp.highly_variable_genes
    in the broader workflow) in which MASI operates
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:33a4d80c5f6b18f46180a8ce348b6e4d2d9d9bbb3ea6612b4be757df487d526b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - SCCAF
  - Scanpy
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# unseen-cell-type-detection

## Summary

Detects query-data cell types absent from a MASI reference dataset by computing a per-cell certainty score (ratio of distance to the closest reference centroid over distance to the least-close centroid) and flagging low-certainty cells as unassigned; the paper found thresholds between 0.5 and 0.6 retrieve the 4 extra cell types in Oetjen et al. with decent accuracy.

## When to use

Use when a query scRNA-seq dataset (AnnData) contains cell types not represented in the reference used for MASI label transfer, e.g. query annotations exceed reference cell types (Oetjen et al. 16 types vs 10x PBMC 12 types).

## When NOT to use

- When every query cell type is present in the reference — standard MASI gene2cell + parallel annotation suffices without certainty filtering
- When input is raw counts or marker-gene expression has not yet been converted to a cell-type score matrix via masi.gene2cell
- When the reference itself is low quality or resolution-mismatched to the query — the article warns accuracy is dependent on the quality of reference data

## Inputs

- Processed query scRNA-seq dataset as an AnnData object (from the figshare deposit 18866264 in the paper)
- Reference dataset cell-type markers (dict from masi.marker_identification on labeled reference data, e.g. 10x Genomics PBMC with 12 cell types)
- Cell-type score matrix (scores) and labels produced by masi.gene2cell
- Published/ground-truth query cell-type annotations (for ARI/NMI validation)

## Outputs

- Per-cell certainty score table (cell barcode, assigned label, certainty ratio) e.g. certainty_scores.csv
- Set of cells flagged as unassigned under thresholds 0.5 and 0.6
- ARI/NMI comparison table for MASI alone vs SCCAF alone vs SCCAF+MASI (e.g. ari_nmi_masi_sccaf_comparison.csv)
- Certainty threshold sweep plot (e.g. certainty_threshold_sweep.png)

## How to apply

After annotating query cells with masi.gene2cell (cell_markers from the reference, use_weight=True) and masi.parallel (batch_size=50000), compute reference cell-type centroids in the cell-type score space and run MASI's cal_centainty function to obtain, for each annotated cell, the ratio of the distance to the closest centroid over the distance to the least-close centroid (certainty C = 1 - D1st/DNth). Sweep the certainty threshold over 0.5–0.6: cells falling below the threshold are reported as unassigned, which surfaces query cell types absent from the reference (in the paper, erythrocytes and erythroid progenitors mislabeled as CD14+ monocytes and HSPCs by MASI alone). Because MASI alone cannot annotate unseen types, combine MASI with SCCAF to identify subtypes within major cell types, and evaluate against published annotations with ARI and NMI. Note that annotation accuracy depends on reference quality, so the reference choice and resolution are critical.

## Related tools

- **MASI** (Computes cell-type scores via gene2cell, transfers labels via parallel, and provides cal_centainty for the certainty-score detection of unseen cell types) — https://github.com/hayatlab/MASI
- **SCCAF** (Combined with MASI to identify subtypes within major cell types when the query has unseen cell types not in the reference)
- **Scanpy** (Provides the AnnData object format and preprocessing (e.g. pp.highly_variable_genes in the broader workflow) in which MASI operates)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- All cells whose true cell type is absent from the reference fall below the tested certainty thresholds in [0.5, 0.6] and are labelled unassigned
- The per-cell certainty table exists with a certainty ratio column for every cell (cell barcode, assigned label, certainty ratio)
- ARI/NMI for MASI alone, SCCAF alone, and SCCAF+MASI reproduce the relative performance in Supplementary Figs. 8–9 (unseen types mislabeled by MASI alone, recovered by SCCAF+MASI)
- The threshold sweep shows the 4 unseen cell types retrieved within the 0.5–0.6 certainty-score range
- certainty_scores.csv and certainty_threshold_sweep.png are produced and populated as specified in the workflow

## Limitations

- MASI alone cannot annotate unseen cell types not present in the reference; it requires combining MASI with SCCAF to identify subtypes within major cell types
- Accurate annotation via MASI is dependent on the quality of reference data; the choice and resolution of the reference are critical to downstream analysis
- Threshold values are empirical: 0.5–0.6 retrieves the 4 extra cell types in Oetjen et al. with 'decent accuracy' — no universally optimal threshold is established

## Evidence

- [results] MASI reports a cell as unassigned when its certainty score falls below a chosen threshold; setting the threshold between 0.5 and 0.6 retrieves the 4 extra cell types in Oetjen et al. with decent accuracy.: [evidence span withheld — anchor status: not_found]
- [discussion] If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types.: "If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types."
- [methods] scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True): "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [methods] annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000): "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
- [discussion] accurate annotation via MASI is also dependent on the quality of reference data. Thus, the choice and resolution of the reference are critical to downstream analysis.: "the choice and resolution of the reference are critical to downstream analysis"
