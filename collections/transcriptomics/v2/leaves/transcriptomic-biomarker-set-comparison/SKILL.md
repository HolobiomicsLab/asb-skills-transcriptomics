---
name: transcriptomic-biomarker-set-comparison
description: Use when two or more gene/biomarker candidate lists from the same transcriptomic
  cohort (e.g., TCGA pan-cancer RNA-seq) need intersection analysis to quantify agreement
  between feature-selection algorithms.
when_to_use_negative:
- The goal is to compare classification accuracy of gene sets (LOOCV or NN/SVM accuracy)
  rather than gene membership overlap
- Input is a single gene list with no reference set to compare against
- Gene lists come from different identifiers or genome annotations that have not been
  harmonized to common symbols
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Wx
  role: Produces the query top-ranked gene list via average DI score over 10,000 iterations
  repo: https://github.com/deargen/DearWXpub
- name: edgeR
  role: Generates the DEG-14-UGCB reference gene set for overlap comparison
- name: Python 3.4
  role: Performs set intersection computations on gene symbol lists
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:0f90af860d5b9b09945cc7c73d5f31a5408713bbfe101dd522fa31bb6cf4760f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - Python 3.4
  - Wx
  - edgeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# transcriptomic-biomarker-set-comparison

## Summary

Quantify overlap between an algorithm-selected gene set (e.g., Wx-14-UGCB ranked by average DI score over 10,000 iterations) and reference biomarker/DEG lists (Peng-14-UGCB, Martinez-7-UGCB, edgeR DEG-14-UGCB) using set operations, to assess whether alternative feature-selection methods converge on the same genes.

## When to use

Use when two or more gene/biomarker candidate lists from the same transcriptomic cohort (e.g., TCGA pan-cancer RNA-seq) need intersection analysis to quantify agreement between feature-selection algorithms.

## When NOT to use

- The goal is to compare classification accuracy of gene sets (LOOCV or NN/SVM accuracy) rather than gene membership overlap
- Input is a single gene list with no reference set to compare against
- Gene lists come from different identifiers or genome annotations that have not been harmonized to common symbols

## Inputs

- Wx top-ranked gene list ranked by average DI score (10,000 iterations)
- Peng-14-UGCB gene list (e.g., KIF4A, NUSAP1, HJURP, NEK2, ...)
- Martinez-7-UGCB gene lists per cancer type (e.g., BLCA: SMAD2, RUNX2, ...)
- edgeR DEG-14-UGCB gene list sorted by ascending adjusted p value

## Outputs

- Overlap count per baseline gene set
- List of overlapping gene symbols per set
- Summary table of pairwise intersections (e.g., 45/500 = 9.0% for Wx vs edgeR; 0 for Wx vs Peng/Martinez)

## How to apply

Load the Wx top-ranked gene list, ordered by average discriminative index (DI) score after 10,000 iterations of the Wx algorithm, plus the baseline sets: Peng-14-UGCB, Martinez-7-UGCB, and edgeR DEG-14-UGCB (top 14 DEGs sorted by ascending adjusted p value). Compute set intersections with Python set operations (gene symbols from the DearWXpub README, e.g., edgeR-14 genes 'LCN1','UMOD','AQP2',...). Assemble a table of overlap counts and overlapping gene symbols per baseline set. Interpret: substantial discrepancy is expected — zero overlap with Peng/Martinez UGCBs and under ~35% overall overlap with edgeR, with only 45 of the top 500 Wx genes (9.0%) common to edgeR selections.

## Related tools

- **Wx** (Produces the query top-ranked gene list via average DI score over 10,000 iterations) — https://github.com/deargen/DearWXpub
- **edgeR** (Generates the DEG-14-UGCB reference gene set for overlap comparison)
- **Python 3.4** (Performs set intersection computations on gene symbol lists)

## Examples

```
overlap = set(wx_top_genes) & set(edger14_genes); print(len(overlap), sorted(overlap))
```

## Evaluation signals

- Overlap with Peng-14-UGCB and Martinez-7-UGCB must be zero genes
- Overall overlap with edgeR DEGs should be below ~35%; for the top 500 Wx genes, 45 shared genes (9.0%) is the reference value
- All gene symbols in the comparison table must resolve to valid symbols present in the source lists
- Reference edgeR-14 set should match the documented list: LCN1, UMOD, AQP2, PATE4, SLC12A1, OTOP2, ACTN3, KRT36, ATP2A1, PRH2, AGER, PYGM, PRR4, ESRRB

## Limitations

- Zero or low overlap does not by itself prove which algorithm is better — selected features from the same data set vary depending on the algorithm used, and experimental verification is needed
- Selected genes can differ with the neural network backend/framework used (README notes Keras vs naive TensorFlow), so overlap results are implementation-sensitive
- The Wx-14-UGCB signature lacks extensive experimental validation

## Evidence

- [results] Wx UGCBs share no genes with prior marker sets: "none of our UGCBs overlapped with those identified by Peng et al.12 or Martinez-Ledesma et al.13"
- [results] Wx ranking procedure that produces the query list: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score"
- [readme] Reference edgeR gene list from README: "***edgeR 14***
`['LCN1','UMOD','AQP2','PATE4','SLC12A1','OTOP2','ACTN3','KRT36','ATP2A1','PRH2','AGER','PYGM','PRR4','ESRRB']`"
- [readme] Framework sensitivity of selected features: "Selected Features can be different as which backend learning framework used"
