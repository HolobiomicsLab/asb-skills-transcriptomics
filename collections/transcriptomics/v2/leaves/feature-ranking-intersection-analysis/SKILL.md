---
name: feature-ranking-intersection-analysis
description: Use when a ranked feature/gene list (e.g. Wx top genes by average DI
  score) must be compared against baseline gene sets such as Peng-14-UGCB, Martinez-7-UGCB,
  or edgeR DEG-14-UGCB to quantify gene-level agreement.
when_to_use_negative:
- The input is a raw gene-expression count matrix rather than a ranked gene list —
  run Wx (or edgeR for the DEG baseline) first to produce rankings before intersecting.
- 'The goal is to determine which feature-selection algorithm performs better: low
  or zero overlap does not establish superiority, and the authors state it is ''difficult
  to establish which algorithm is better by comparison without experimental verification''
  — use classification accuracy (e.g. LOOCV or NN evaluation) instead.'
- Intersecting gene lists generated under different computational backends (e.g. original
  TensorFlow vs. Keras Wx implementations), since selected features can differ depending
  on the backend learning framework.
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Wx
  role: Neural network-based feature selection algorithm that produces the DI-ranked
    gene list being intersected against the baseline sets
  repo: https://github.com/deargen/DearWXpub
- name: edgeR
  role: Differential expression method used to build the DEG-14-UGCB baseline set
    (top 14 DEGs by ascending adjusted p value)
- name: Python
  role: Used to compute set intersections between the Wx gene list and each baseline
    gene set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:21e37a2c348ab9ea2cb6e58a1406f509fc01529524da1d196a40dc58180b5ab8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - Wx
  - edgeR
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# feature-ranking-intersection-analysis

## Summary

Set-intersection analysis that quantifies how much a Wx-ranked gene list (genes ordered by average discriminative index (DI) score after 10,000 iterations) overlaps with benchmark biomarker sets (Peng-14-UGCB, Martinez-7-UGCB) and edgeR DEGs, revealing that different feature-selection methods yield largely non-overlapping gene sets.

## When to use

Use when a ranked feature/gene list (e.g. Wx top genes by average DI score) must be compared against baseline gene sets such as Peng-14-UGCB, Martinez-7-UGCB, or edgeR DEG-14-UGCB to quantify gene-level agreement.

## When NOT to use

- The input is a raw gene-expression count matrix rather than a ranked gene list — run Wx (or edgeR for the DEG baseline) first to produce rankings before intersecting.
- The goal is to determine which feature-selection algorithm performs better: low or zero overlap does not establish superiority, and the authors state it is 'difficult to establish which algorithm is better by comparison without experimental verification' — use classification accuracy (e.g. LOOCV or NN evaluation) instead.
- Intersecting gene lists generated under different computational backends (e.g. original TensorFlow vs. Keras Wx implementations), since selected features can differ depending on the backend learning framework.

## Inputs

- Wx top-ranked gene list (genes ranked by average DI score after 10,000 Wx algorithm iterations)
- Peng-14-UGCB universal gene-expression cancer biomarker candidate set (14 genes)
- Martinez-7-UGCB gene set (per-cancer-type sets of 7 genes)
- edgeR DEG-14-UGCB: top 14 differentially expressed genes sorted in ascending order of adjusted p value

## Outputs

- Overlap table recording the overlap gene count and overlapping gene symbols for each baseline set (Peng-14-UGCB, Martinez-7-UGCB, DEG-14-UGCB)

## How to apply

Load the Wx top-ranked gene list, where genes are ranked by average DI score after 10,000 iterations of the Wx algorithm on the TCGA pan-cancer RNA-seq cohort. Load the baseline comparison sets: Peng-14-UGCB, Martinez-7-UGCB, and the top 14 DEGs (DEG-14-UGCB) identified with edgeR and sorted in ascending order of adjusted p value. Compute set intersections between the Wx list and each baseline set using Python set operations (e.g. set intersection of the top 500 Wx genes with each baseline set). Assemble a table recording the overlap gene count and the overlapping gene symbols for each baseline set. Validate against the reported results: expect zero overlap with Peng-14-UGCB and Martinez-7-UGCB, and less than 35% overall overlap with edgeR genes — only 45 of the top 500 genes (9.0%) were common between the two algorithms.

## Related tools

- **Wx** (Neural network-based feature selection algorithm that produces the DI-ranked gene list being intersected against the baseline sets) — https://github.com/deargen/DearWXpub
- **edgeR** (Differential expression method used to build the DEG-14-UGCB baseline set (top 14 DEGs by ascending adjusted p value))
- **Python** (Used to compute set intersections between the Wx gene list and each baseline gene set)

## Examples

```
overlap = set(wx_ranked_genes[:500]) & set(edger14_genes); print(len(overlap), sorted(overlap))
```

## Evaluation signals

- Overlap count against Peng-14-UGCB and Martinez-7-UGCB must be exactly zero, matching the article's reported result.
- Overall overlap with edgeR-selected genes must be below 35%.
- Intersection of the top 500 Wx genes with the edgeR gene set must equal 45 genes (9.0%).
- The overlap table must list concrete overlapping gene symbols (e.g. 'UMOD', shared between the Wx Keras-14 list and the edgeR-14 list) rather than counts alone.
- Recomputing intersections with independent Python set operations must reproduce identical overlap counts and gene symbol sets.

## Limitations

- Selected features from the same data set vary depending on the algorithm used, so zero overlap does not invalidate either method — Wx-14-UGCB still showed higher classification accuracy than Peng-14-UGCB for five out of seven cancer types.
- Selected features can differ depending on the backend learning framework (original TensorFlow vs. Keras in the DearWXpub repo) and because some TCGA samples were updated, so cross-implementation intersections are not directly comparable.
- Overlap analysis alone cannot rank methods; the edgeR-14 set still achieved 94.81% total classification accuracy in the README benchmark despite low gene-level overlap with Wx.

## Evidence

- [results] None of the Wx UGCBs overlapped with those identified by Peng et al. or Martinez-Ledesma et al.: "none of our UGCBs overlapped with those identified by Peng et al.12 or Martinez-Ledesma et al.13"
- [results] edgeR DEG baseline construction: "the top 14 DEGs (named DEG-14-UGCB; sorted into ascending order of adjusted p value) identified using a popular DEG analysis method called edgeR7"
- [abstract] Wx ranking basis: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [readme] Backend-dependent feature selection: "Selected Features can be different as which backend learning framework used"
