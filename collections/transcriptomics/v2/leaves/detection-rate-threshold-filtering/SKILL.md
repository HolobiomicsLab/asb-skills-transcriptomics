---
name: detection-rate-threshold-filtering
description: Use when constructing cell-to-cell communication networks from single-cell
  expression data and you need to decide which ligands/receptors count as expressed
  in each cell type before edge extraction.
when_to_use_negative:
- Data is bulk expression or proteomics where per-cell detection rates cannot be computed
  — use expression-level thresholds instead
- The goal is to visualise edges below the detection cutoff (e.g. to inspect weakly
  supported interactions), since VisInteractions/ExtractEdges drop pairs whose detection
  rates fall under the threshold
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NATMI
  role: Implements the detection-rate filter via the --detectionThreshold parameter
    (default 0.2) in ExtractEdges.py, gating which ligand-receptor interactions are
    extracted
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:e2ce54645229e4eb340df9907b61cab9493e1367b628807fd5b95cb130e7658a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Detection-rate threshold filtering

## Summary

A gene is treated as expressed in a cell type only if it is detected in more than 20% of that cell type's cells; NATMI applies this detection-rate filter (default 0.2) to both ligand and receptor before computing edge weights, pruning spurious co-expression edges.

## When to use

Use when constructing cell-to-cell communication networks from single-cell expression data and you need to decide which ligands/receptors count as expressed in each cell type before edge extraction.

## When NOT to use

- Data is bulk expression or proteomics where per-cell detection rates cannot be computed — use expression-level thresholds instead
- The goal is to visualise edges below the detection cutoff (e.g. to inspect weakly supported interactions), since VisInteractions/ExtractEdges drop pairs whose detection rates fall under the threshold

## Inputs

- Normalised single-cell expression matrix (genes x cells, e.g. toy.sc.em.txt, CPM-normalised)
- Cell-label metafile mapping each cell barcode to a cell-type annotation (e.g. toy.sc.ann.txt)
- Ligand-receptor pair list (connectomeDB2020 lrc2p or user-supplied)

## Outputs

- Filtered set of ligand-receptor edges passing the 0.2 detection-rate threshold for both partners
- Edge table (edge_weights.csv) with sender, receiver, ligand, receptor and weight columns, containing only detection-passing pairs

## How to apply

Aggregate the normalised (CPM) expression matrix per cell type using the cell-label metafile, then for each gene compute the fraction of cells within a cell type with non-zero counts; a gene passes only if detected in >20% of the cell type's cells. In NATMI this is controlled by the --detectionThreshold parameter of ExtractEdges.py (default 0.2), and an interaction is only extracted if the detection rates of BOTH the ligand and the receptor exceed the threshold. The rationale is to exclude lowly detected, potentially dropout-driven gene calls that would otherwise produce artefactual ligand-receptor edges. Detection filtering is applied prior to computing the mean-expression and specificity edge weights, so edges failing the filter never enter edge_weights output. Users may relax or tighten the threshold via --detectionThreshold depending on dataset depth, but the default 0.2 is the value validated in the NATMI publication.

## Related tools

- **NATMI** (Implements the detection-rate filter via the --detectionThreshold parameter (default 0.2) in ExtractEdges.py, gating which ligand-receptor interactions are extracted) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile toy.sc.em.txt --annFile toy.sc.ann.txt --species mouse --detectionThreshold 0.2 --out outfolder
```

## Evaluation signals

- Every edge in the output table has both ligand and receptor detection rates strictly greater than 0.2 in their respective cell types
- No edge exists for a gene expressed in ≤20% of the relevant cell type's cells
- Number of edges in edge_weights.csv is smaller than the unfiltered ligand-receptor x cell-type-pair enumeration
- Validation edges such as Efnb2-Pecam1 survive the filter and retain their expected high-scoring endothelial-to-endothelial connection

## Limitations

- Predictions are based on mRNA expression and are subject to dropouts, so the 20% cutoff may discard genuinely expressed but poorly captured genes
- The threshold default (0.2) was chosen by the authors; users changing it have no published sensitivity benchmark for their dataset
- Detection rates are computed per input dataset, so filter outcomes are dataset-specific rather than a global reference

## Evidence

- [discussion] 20% detection-rate definition of expression: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold parameter default in NATMI: "do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] ExtractEdges.py command that applies the filter: [evidence span withheld — anchor status: not_found]
