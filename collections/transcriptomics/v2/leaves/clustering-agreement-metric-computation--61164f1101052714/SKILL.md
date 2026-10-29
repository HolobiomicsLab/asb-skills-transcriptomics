---
name: clustering-agreement-metric-computation
description: Use when comparing cell-type label assignments (e.g., MASI alone vs SCCAF
  vs SCCAF+MASI) against published annotations on an scRNA-seq query dataset, or when
  sweeping a certainty-score threshold to recover unseen cell types.
when_to_use_negative:
- The query dataset has no ground-truth or published cell-type annotations, making
  ARI/NMI against reference labels undefined
- The goal is to compare integration/latent-space quality rather than discrete label
  agreement (ARI/NMI on labels does not evaluate batch-corrected embeddings)
- The reference contains all query cell types, so the comparison is not about unseen-type
  failure modes — a simpler single-condition annotation may suffice
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Produces the predicted labels and certainty scores (masi.gene2cell, masi.parallel,
    masi.cal_centainty) whose agreement with published annotations is being scored
  repo: https://github.com/hayatlab/MASI
- name: SCCAF
  role: Alternative and combined annotation condition (SCCAF alone, SCCAF+MASI) included
    in the ARI/NMI comparison and threshold recovery of unseen cell types
- name: Scanpy
  role: AnnData handling of the query and reference expression matrices feeding the
    annotation pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:a4ea87ec33c30c98552bb1dc305ad9173a79402668c59f3f82f81a6e68dbb244
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# clustering-agreement-metric-computation

## Summary

Quantifies agreement between predicted and reference cell-type annotations using adjusted Rand index (ARI) and normalized mutual information (NMI), as used to compare MASI alone, SCCAF alone, and SCCAF+MASI on the Oetjen et al. query data against its published annotations.

## When to use

Use when comparing cell-type label assignments (e.g., MASI alone vs SCCAF vs SCCAF+MASI) against published annotations on an scRNA-seq query dataset, or when sweeping a certainty-score threshold to recover unseen cell types.

## When NOT to use

- The query dataset has no ground-truth or published cell-type annotations, making ARI/NMI against reference labels undefined
- The goal is to compare integration/latent-space quality rather than discrete label agreement (ARI/NMI on labels does not evaluate batch-corrected embeddings)
- The reference contains all query cell types, so the comparison is not about unseen-type failure modes — a simpler single-condition annotation may suffice

## Inputs

- Annotated query AnnData with published cell-type labels (Oetjen et al., 16 cell types)
- Predicted labels for each annotation condition (MASI alone, SCCAF alone, SCCAF+MASI)
- Cell-type certainty scores from masi.cal_centainty for threshold sweeping

## Outputs

- ari_nmi_masi_sccaf_comparison.csv — ARI and NMI per annotation condition
- certainty_threshold_sweep.png — unseen-cell-type recovery across 0.5–0.6 certainty thresholds

## How to apply

Annotate the query dataset under each condition being compared (e.g., masi.gene2cell followed by masi.parallel for MASI alone, SCCAF alone, and the combined SCCAF+MASI annotation), then compute ARI and NMI of each condition against the query's published annotations. In the Oetjen et al. (16 cell types) vs 10x Genomics PBMC reference (12 cell types) experiment, expect MASI alone to mislabel the 4 unseen types — erythrocytes as CD14+ monocytes and erythroid progenitors as HSPCs — so compare relative ARI/NMI across the three conditions as in Supplementary Figs. 8–9 rather than absolute values. To test recovery of unseen types, sweep the certainty-score threshold from masi.cal_centainty over 0.5–0.6 and check which labels fall below it (low-certainty cells are candidates for re-clustering with SCCAF). Write the ARI/NMI values for all conditions to ari_nmi_masi_sccaf_comparison.csv and save the threshold sweep plot as certainty_threshold_sweep.png. Validation: the three-condition ARI/NMI comparison must reproduce the relative performance in Supplementary Figs. 8–9, and the sweep must show the 4 unseen cell types retrieved within the 0.5–0.6 certainty range.

## Related tools

- **MASI** (Produces the predicted labels and certainty scores (masi.gene2cell, masi.parallel, masi.cal_centainty) whose agreement with published annotations is being scored) — https://github.com/hayatlab/MASI
- **SCCAF** (Alternative and combined annotation condition (SCCAF alone, SCCAF+MASI) included in the ARI/NMI comparison and threshold recovery of unseen cell types)
- **Scanpy** (AnnData handling of the query and reference expression matrices feeding the annotation pipeline)

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score; scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); ann = masi.parallel(scores=scores, labels=labels, batch_size=50000); ari = adjusted_rand_score(ad.obs['cell_type'], ann)
```

## Evaluation signals

- ARI/NMI for the three annotation conditions (MASI alone, SCCAF alone, SCCAF+MASI) reproduce the relative performance shown in Supplementary Figs. 8–9
- The certainty-threshold sweep over 0.5–0.6 retrieves all 4 unseen query cell types
- MASI-alone condition mislabels erythrocytes as CD14+ monocytes and erythroid progenitors as HSPCs, matching Supplementary Fig. 8
- ari_nmi_masi_sccaf_comparison.csv contains one ARI and one NMI row per condition against the Oetjen et al. published annotations
- Annotations are generated with the reported parameters (batch_size=50000 in masi.parallel, use_weight=True in masi.gene2cell)

## Limitations

- MASI alone cannot annotate unseen cell types absent from the reference; it must be combined with SCCAF to identify subtypes within major cell types
- MASI annotation accuracy depends on the quality of the reference data; the choice and resolution of the reference are critical to downstream analysis
- Benchmark conclusions rest on the datasets used in this study, and no software version numbers are reported for the tools involved

## Evidence

- [discussion] If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types.: "If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types."
- [supplementary] Because erythrocytes and erythroid progenitors were not present in the 10x Genomics reference, MASI mislabeled these unseen query cell types as CD14+ monocytes and HSPCs: [evidence span withheld — anchor status: not_found]
- [methods] scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True): "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [methods] annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000): "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
