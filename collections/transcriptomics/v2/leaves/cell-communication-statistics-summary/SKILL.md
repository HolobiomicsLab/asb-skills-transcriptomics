---
name: cell-communication-statistics-summary
description: Use when a single-cell atlas (e.g. Tabula Muris, ~117 cell types) has
  been processed with NATMI and you need organism-wide summary metrics of autocrine
  signalling, such as the median number of co-detected ligand–receptor pairs per cell
  type.
when_to_use_negative:
- The goal is spatially-resolved communication inference — NATMI 'has not been specifically
  designed for spatial data'
- Downstream signalling consequences or binding kinetics are required — NATMI does
  not predict these and has 'no estimates of the binding kinetics for each ligand–receptor
  pair'
- Ligand–receptor pairs involve heteromeric complexes — NATMI does not model heteromeric
  complexes, heterodimerization or co-receptor requirements
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: NATMI
  role: ExtractEdges.py computes ligand–receptor edges between cell types with expression/detection
    thresholds and specificity weights, from which autocrine (self-signalling) edges
    are aggregated into summary statistics
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:e6cdaf342f1cf22fac12d2cd37995d2ead2d356361def5858a8a78dce20e33d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-communication-statistics-summary

## Summary

Aggregates NATMI-extracted ligand–receptor edges into summary statistics of autocrine/self-signalling potential — e.g. median co-detected connectomeDB2020 pairs per cell type and the fraction of the 2,293 curated pairs co-detected in at least one cell type — to quantify how much predicted communication occurs within individual cells rather than between them.

## When to use

Use when a single-cell atlas (e.g. Tabula Muris, ~117 cell types) has been processed with NATMI and you need organism-wide summary metrics of autocrine signalling, such as the median number of co-detected ligand–receptor pairs per cell type.

## When NOT to use

- The goal is spatially-resolved communication inference — NATMI 'has not been specifically designed for spatial data'
- Downstream signalling consequences or binding kinetics are required — NATMI does not predict these and has 'no estimates of the binding kinetics for each ligand–receptor pair'
- Ligand–receptor pairs involve heteromeric complexes — NATMI does not model heteromeric complexes, heterodimerization or co-receptor requirements

## Inputs

- Single-cell gene expression matrix (gene symbols, normalised or normalisable to CPM)
- Cell-type annotation metafile mapping each cell to one of ~117 Tabula Muris cell types
- connectomeDB2020 ligand–receptor pair list (2,293 curated pairs; lrc2p format)

## Outputs

- Per-cell-type autocrine edge table with summed-specificity weights
- Summary statistic: median number of co-detected ligand–receptor pairs per cell type (~54 for Tabula Muris)
- Total count of connectomeDB2020 pairs co-detected in ≥1 cell type at 10 CPM / 20% detection thresholds (719/2,293, ~31%)
- Ranked comparison of autocrine vs. intra-organ and inter-organ edges by summed specificity

## How to apply

Normalise each expression profile to counts per million by dividing by total UMI counts and multiplying by 1,000,000, then run NATMI's ExtractEdges.py with --species mouse and the lrc2p/connectomeDB2020 pair list (2,293 literature-supported pairs). Retain edges passing NATMI's default detectionThreshold of 0.2 (gene expressed if detected in >20% of a cell type's cells) together with a 10 CPM expression threshold. Classify edges where source and target cell type are identical as autocrine (self-signalling) and aggregate the number of co-detected pairs per cell type. Rank edges by summed-specificity weight to confirm that autocrine edges rank comparably to (or above) intra-organ and inter-organ edges. Accept the summary if the median co-detected pairs per cell type is ~54 and ~719 of 2,293 pairs (~31%) are co-detected in at least one cell type.

## Related tools

- **NATMI** (ExtractEdges.py computes ligand–receptor edges between cell types with expression/detection thresholds and specificity weights, from which autocrine (self-signalling) edges are aggregated into summary statistics) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --species mouse --interDB lrc2p --coreNum 4
```

## Evaluation signals

- Median co-detected ligand–receptor pairs per cell type is ~54 (Tabula Muris reference value)
- Co-detected pair count is 719 of 2,293 connectomeDB2020 pairs (~31%)
- All reported pairs pass the detection threshold of >20% of cells and the 10 CPM expression threshold
- Autocrine edges show summed-specificity ranks comparable to or higher than intra-organ and inter-organ edges
- Edge table schema matches NATMI ExtractEdges.py output (ligand, receptor, source, target, expression, detection rate, specificity columns)

## Limitations

- Specificity weights are local/dataset-specific, not global; a global specificity reference based on the Human Cell Atlas is deferred to future NATMI versions
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts
- Networks rely on the original authors' cell annotations, which can affect accuracy of predicted networks
- Highlighted edges are based on experiments without replicates

## Evidence

- [abstract] connectomeDB2020 pair universe: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] normalisation and co-expression finding: "normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000"
- [methods] detection threshold parameter: "do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [readme] autocrine signalling confirmation via Tabula Muris: "we confirm our previous predictions from bulk data that autocrine signalling is a major feature of cell-to-cell communication networks"
- [abstract] self-signalling via co-expressed pairs: "hundreds of ligands and their cognate receptors are co-expressed in individual cells suggesting a substantial potential for self-signalling"
