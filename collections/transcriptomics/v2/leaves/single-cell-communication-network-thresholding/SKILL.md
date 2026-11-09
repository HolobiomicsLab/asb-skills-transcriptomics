---
name: single-cell-communication-network-thresholding
description: Use when building or visualizing a cell-to-cell communication network
  from single-cell expression data plus cell-type labels and you must decide which
  ligand–receptor edges pass detection (e.g. >20% of cells), specificity, and expression
  cutoffs before drawing.
when_to_use_negative:
- Input is spatial transcriptomics data — NATMI has not been specifically designed
  for spatial data
- The analysis requires modelling heteromeric ligand/receptor complexes, co-receptor
  requirements, or downstream signalling consequences — NATMI does not model these
  (use tools like CellPhoneDB/NicheNet instead)
- Estimates of ligand–receptor binding kinetics are required — NATMI provides none
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Runs ExtractEdges.py to compute per-pair expression/specificity edge weights
    and VisInteractions.py to enforce detection/specificity/expression thresholds
    and draw the filtered communication network
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:404c00621afee856299071193e1d78134294b83e51b7b176d606ace6e03e4740
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-communication-network-thresholding

## Summary

Applies NATMI's expression, detection, and specificity cutoffs when constructing and drawing ligand–receptor-mediated cell-to-cell communication networks, so that only edges supported by co-expressed ligand–receptor pairs in defined cell types are retained. This mirrors the manuscript's Tabula Muris workflow (detection threshold 0.2, specificity/expression thresholds 0) used to identify highly communicating cell pairs and communities.

## When to use

Use when building or visualizing a cell-to-cell communication network from single-cell expression data plus cell-type labels and you must decide which ligand–receptor edges pass detection (e.g. >20% of cells), specificity, and expression cutoffs before drawing.

## When NOT to use

- Input is spatial transcriptomics data — NATMI has not been specifically designed for spatial data
- The analysis requires modelling heteromeric ligand/receptor complexes, co-receptor requirements, or downstream signalling consequences — NATMI does not model these (use tools like CellPhoneDB/NicheNet instead)
- Estimates of ligand–receptor binding kinetics are required — NATMI provides none

## Inputs

- Gene expression matrix (counts, normalized to CPM/UMI-per-million) with official gene symbols
- Cell labels metafile mapping each cell to a cell-type annotation
- Ligand–receptor pair list (connectomeDB2020 or user-supplied, e.g. lrc2p)

## Outputs

- Per-pair edge weight tables (expression, specificity) from ExtractEdges.py
- Filtered/visualized network PDF (e.g. network_total-specificity-based_layout_circle.pdf) containing only edges passing detectionThreshold/specificityThreshold/expressionThreshold
- Ranked cell-pair summary edges (summed specificity weights) identifying highly communicating cell-type pairs/communities

## How to apply

Normalise expression to counts per million (divide each profile by total UMIs/reads and multiply by 1,000,000), then run ExtractEdges.py with the lrc2p/connectomeDB2020 ligand–receptor list to compute per-pair expression and specificity edge weights. Apply the detection rule — a gene is considered expressed in a cell type if more than 20% of cells of that type have non-zero counts — via VisInteractions.py's --detectionThreshold 0.2, which drops interactions whose ligand and receptor detection rates fall below the cutoff. Set --specificityThreshold and --expressionThreshold (both 0 in the Tabula Muris workflow) to further exclude edges, and optionally --keepTopEdge n to draw only the top n interactions passing thresholds. Aggregate edges sharing sending/receiving cell types into summary edges whose weights are the sum of ligand–receptor specificity weights, then rank to identify the top communicating cell-type pairs/communities (e.g. cardiac Nppa–Npr3, Thpo–Mpl hepatocyte-to-megakaryocyte-erythroid).

## Related tools

- **NATMI** (Runs ExtractEdges.py to compute per-pair expression/specificity edge weights and VisInteractions.py to enforce detection/specificity/expression thresholds and draw the filtered communication network) — https://github.com/forrest-lab/NATMI/

## Examples

```
python VisInteractions.py --sourceFolder edges_folder --interDB lrc2p --weightType mean --detectionThreshold 0.2 --specificityThreshold 0 --expressionThreshold 0 --drawNetwork y --layout circle --edgeWidth 6
```

## Evaluation signals

- Edges drawn in the network all have ligand and receptor detection rates ≥ 0.2 in their respective cell types
- Generated network file (e.g. Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean/network_total-specificity-based_layout_circle.pdf) exists and its top summary edges contain the reported ligand–receptor assignments (Nppa–Npr3, Sst–Sstr1/Sstr5, Thpo–Mpl, fibrinogen–integrin)
- Summary edge weights equal the sum of the constituent ligand–receptor specificity weights for each sending/receiving cell-type pair
- Edges excluded by --specificityThreshold/--expressionThreshold are absent from the visualization, matching the parameter values used

## Limitations

- Specificity weights are dataset-specific (local), not global; global specificities were deferred to future NATMI versions
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts
- Edges highlighted in the manuscript are based on experiments without replicates; results depend on the original authors' cell annotations

## Evidence

- [discussion] Detection rule (>20% cells) grounding the 0.2 detection threshold: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold parameter semantics: "--detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] specificity/expression threshold parameters: "--specificityThreshold SPECIFICITYTHRESHOLD do not draw the edges whose specificities are not greater than the threshold (default 0)"
- [methods] CPM normalization step preceding thresholding: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
- [intro] network summarization to identify highly communicating cell pairs/communities: "summarise the entire communication network to show how strongly or specifically each cell type in a complex sample communicates to every other cell type, thus identifying highly communicating cell"
