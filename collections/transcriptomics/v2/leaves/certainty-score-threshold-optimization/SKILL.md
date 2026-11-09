---
name: certainty-score-threshold-optimization
description: Use when MASI-annotated query data contains cell types absent from the
  reference (e.g., Oetjen et al. 16 cell types vs. 12-type 10x PBMC reference) and
  you must recover the unseen types by tuning the certainty-score threshold in the
  0.5–0.6 range.
when_to_use_negative:
- All query cell types are represented in the reference dataset — MASI alone annotates
  these correctly, so the certainty-threshold sweep and SCCAF combination are unnecessary
- No labeled reference dataset is available to compute cell_markers and cell-type
  scores via masi.gene2cell
- Input is not normalized single-cell transcriptomics data (AnnData/h5ad) but, e.g.,
  a feature table or bulk expression matrix
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MASI
  role: Computes cell-type scores (masi.gene2cell), transfers labels (masi.parallel),
    and provides masi.cal_centainty for the certainty-score threshold sweep
  repo: https://github.com/hayatlab/MASI
- name: SCCAF
  role: Combined with MASI to identify subtypes within major cell types when query
    data contains unseen cell types
- name: Scanpy
  role: AnnData handling and preprocessing of the query/reference datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:b87abc9672c51440c7ce71cfde823d3cee0ece00fc04e9d0801cfe2f1f4e8e24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - SCCAF
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# certainty-score-threshold-optimization

## Summary

Sweeps the MASI certainty-score threshold (0.5–0.6, via masi.cal_centainty) to recover query cell types that are absent from the reference dataset and are otherwise mislabeled by MASI alone, benchmarked with ARI/NMI for MASI-alone, SCCAF-alone, and SCCAF+MASI conditions.

## When to use

Use when MASI-annotated query data contains cell types absent from the reference (e.g., Oetjen et al. 16 cell types vs. 12-type 10x PBMC reference) and you must recover the unseen types by tuning the certainty-score threshold in the 0.5–0.6 range.

## When NOT to use

- All query cell types are represented in the reference dataset — MASI alone annotates these correctly, so the certainty-threshold sweep and SCCAF combination are unnecessary
- No labeled reference dataset is available to compute cell_markers and cell-type scores via masi.gene2cell
- Input is not normalized single-cell transcriptomics data (AnnData/h5ad) but, e.g., a feature table or bulk expression matrix

## Inputs

- Processed query scRNA-seq AnnData with published cell-type annotations (e.g., Oetjen et al., 16 cell types, from figshare deposit 18866264)
- Processed reference AnnData with cell-type labels (e.g., 10x Genomics PBMC, 12 cell types) and its MASI-derived cell_markers dict
- Cell-type score matrix from masi.gene2cell (raw scores plus PlinerScore conversion with TF-IDF and 0.25 percentile threshold)

## Outputs

- Per-cell certainty scores across the 0.5–0.6 threshold sweep (masi.cal_centainty)
- ari_nmi_masi_sccaf_comparison.csv — ARI/NMI table for MASI alone, SCCAF alone, and SCCAF+MASI
- certainty_threshold_sweep.png — plot showing the 4 unseen cell types recovered in the 0.5–0.6 range
- Recovered query labels for unseen cell types (e.g., erythrocytes, erythroid progenitors)

## How to apply

Compute cell-type scores from the query AnnData with masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) using markers from the reference (e.g., the 10x Genomics PBMC 12-cell-type reference), then annotate with masi.parallel(scores=scores, labels=labels, batch_size=50000). Because MASI alone forces unseen query types into the nearest reference label — in the Oetjen et al. query, erythrocytes became CD14+ monocytes and erythroid progenitors became HSPCs — sweep the certainty-score threshold over 0.5–0.6 with masi.cal_centainty to flag low-confidence cells and recover the 4 unseen types, pairing MASI with SCCAF to identify subtypes within major cell types. For each condition (MASI alone, SCCAF alone, SCCAF+MASI), compute adjusted Rand index (ARI) and normalized mutual information (NMI) against the query's published annotations and write the comparison table (ari_nmi_masi_sccaf_comparison.csv) plus a threshold-sweep plot (certainty_threshold_sweep.png). The sweep is validated only if the 4 unseen cell types are retrieved within the 0.5–0.6 certainty range and the ARI/NMI ordering matches Supplementary Figs. 8–9.

## Related tools

- **MASI** (Computes cell-type scores (masi.gene2cell), transfers labels (masi.parallel), and provides masi.cal_centainty for the certainty-score threshold sweep) — https://github.com/hayatlab/MASI
- **SCCAF** (Combined with MASI to identify subtypes within major cell types when query data contains unseen cell types)
- **Scanpy** (AnnData handling and preprocessing of the query/reference datasets)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- Certainty-threshold sweep plot shows the 4 unseen query cell types retrieved within the 0.5–0.6 certainty-score range
- ARI/NMI computed against the query's published annotations reproduce the relative performance of MASI-alone vs SCCAF-alone vs SCCAF+MASI shown in Supplementary Figs. 8–9
- MASI-alone condition reproduces the characteristic mislabels: erythrocytes as CD14+ monocytes and erythroid progenitors as HSPCs (Supplementary Fig. 8)
- Output files ari_nmi_masi_sccaf_comparison.csv and certainty_threshold_sweep.png are produced with the specified schemas

## Limitations

- MASI alone cannot annotate unseen cell types not present in the reference; recovery requires combining MASI with SCCAF
- Accurate MASI annotation depends on the quality of the reference data; reference choice and resolution are critical to downstream analysis
- No software version numbers were reported for the tools used (Scanpy, MASI, SCCAF, etc.), complicating exact reproduction

## Evidence

- [discussion] MASI alone cannot annotate unseen cell types: "If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types."
- [readme] MACA-based label transfer pipeline: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
- [methods] gene2cell/parallel invocation: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] Reference quality caveat: "accurate annotation via MASI is also dependent on the quality of reference data. Thus, the choice and resolution of the reference are critical to downstream analysis."
