---
name: aging-comparative-network-visualization
description: Use when comparing predicted ligand-receptor communication networks across
  two conditions or ages (e.g. young vs old tissue) from single-cell expression data
  and you need differential edge counts and a delta network plot.
when_to_use_negative:
- Data is spatial transcriptomics — NATMI 'has not been specifically designed for
  spatial data'
- The goal is predicting downstream signalling consequences or modeling heteromeric
  complexes — NATMI 'do[es] not model them in NATMI' and downstream prediction requires
  tools like NicheNet or SoptSC
- Only one condition/age is available — delta network analysis with DiffEdges.py requires
  networks from two conditions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: NATMI
  role: Toolkit whose ExtractEdges.py, DiffEdges.py and VisInteractions.py scripts
    build per-age communication networks, compute the delta network, and render the
    age-comparison figure
  repo: https://github.com/forrest-lab/NATMI/
- name: connectomeDB2020
  role: Curated ligand-receptor pair database (lrc2p) used as the interaction reference
    for edge extraction and differencing
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:7019a9094e822ed4fcb75819b573fa3e0060a46f92ba6bed066d5222954bfa5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  - Python
  - connectomeDB2020
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# aging-comparative-network-visualization

## Summary

Reproduces NATMI differential (delta) network analysis of cell-to-cell communication between 3- and 18-month-old murine mammary gland (Tabula Muris Senis), using ExtractEdges.py per age, DiffEdges.py for edge-weight changes, and VisInteractions.py to render the age-comparison network with red/blue edge coloring.

## When to use

Use when comparing predicted ligand-receptor communication networks across two conditions or ages (e.g. young vs old tissue) from single-cell expression data and you need differential edge counts and a delta network plot.

## When NOT to use

- Data is spatial transcriptomics — NATMI 'has not been specifically designed for spatial data'
- The goal is predicting downstream signalling consequences or modeling heteromeric complexes — NATMI 'do[es] not model them in NATMI' and downstream prediction requires tools like NicheNet or SoptSC
- Only one condition/age is available — delta network analysis with DiffEdges.py requires networks from two conditions

## Inputs

- CPM-normalized (divided by total UMIs, multiplied by 1,000,000) 3-month mammary gland expression matrix (3m.upm.em.csv)
- CPM-normalized 18-month mammary gland expression matrix (18m.upm.em.csv)
- Cell-type annotation files (3m.ann.csv, 18m.ann.csv)
- connectomeDB2020 (lrc2p) ligand-receptor pair database

## Outputs

- delta_edges_3m-18m.csv summary table with 2045 common, 1247 3-month-only, and 340 18-month-only edges
- network_total-specificity-based_layout_circle.pdf delta network plot with red (reference-higher) and blue (target-higher) edges
- DiffEdges.py output folder 3m-18m with edge-weight changes

## How to apply

Download the pre-normalized CPM expression matrices and cell annotations for 3- and 18-month mammary gland from the figshare deposit, then run ExtractEdges.py separately per age with --species mouse and the connectomeDB2020 (lrc2p) ligand-receptor list. Run DiffEdges.py with --refFolder 3m.mg --targetFolder 18m.mg --out 3m-18m using the mean expression weight type to obtain edge weight changes between ages. Visualize with VisInteractions.py (--detectionThreshold 0.2, --weightType mean, --layout circle) to reproduce the Figure 6 delta network, where red edges are reference-higher (3-month) and blue edges target-higher (18-month). Finally tabulate the differential edge counts (2045 shared, 1247 3-month-only, 340 18-month-only) and verify the derived fractions: 49.2% of 3-month-only edges involve basal cells and 78.2% of 18-month-only edges involve T/B cells.

## Related tools

- **NATMI** (Toolkit whose ExtractEdges.py, DiffEdges.py and VisInteractions.py scripts build per-age communication networks, compute the delta network, and render the age-comparison figure) — https://github.com/forrest-lab/NATMI/
- **connectomeDB2020** (Curated ligand-receptor pair database (lrc2p) used as the interaction reference for edge extraction and differencing) — https://github.com/forrest-lab/NATMI/

## Examples

```
python DiffEdges.py --refFolder 3m.mg --targetFolder 18m.mg --interDB lrc2p --out 3m-18m; python VisInteractions.py --sourceFolder 3m-18m --interDB lrc2p --weightType mean --detectionThreshold 0.2 --drawNetwork y --layout circle
```

## Evaluation signals

- DiffEdges output must show exactly 2045 edges detected at both ages, 1247 only at 3 months, and 340 only at 18 months
- 613/1247 (49.2%) of 3-month-only edges must involve basal cells with >2x higher weight at 3 months
- 266/340 (78.2%) of 18-month-only edges must involve T and B cells with >2x higher weight at 18 months
- Rendered delta network must reproduce Fig. 6 red (reference-higher) / blue (target-higher) edge coloring under --weightType mean and --detectionThreshold 0.2

## Limitations

- Specificity weights are dataset-specific (local, computed from the input dataset), not global; a global reference based on the Human Cell Atlas is deferred to future versions
- Predictions rely on original authors' cell annotations, which can affect network accuracy
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts; highlighted edges derive from experiments without replicates
- Heteromeric complexes, heterodimerization, co-receptor requirements, and binding kinetics are not modeled

## Evidence

- [methods] Delta network analysis of Tabula Muris Senis mammary gland at 3 and 18 months: "To demonstrate the usage of NATMI in delta network analysis, we show the analysis on Tabula Muris Senis"
- [methods] DiffEdges purpose: "DiffEdges: Identification of changes in ligand-receptor edge weights between a cell-type pair in two conditions."
- [methods] detectionThreshold default: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] Aging finding: "Using the Tabula Muris Senis dataset, we showed that signalling involving mammary basal cells is down-regulated between 3 and 18 months"
- [intro] Delta network capability: "compare communication networks from two different conditions and identify edges (ligand–receptor pairs) that differ (delta network) between them"
