---
name: hierarchical-cell-type-marker-selection
description: Use when scRNA-seq data comes with a multi-level cell-type label hierarchy
  and the goal is a minimal marker panel that discriminates all levels jointly (e.g.
  selecting 30–40 markers for the Zeisel mouse cortex labels).
when_to_use_negative:
- The cell labels are not organized in a hierarchy and a simple one-vs-all per-label
  marker ranking is adequate — scGeneFit's advantage lies in jointly recovering a
  label hierarchy with fewer markers.
- Cell types may not be linearly separable in the reduced marker space, or the goal
  requires nonlinear dimension reduction to place unlabeled cells on trajectories
  or spatial locations.
- The goal is exhaustive enumeration of small marker sets ranked one-vs-all (e.g.
  the COMET-style approach) rather than joint optimization of a single panel.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3318
tools:
- name: scGeneFit
  role: 'Primary tool: jointly selects the marker panel that optimizes recovery of
    the hierarchical cell label partition; run in its hierarchical variant with the
    two-layer labels.'
  repo: github.com/solevillar/scGeneFit
- name: COMET
  role: 'Contrasting baseline approach: exhaustively selects sets of k markers ranked
    in a one-vs-all task, representing the per-label strategy scGeneFit improves upon.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:6dd3f1472515b2cb53d6ffd0388e1c84ce03cb664da938ee502f251e93053645
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hierarchical-cell-type-marker-selection

## Summary

Select a compact panel of marker genes that jointly recovers a given hierarchical partition of single-cell labels, using scGeneFit's label-aware compressive classification instead of per-gene one-vs-all ranking.

## When to use

Use when scRNA-seq data comes with a multi-level cell-type label hierarchy and the goal is a minimal marker panel that discriminates all levels jointly (e.g. selecting 30–40 markers for the Zeisel mouse cortex labels).

## When NOT to use

- The cell labels are not organized in a hierarchy and a simple one-vs-all per-label marker ranking is adequate — scGeneFit's advantage lies in jointly recovering a label hierarchy with fewer markers.
- Cell types may not be linearly separable in the reduced marker space, or the goal requires nonlinear dimension reduction to place unlabeled cells on trajectories or spatial locations.
- The goal is exhaustive enumeration of small marker sets ranked one-vs-all (e.g. the COMET-style approach) rather than joint optimization of a single panel.

## Inputs

- scRNA-seq gene expression matrix (e.g. Zeisel mouse cortex/hippocampus data, GEO GSE60361)
- Hierarchical cell-type label annotation for each cell (two-layer cell-type partition)
- Target marker panel size k (e.g. 30 or 40)

## Outputs

- Selected marker gene list of size k per run (e.g. scGeneFit-30, scGeneFit-40)
- First-layer classification error rate on held-out cells per marker panel
- CSV files with the error table and the three selected marker gene lists

## How to apply

Provide the scRNA-seq count matrix together with the full two-layer (or deeper) cell-type label annotation — e.g. the Zeisel mouse cortex/hippocampus dataset, GEO accession GSE60361. Run the hierarchical variant of scGeneFit with the multi-level labels to jointly select a target panel of k markers (k = 30 and k = 40 in the reference benchmark), rather than testing each gene separately; the joint optimization is what allows fewer markers than the number of labels to be sufficient. As a baseline, rank genes by one-vs-all label discrimination and retain the top markers (48 in the reference benchmark). For each panel, classify held-out cells at the first layer of the hierarchy and compare classification error — in the benchmark, hierarchical scGeneFit reached 8.75% error with 30 markers and 6.44% with 40, beating one-vs-all's 9.74% with 48 markers. Write the error table and the selected marker gene lists to CSV for inspection and reuse.

## Related tools

- **scGeneFit** (Primary tool: jointly selects the marker panel that optimizes recovery of the hierarchical cell label partition; run in its hierarchical variant with the two-layer labels.) — github.com/solevillar/scGeneFit
- **COMET** (Contrasting baseline approach: exhaustively selects sets of k markers ranked in a one-vs-all task, representing the per-label strategy scGeneFit improves upon.)

## Evaluation signals

- First-layer classification error on held-out cells should match the benchmark values: 8.75% for scGeneFit-30, 6.44% for scGeneFit-40, 9.74% for one-vs-all-48.
- Hierarchical scGeneFit panels of 30 or 40 markers should classify at least as well as (error ≤) the one-vs-all panel of 48 markers.
- Selected markers must be a jointly optimized set (not a per-gene ranking), and should be more robust and less redundant than one-vs-all markers.
- Output CSVs must contain both the first-layer error table and the three complete marker gene lists.

## Limitations

- Both scGeneFit and one-vs-all methods assume cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions.
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work.
- Marker selection is limited to categorical labels; there is no support for placing unlabeled cells along trajectories or in spatial assays (a manifold constraint is envisioned for the future).

## Evidence

- [abstract] Joint optimization selects markers that recover the hierarchy: "the markers found by our method improves the recovery of the cell type hierarchy with fewer markers than existing methods"
- [abstract] scGeneFit's core selection principle: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [intro] Hierarchical marker selection is the method's design goal: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels."
- [discussion] Linear separability assumption underlying the method: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
- [discussion] No support yet for manifold/trajectory-based marker placement: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at"
