---
name: delta-network-differential-analysis
description: Use when comparing two single-cell expression conditions (e.g. 3- vs
  18-month mammary gland CPM-normalized matrices) to identify which ligand-receptor
  edges are gained, lost, or fold-changed between them.
when_to_use_negative:
- When the goal is predicting downstream signalling consequences or spatially-resolved
  communication — NATMI is not designed for spatial data and does not model downstream
  effects (use NicheNet/SpaOTsc instead)
- When ligand-receptor pairs require heteromeric complex or co-receptor modelling
  — NATMI does not model heterodimerization or co-receptor requirements
- When input is a bulk-only dataset without cell-type labels and a single condition
  — a delta network requires two ExtractEdges.py network folders from labelled cell
  populations
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Python toolkit providing DiffEdges.py for the two-condition edge comparison
    and VisInteractions.py for rendering the red/blue delta network
  repo: https://github.com/forrest-lab/NATMI/
- name: connectomeDB2020
  role: Manually curated ligand-receptor pair database (2293 pairs, lrc2p build) against
    which both condition networks are built so edges are comparable
  repo: https://github.com/forrest-lab/NATMI/
- name: Python
  role: Runtime for ExtractEdges.py, DiffEdges.py and VisInteractions.py (tested on
    Python 2.7.17 and 3.7.6 with pandas, seaborn, igraph, NetworkX, PyGraphviz)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:2917ec31614e1c11c9cf7e96d1a905fefe1541e6307e2c1df54f250e78007631
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  - Python
  - connectomeDB2020
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# delta-network-differential-analysis

## Summary

Differential (delta) network analysis of cell-to-cell communication between two conditions using NATMI's DiffEdges.py, which identifies ligand-receptor edge-weight changes between paired ExtractEdges.py output folders, as demonstrated on the Tabula Muris Senis mammary gland at 3 vs 18 months.

## When to use

Use when comparing two single-cell expression conditions (e.g. 3- vs 18-month mammary gland CPM-normalized matrices) to identify which ligand-receptor edges are gained, lost, or fold-changed between them.

## When NOT to use

- When the goal is predicting downstream signalling consequences or spatially-resolved communication — NATMI is not designed for spatial data and does not model downstream effects (use NicheNet/SpaOTsc instead)
- When ligand-receptor pairs require heteromeric complex or co-receptor modelling — NATMI does not model heterodimerization or co-receptor requirements
- When input is a bulk-only dataset without cell-type labels and a single condition — a delta network requires two ExtractEdges.py network folders from labelled cell populations

## Inputs

- CPM-normalized gene expression matrices for each condition (e.g. 3m.upm.em.csv, 18m.upm.em.csv, counts per million UMIs)
- Cell-type annotation/metafile per condition mapping each cell to a label (e.g. 3m.ann.csv)
- connectomeDB2020 (lrc2p) ligand-receptor pair list, mapped to mouse via HomoloGene homologs
- ExtractEdges.py output folders for both conditions (3m.mg, 18m.mg)

## Outputs

- DiffEdges.py delta-network output folder (e.g. 3m-18m) with edge-level fold changes between conditions
- Summary table delta_edges_3m-18m.csv with counts: 2045 shared, 1247 3-month-only, 340 18-month-only edges
- Delta network plot network_total-specificity-based_layout_circle.pdf with red (reference-higher) / blue (target-higher) edges
- Condition-specific edge fractions (e.g. 49.2% of 3-month-only edges to/from basal cells; 78.2% of 18-month-only edges involving T and B cells)

## How to apply

Normalize each condition's expression matrix to counts per million (divide by total UMIs, multiply by 1,000,000) and run ExtractEdges.py per condition against connectomeDB2020 (lrc2p) using the species-specific homologs (mouse via NCBI HomoloGene). Then run DiffEdges.py with the 3-month folder as --refFolder and the 18-month folder as --targetFolder and the mean expression weight type to classify each edge as shared, reference-only, or target-only, with fold-change direction encoded as red (reference-higher) or blue (target-higher). Tabulate the differential edge counts (expected 2045 common, 1247 3-month-only, 340 18-month-only for the mammary gland) and compute condition-specific fractions against cell types (basal cells, T/B cells). Finally visualize with VisInteractions.py using --detectionThreshold 0.2, a circle layout, and --weightType mean to reproduce the delta network plot. Rationale: the 20% detection-rate threshold defines a gene as expressed in a cell type, and mean expression weights are the default NATMI edge-weighting for delta comparisons.

## Related tools

- **NATMI** (Python toolkit providing DiffEdges.py for the two-condition edge comparison and VisInteractions.py for rendering the red/blue delta network) — https://github.com/forrest-lab/NATMI/
- **connectomeDB2020** (Manually curated ligand-receptor pair database (2293 pairs, lrc2p build) against which both condition networks are built so edges are comparable) — https://github.com/forrest-lab/NATMI/
- **Python** (Runtime for ExtractEdges.py, DiffEdges.py and VisInteractions.py (tested on Python 2.7.17 and 3.7.6 with pandas, seaborn, igraph, NetworkX, PyGraphviz))

## Examples

```
python DiffEdges.py --refFolder 3m.mg --targetFolder 18m.mg --interDB lrc2p --out 3m-18m
```

## Evaluation signals

- DiffEdges.py edge counts must equal the reported values: 2045 edges at both ages, 1247 only at 3 months, 340 only at 18 months
- Derived fractions must match: 613/1247 (49.2%) of 3-month-only edges involve basal cells (>2x higher at 3 months) and 266/340 (78.2%) of 18-month-only edges involve T/B cells (>2x higher at 18 months)
- VisInteractions.py output (detectionThreshold 0.2, circle layout) must reproduce the red (reference-higher) / blue (target-higher) edge coloring of Figure 6
- Both condition networks must be built with the same interDB (lrc2p/connectomeDB2020) and mean weight type, otherwise edge sets are not comparable
- Genes expressed in a cell type must satisfy the >20% detection-rate rule before edge extraction

## Limitations

- Specificity weights are dataset-specific (local), not global; a global specificity reference based on the Human Cell Atlas is deferred to future NATMI versions
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts; highlighted edges derive from experiments without replicates
- Networks rely on original authors' cell annotations, which can affect accuracy; ligand-receptor pairs carry no binding-kinetics estimates

## Evidence

- [methods] Delta network analysis of Tabula Muris Senis mammary gland at 3 and 18 months: "To demonstrate the usage of NATMI in delta network analysis, we show the analysis on Tabula Muris Senis"
- [methods] DiffEdges identifies changed edges between two condition networks: "DiffEdges: Identification of changes in ligand-receptor edge weights between a cell-type pair in two conditions."
- [intro] NATMI's capability to compare two condition networks: "compare communication networks from two different conditions and identify edges (ligand–receptor pairs) that differ (delta network) between them"
- [discussion] Biological finding reproduced by this delta analysis: "Using the Tabula Muris Senis dataset, we showed that signalling involving mammary basal cells is down-regulated between 3 and 18 months"
- [readme] The README confirms the mammary gland ageing use case: "we identify age related changes in intercellular communication between the mammary gland of 3 and 18-month-old mice"
