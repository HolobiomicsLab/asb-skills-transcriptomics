---
name: gene-set-overlap-analysis
description: Use when comparing a ranked or selected gene list against published biomarker
  signatures or DEG lists (e.g., top-14 UGCB sets) to report overlap counts, percentages,
  and shared gene symbols.
when_to_use_negative:
- When the goal is functional enrichment rather than direct gene-identity overlap
  (the article used Metascape GO/network analysis for function of the top 50 UGCBs,
  which is a separate step)
- When the input is a ranked continuous score list without gene symbols that can be
  matched against the reference sets
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Wx
  role: Produces the query top-ranked gene list by average DI score used as one side
    of each set intersection
  repo: https://github.com/deargen/DearWXpub
- name: edgeR
  role: Source of the DEG-14-UGCB baseline gene set (top 14 DEGs by ascending adjusted
    p value) compared against the Wx list
- name: Python 3.4
  role: Language used to compute set intersections between the Wx gene list and each
    baseline set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:2e6c7ea873e6555311e0a2f05582e54557a18e99a052ebc12d6165337a7dff3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - Python 3.4
  - Wx
  - edgeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-set-overlap-analysis

## Summary

Quantify the overlap between a newly ranked gene list (e.g., Wx top genes by average DI score) and external reference gene sets (Peng-14-UGCB, Martinez-7-UGCB, edgeR DEG-14-UGCB) using Python set operations, to establish how much a feature-selection method agrees with or diverges from prior biomarker/DEG approaches.

## When to use

Use when comparing a ranked or selected gene list against published biomarker signatures or DEG lists (e.g., top-14 UGCB sets) to report overlap counts, percentages, and shared gene symbols.

## When NOT to use

- When the goal is functional enrichment rather than direct gene-identity overlap (the article used Metascape GO/network analysis for function of the top 50 UGCBs, which is a separate step)
- When the input is a ranked continuous score list without gene symbols that can be matched against the reference sets

## Inputs

- Wx top-ranked gene list ordered by average DI score after 10,000 iterations
- Peng-14-UGCB reference gene set (e.g., KIF4A, NUSAP1, HJURP, NEK2, FANCI, DTL, UHRF1, FEN1, IQGAP3, KIF20A, TRIM59, CENPL, C16ORF59, UBE2C)
- Martinez-7-UGCB per-cancer-type gene sets
- edgeR DEG-14-UGCB list sorted by ascending adjusted p value (e.g., LCN1, UMOD, AQP2, PATE4, SLC12A1, OTOP2, ACTN3, KRT36, ATP2A1, PRH2, AGER, PYGM, PRR4, ESRRB)

## Outputs

- Table of overlap gene count and overlapping gene symbols per baseline set
- Overlap percentage (e.g., 45/500 = 9.0% for Wx vs edgeR top 500)
- Binary overlap validation flags (zero vs. non-zero intersection per set)

## How to apply

Load the query gene list ranked by its selection metric (e.g., Wx genes ranked by average discriminative index (DI) score after 10,000 algorithm iterations). Load each baseline set: Peng-14-UGCB, Martinez-7-UGCB, and the top 14 edgeR DEGs (DEG-14-UGCB, sorted by ascending adjusted p value). Compute set intersections between the query list and each baseline set using Python set operations, then assemble a table recording the overlap gene count and the overlapping gene symbols for each comparison. Validate by checking the results against expected magnitudes: here, zero overlap with Peng-14-UGCB and Martinez-7-UGCB, and less than 35% overall overlap (45 of the top 500 genes, 9.0%) with the edgeR DEG set, indicating substantial discrepancy between feature-selection methods. The README supplies concrete gene symbols for each set (e.g., Wx-14: EEF1A1, FN1, GAPDH, ...; edgeR-14: LCN1, UMOD, AQP2, ...) and even Wx ranking positions for Peng/edgeR genes, which can be used to reproduce intersection checks.

## Related tools

- **Wx** (Produces the query top-ranked gene list by average DI score used as one side of each set intersection) — https://github.com/deargen/DearWXpub
- **edgeR** (Source of the DEG-14-UGCB baseline gene set (top 14 DEGs by ascending adjusted p value) compared against the Wx list)
- **Python 3.4** (Language used to compute set intersections between the Wx gene list and each baseline set)

## Examples

```
wx_top = set(['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']); edgr = set(['LCN1','UMOD','AQP2','PATE4','SLC12A1','OTOP2','ACTN3','KRT36','ATP2A1','PRH2','AGER','PYGM','PRR4','ESRRB']); print(len(wx_top & edgr), sorted(wx_top & edgr))
```

## Evaluation signals

- Intersection with Peng-14-UGCB and Martinez-7-UGCB must equal zero genes
- Overlap with the edgeR top genes must remain below 35% overall; specifically 45 of the top 500 genes (9.0%) shared
- Every reported overlapping gene symbol must appear in both the query list and the corresponding baseline set (set-membership check)
- Query list must be sorted by average DI score from the 10,000-iteration Wx run before taking the top-N subset
- The edgeR DEG-14-UGCB input must be ordered by ascending adjusted p value

## Limitations

- None of the Wx UGCBs overlapped with Peng et al. or Martinez-Ledesma et al., so overlap analysis alone cannot establish which algorithm is better without experimental verification
- Selected features from the same data set vary depending on the algorithm and even on the neural-network backend (the README notes the Keras implementation can select different features than the original TensorFlow paper)
- The Wx-14-UGCB gene signature lacks extensive experimental validation, limiting interpretation of overlap-based comparisons

## Evidence

- [results] None of the Wx UGCBs overlapped with those identified by Peng et al. or Martinez-Ledesma et al.: [evidence span withheld — anchor status: not_found]
- [results] Iterated Wx selection: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score"
- [results] edgeR baseline definition: "the top 14 DEGs (named DEG-14-UGCB; sorted into ascending order of adjusted p value) identified using a popular DEG analysis method called edgeR"
- [readme] Keras Wx 14 gene list: "`['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']`"
- [readme] Backend affects feature selection: "Selected Features can be different as which backend learning framework used"
