---
name: top-ranked-pair-comparison-across-weightings
description: Use when you need to show that a cell-communication network's top-ranked
  ligand–receptor pairs depend on the edge-weighting metric (expression vs. specificity),
  e.g. to expose housekeeping-edge inflation in expression-ranked networks.
when_to_use_negative:
- Your ligand–receptor pairs involve heteromeric complexes — NATMI does not model
  heteromeric complexes, heterodimerization, or co-receptor requirements
- You need downstream signalling consequences or spatial-context-aware communication
  predictions — NATMI is not designed for these (use NicheNet, SoptSC, or spatial
  tools instead)
- You need global (cross-dataset) specificity values — the specificity weights NATMI
  computes are local/dataset-specific and a Human Cell Atlas-based global reference
  was not yet available
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NATMI
  role: Run ExtractEdges.py on the expression matrix and annotation file to extract
    all ligand–receptor edges with mean-expression and specificity weight columns
    for subsequent ranking and top-20 comparison
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:38603cbbea4e471d9f28b669fb66ebb91c5425b70629f0fe76ecfd8403eb67c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# top-ranked-pair-comparison-across-weightings

## Summary

A NATMI-based analysis skill that ranks ligand–receptor communication edges by two alternative edge weights (mean expression vs. specificity) and compares the resulting top-20 lists, demonstrating that weighting choice substantially changes which ligand–receptor pairs and cell-type pairs appear most important.

## When to use

Use when you need to show that a cell-communication network's top-ranked ligand–receptor pairs depend on the edge-weighting metric (expression vs. specificity), e.g. to expose housekeeping-edge inflation in expression-ranked networks.

## When NOT to use

- Your ligand–receptor pairs involve heteromeric complexes — NATMI does not model heteromeric complexes, heterodimerization, or co-receptor requirements
- You need downstream signalling consequences or spatial-context-aware communication predictions — NATMI is not designed for these (use NicheNet, SoptSC, or spatial tools instead)
- You need global (cross-dataset) specificity values — the specificity weights NATMI computes are local/dataset-specific and a Human Cell Atlas-based global reference was not yet available

## Inputs

- CPM-normalized single-cell gene expression matrix (genes x cells, e.g. expression.matrix.txt)
- Cell-annotation metafile mapping each cell barcode to a cell-type label (e.g. annotation.txt)
- connectomeDB2020 (lrc2p) ligand–receptor interaction list with 2,293 manually curated, literature-supported pairs

## Outputs

- Extracted edge list with one row per ligand–receptor pair per communicating cell-type pair, carrying both mean-expression and specificity weight columns (126,738 edges for the Skelly cardiac dataset)
- Top-20 ligand–receptor pairs ranked by mean-expression weight
- Top-20 ligand–receptor pairs ranked by specificity weight
- Rank-change comparison between the two weightings (Fig. 1-style summary)

## How to apply

Load a CPM-normalized single-cell expression matrix (normalized by total UMI count and multiplied by 1,000,000) plus a cell-annotation metafile mapping each barcode to one of the cell types (e.g. the 12 Skelly et al. cardiac cell types). Run NATMI ExtractEdges.py with the connectomeDB2020 ligand–receptor list (lrc2p, 2,293 literature-supported pairs) using --weightType mean and the default detection threshold of 0.2, so a gene counts as expressed only if detected in more than 20% of cells of a cell type. Verify the extracted edge list (126,738 predicted edges for the Skelly cardiac dataset, matching Supplementary Data 2) contains both a mean-expression weight and a specificity weight column for every edge. Sort the full edge table independently by each weight column and extract the top-20 ligand–receptor pairs per ranking, then summarize rank changes between the two weightings as in Fig. 1 of the manuscript. The rationale is that specificity weighting down-weights ubiquitous housekeeping edges, so expression-ranked top pairs span a broader range of communicating cell-type pairs and are less informative.

## Related tools

- **NATMI** (Run ExtractEdges.py on the expression matrix and annotation file to extract all ligand–receptor edges with mean-expression and specificity weight columns for subsequent ranking and top-20 comparison) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --interDB lrc2p --weightType mean --detectionThreshold 0.2
```

## Evaluation signals

- The reproduced edge count equals 126,738 for the Skelly et al. cardiac dataset, matching Supplementary Data 2
- Every edge in the extracted table carries both a mean-expression weight and a specificity weight column
- Genes are treated as expressed only when detected in more than 20% of cells of a cell type (detectionThreshold 0.2)
- The expression-ranked top-20 pairs differ substantially from the specificity-ranked top-20 pairs, with expression-ranked pairs spanning a broader range of communicating cell-type pairs
- Expression matrix is CPM-normalized (each profile divided by total UMI count and multiplied by 1,000,000) before edge extraction

## Limitations

- Specificity weights are calculated from the input dataset and are local/dataset-specific, not global; global specificities based on the Human Cell Atlas were deferred to future NATMI versions
- Predictions are based on mRNA expression levels rather than protein concentration and are subject to dropouts
- No estimates of binding kinetics for ligand–receptor pairs; heteromeric complexes, heterodimerization, and co-receptor requirements are not modeled
- Predicted networks rely on the original authors' cell annotations, which can affect accuracy

## Evidence

- [methods] detection threshold default: "do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] 20% expression rule: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [discussion] local specificity caveat: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [discussion] ExtractEdges command: [evidence span withheld — anchor status: not_found]
- [readme] connectomeDB2020 database: "NATMI uses connectomeDB2020 (the most up-to-date manually curated ligand-receptor interaction list)"
