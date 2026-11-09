---
name: deg-list-comparison
description: Use when you have two or more ranked or unranked gene lists (e.g., Wx
  DI-ranked UGCBs, edgeR DEGs sorted by adjusted p value, published biomarker signatures)
  and need to measure their overlap to assess method concordance.
when_to_use_negative:
- You need to compare the classification performance of the gene sets rather than
  their membership — use LOOCV or a classifier (NN/SVM/XGBoost) comparison instead.
- The gene lists come from different data sets or different gene identifier schemes
  without harmonization — symbol-level intersection would be meaningless.
- You expect functional similarity rather than literal gene overlap — use GO/network
  analysis (e.g., Metascape on top 50 UGCBs) instead.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Python 3.4
  role: Run set-intersection operations between the Wx gene list and each baseline
    gene set
  repo: https://github.com/deargen/DearWXpub
- name: Wx
  role: Produces the query top-ranked gene list via DI scores after 10,000 iterations
  repo: https://github.com/deargen/DearWXpub
- name: edgeR
  role: Produces the baseline DEG-14-UGCB list (top 14 DEGs by adjusted p value) from
    the same gene-expression data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:a06d305408a46d0a9bd9d216e1fcbdc31bdcfc9df1ce3b89b2a689d7ecbb59f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - Python 3.4
  - Wx
  - edgeR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# deg-list-comparison

## Summary

Quantifies the agreement between gene lists produced by different feature-selection or DEG methods (e.g., Wx UGCBs vs. Peng-14, Martinez-7, or edgeR DEG-14-UGCB) by computing set intersections and reporting overlap counts and percentages. This exposes the substantial discrepancy between neural-network-based feature selection and classical DEG ranking on the same TCGA pan-cancer RNA-seq data.

## When to use

Use when you have two or more ranked or unranked gene lists (e.g., Wx DI-ranked UGCBs, edgeR DEGs sorted by adjusted p value, published biomarker signatures) and need to measure their overlap to assess method concordance.

## When NOT to use

- You need to compare the classification performance of the gene sets rather than their membership — use LOOCV or a classifier (NN/SVM/XGBoost) comparison instead.
- The gene lists come from different data sets or different gene identifier schemes without harmonization — symbol-level intersection would be meaningless.
- You expect functional similarity rather than literal gene overlap — use GO/network analysis (e.g., Metascape on top 50 UGCBs) instead.

## Inputs

- Wx top-ranked gene list (genes ranked by average DI score from 10,000 iterations)
- Peng-14-UGCB gene set (14 genes, e.g., KIF4A, NUSAP1, HJURP, NEK2, ...)
- Martinez-7-UGCB gene sets (per-cancer-type 7-gene panels)
- edgeR DEG-14-UGCB gene set (top 14 DEGs sorted by ascending adjusted p value, e.g., LCN1, UMOD, AQP2, PATE4, ...)

## Outputs

- Overlap table recording gene-count and overlapping gene symbols for each baseline set (Peng-14-UGCB, Martinez-7-UGCB, DEG-14-UGCB)
- Overlap percentage summary (e.g., <35% overall with edgeR; 45/500 = 9.0% in top 500)

## How to apply

Load the Wx top-ranked gene list, where genes are ordered by their average discriminative index (DI) score after 10,000 iterations of the Wx algorithm. Load the baseline gene sets for comparison: Peng-14-UGCB, Martinez-7-UGCB, and the top 14 edgeR DEGs (DEG-14-UGCB, sorted by ascending adjusted p value). Compute set intersections between the Wx list and each baseline set using Python set operations (e.g., set(a) & set(b)), then assemble a table recording the overlap gene count and the overlapping gene symbols for each baseline set. Validate the outcome against the reported expectations: zero overlap with Peng-14-UGCB and Martinez-7-UGCB, and under 35% overall overlap with edgeR-selected DEGs (only 45 of the top 500 genes, 9.0%, common between the two algorithms).

## Related tools

- **Python 3.4** (Run set-intersection operations between the Wx gene list and each baseline gene set) — https://github.com/deargen/DearWXpub
- **Wx** (Produces the query top-ranked gene list via DI scores after 10,000 iterations) — https://github.com/deargen/DearWXpub
- **edgeR** (Produces the baseline DEG-14-UGCB list (top 14 DEGs by adjusted p value) from the same gene-expression data)

## Examples

```
overlap = set(wx14) & set(edger14); print(len(overlap), sorted(overlap))
```

## Evaluation signals

- Overlap count against Peng-14-UGCB and Martinez-7-UGCB must be exactly zero, matching the reported result that 'none of our UGCBs overlapped with those identified by Peng et al. or Martinez-Ledesma et al.'
- Overall overlap with edgeR-selected DEGs must be below 35%
- Top-500 overlap with edgeR must reproduce 45 common genes (9.0%)
- The output table must contain both a gene count and gene symbols for each baseline set
- All intersected genes must be valid HGNC-style symbols present in both input lists (e.g., UMOD appears in both Keras Wx 14 and edgeR 14 lists)

## Limitations

- Selected features from the same data set vary depending on the algorithm used, so low overlap does not by itself establish which algorithm is better without experimental verification.
- The comparison is sensitive to the ranking depth chosen (e.g., top 14 vs. top 500 genes); different cutoffs change the overlap percentage.
- The 14-gene Wx signature lacks extensive experimental validation, limiting the biological interpretation of non-overlap.

## Evidence

- [results] none of our UGCBs overlapped with those identified by Peng et al.12 or Martinez-Ledesma et al.13: "none of our UGCBs overlapped with those identified by Peng et al.12 or Martinez-Ledesma et al.13"
- [results] the top 14 DEGs (named DEG-14-UGCB; sorted into ascending order of adjusted p value) identified using a popular DEG analysis method called edgeR: "the top 14 DEGs (named DEG-14-UGCB; sorted into ascending order of adjusted p value) identified using a popular DEG analysis method called edgeR7"
- [discussion] In addition, selected features from the same data set vary depending on algorithms.: "In addition, selected features from the same data set vary depending on algorithms."
- [readme] Peng 14 list from README: "***Peng 14***
`['KIF4A','NUSAP1','HJURP','NEK2','FANCI','DTL','UHRF1','FEN1','IQGAP3','KIF20A','TRIM59','CENPL','C16ORF59','UBE2C']`"
- [results] we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
