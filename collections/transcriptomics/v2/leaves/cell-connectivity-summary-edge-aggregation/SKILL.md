---
name: cell-connectivity-summary-edge-aggregation
description: Use when NATMI ExtractEdges.py output edges exist and the goal is to
  summarise the entire communication network to show how strongly or specifically
  each cell type communicates to every other cell type, identifying highly communicating
  cell-type pairs or communities.
when_to_use_negative:
- The goal is per-ligand-receptor edge-level analysis or delta-network comparison
  between conditions (use VisInteractions.py edge views or DiffEdges.py instead of
  summary aggregation)
- Spatially-resolved communication inference is required — NATMI 'has not been specifically
  designed for spatial data'
- Heteromeric ligand or receptor complexes must be modelled — 'thus we do not model
  them in NATMI'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: NATMI
  role: Provides ExtractEdges.py to compute per-pair ligand-receptor expression/specificity
    edge weights and VisInteractions.py to visualise the summed-specificity cell-connectivity-summary
    network
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:782116ee99d2e2741c08c43921bc79057a02363634813e5e8d260f764db8bbda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-connectivity-summary-edge-aggregation

## Summary

Aggregate NATMI ligand-receptor-specificity edges between the same sending and receiving cell-type pairs into summary cell-connectivity edges whose weights are the sum of per-pair specificity weights, then rank the merged edges to reveal highly communicating cellular communities (e.g. in the Tabula Muris atlas).

## When to use

Use when NATMI ExtractEdges.py output edges exist and the goal is to summarise the entire communication network to show how strongly or specifically each cell type communicates to every other cell type, identifying highly communicating cell-type pairs or communities.

## When NOT to use

- The goal is per-ligand-receptor edge-level analysis or delta-network comparison between conditions (use VisInteractions.py edge views or DiffEdges.py instead of summary aggregation)
- Spatially-resolved communication inference is required — NATMI 'has not been specifically designed for spatial data'
- Heteromeric ligand or receptor complexes must be modelled — 'thus we do not model them in NATMI'

## Inputs

- ExtractEdges.py output folder with per-pair edge weight files (expression, specificity) computed with the lrc2p ligand-receptor database
- NATMI VisInteractions.py with --sourceFolder pointing to the edges folder and --interDB lrc2p
- Visualization parameters: --weightType mean, --detectionThreshold 0.2, --specificityThreshold 0, --expressionThreshold 0, --drawNetwork y, --layout circle, --edgeWidth 6, --plotWidth 10, --plotHeight 10, --fontSize 15, --maxClusterSize 0, --clusterDistance 0.6

## Outputs

- Ranked merged cell-type-pair summary edges with summed specificity weights identifying top communicating cell-type pairs/communities
- network_total-specificity-based_layout_circle.pdf written to the Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean folder
- Identified ligand-receptor pairs driving each community (e.g. Nppa-Npr3 cardiac-to-endocardial, Sst-Sstr1/Sstr5 delta cells, Thpo-Mpl hepatocyte-to-megakaryocyte-erythroid, Fga/Fgb/Fgg-to-Itgb1/Itga2b/Itgb2/Itgam fibrinogen-integrin)

## How to apply

Load the ExtractEdges.py output folder containing per-pair edge weights (expression and specificity) computed with the literature-supported lrc2p ligand-receptor list, then run VisInteractions.py with --weightType mean, --detectionThreshold 0.2, --specificityThreshold 0, --expressionThreshold 0, --drawNetwork y, and a circle layout. Aggregate all edges sharing the same sending and receiving cell types into summary edges whose weights are the sum of ligand-receptor specificity weights, then rank the merged cell-pair summary edges to identify the top communicating communities. Rationale: summing dataset-specific (local) specificity weights highlights cell-type pairs connected by many highly specific ligand-receptor paths, revealing communities such as cardiac Nppa-Npr3, delta Sst-Sstr1/Sstr5, hepatocyte-to-megakaryocyte-erythroid Thpo-Mpl, and fibrinogen-integrin signalling. Write the visualised network to the Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean folder as network_total-specificity-based_layout_circle.pdf.

## Related tools

- **NATMI** (Provides ExtractEdges.py to compute per-pair ligand-receptor expression/specificity edge weights and VisInteractions.py to visualise the summed-specificity cell-connectivity-summary network) — https://github.com/forrest-lab/NATMI/

## Examples

```
python VisInteractions.py --sourceFolder outfolder --interDB lrc2p --weightType mean --detectionThreshold 0.2 --specificityThreshold 0 --expressionThreshold 0 --drawNetwork y --layout circle
```

## Evaluation signals

- The output file network_total-specificity-based_layout_circle.pdf exists inside the Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean folder
- The top summary edges contain the reported ligand-receptor assignments: Nppa-Npr3, Sst-Sstr1/Sstr5, Thpo-Mpl, and fibrinogen (Fga/Fgb/Fgg) to integrins (Itgb1/Itga2b/Itgb2/Itgam)
- Each summary edge weight equals the sum of its constituent ligand-receptor specificity weights for that sending/receiving cell-type pair
- Summary edge weights were computed with --weightType mean specificity weights and detectionThreshold 0.2 filtering, matching Fig. 5 of the manuscript

## Limitations

- Specificity weights are dataset-specific (local), not global; 'the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific'
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts
- Highlighted edges are based on experiments without replicates
- Networks rely on the original authors' cell annotations, 'which can potentially affect the accuracy of predicted networks'

## Evidence

- [intro] summarise the entire communication network to show how strongly or specifically each cell type in a complex sample communicates to every other cell type, thus identifying highly communicating cell: "summarise the entire communication network to show how strongly or specifically each cell type in a complex sample communicates to every other cell type, thus identifying highly communicating cell"
- [methods] --detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2): "--detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [readme] By interrogating the Tabula Muris cell atlas we demonstrate the utility of NATMI to identify cellular communities and the key ligands and receptors involved.: "By interrogating the Tabula Muris cell atlas we demonstrate the utility of NATMI to identify cellular communities and the key ligands and receptors involved."
