---
name: specificity-weighted-cell-pair-ranking
description: Use when a NATMI ExtractEdges.py output folder of per-pair specificity/expression
  edge weights is available and the goal is to identify the top communicating cell-type
  pairs or communities (e.g. Fig. 5 Tabula Muris cardiac Nppa-Npr3).
when_to_use_negative:
- When comparing two conditions and looking for differing edges — use NATMI's DiffEdges.py
  delta network analysis instead
- When spatial context or downstream signalling consequences are required — NATMI
  does not support spatial transcriptomics or downstream signalling prediction
- When heteromeric ligand-receptor complexes are essential to the question — NATMI
  does not model them
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: ExtractEdges.py computes per-pair specificity/expression edge weights and
    VisInteractions.py visualises and summarises the cell-to-cell communication network
    used for the cell-pair ranking
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:bd3c3949cf1bd2c7e099580649d8ccee31c5c4777b1c6885b3b7319536e446aa
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# specificity-weighted-cell-pair-ranking

## Summary

Aggregates NATMI ligand-receptor specificity edges between the same sending/receiving cell-type pair into summary edges whose weights are the sum of per-pair specificity weights, then ranks these cell-pair summary edges to identify the most highly communicating cellular communities and the ligand-receptor pairs driving them.

## When to use

Use when a NATMI ExtractEdges.py output folder of per-pair specificity/expression edge weights is available and the goal is to identify the top communicating cell-type pairs or communities (e.g. Fig. 5 Tabula Muris cardiac Nppa-Npr3).

## When NOT to use

- When comparing two conditions and looking for differing edges — use NATMI's DiffEdges.py delta network analysis instead
- When spatial context or downstream signalling consequences are required — NATMI does not support spatial transcriptomics or downstream signalling prediction
- When heteromeric ligand-receptor complexes are essential to the question — NATMI does not model them

## Inputs

- ExtractEdges.py output folder containing per-pair edge weights (expression, specificity) computed with the lrc2p ligand-receptor list
- Tabula Muris single-cell expression matrix and cell-type annotation file
- lrc2p ligand-receptor interaction database (--interDB lrc2p)

## Outputs

- Ranked list of merged (summed-specificity) cell-type pair summary edges
- Network PDF (network_total-specificity-based_layout_circle.pdf) written to the Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean folder
- Identification of dominant ligand-receptor pairs per top community (Nppa-Npr3, Sst-Sstr1/Sstr5, Thpo-Mpl, fibrinogen-integrin)

## How to apply

Load the ExtractEdges.py output folder computed with the lrc2p literature-supported ligand-receptor list, then run VisInteractions.py with --weightType mean, --detectionThreshold 0.2, --specificityThreshold 0, --expressionThreshold 0 and --drawNetwork y to visualise the network. Aggregate all edges sharing the same sending and receiving cell types into summary edges whose weights equal the sum of the ligand-receptor specificity weights. Rank the merged cell-pair summary edges and identify the top communicating cell-type pairs/communities, checking that the drivers match the reported pairs (Nppa-Npr3 from cardiac muscle to endocardial cells, Sst-Sstr1/Sstr5 from delta cells, Thpo-Mpl from hepatocytes to megakaryocyte-erythroid cells, fibrinogen Fga/Fgb/Fgg to Itgb1/Itga2b/Itgb2/Itgam). Write the visualised network to the Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean folder as network_total-specificity-based_layout_circle.pdf.

## Related tools

- **NATMI** (ExtractEdges.py computes per-pair specificity/expression edge weights and VisInteractions.py visualises and summarises the cell-to-cell communication network used for the cell-pair ranking) — https://github.com/forrest-lab/NATMI/

## Examples

```
python VisInteractions.py --sourceFolder edges_folder --interDB lrc2p --weightType mean --detectionThreshold 0.2 --specificityThreshold 0 --expressionThreshold 0 --drawNetwork y --layout circle
```

## Evaluation signals

- The generated network file exists at Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean/network_total-specificity-based_layout_circle.pdf
- Top summary edges contain the reported ligand-receptor assignments (Nppa-Npr3, Sst-Sstr1/Sstr5, Thpo-Mpl, fibrinogen-integrin)
- Summary edge weights equal the sum of ligand-receptor specificity weights for all edges sharing the same sending and receiving cell types
- Detection threshold 0.2 was applied (genes considered expressed if detected in more than 20% of cells of a cell type)

## Limitations

- Specificity weights are dataset-specific (local, calculated from the input dataset), not global; global specificities based on the Human Cell Atlas are deferred to future NATMI versions
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts
- Networks rely on the original authors' cell annotations, which can affect accuracy; NATMI does not model heteromeric complexes, heterodimerization, co-receptor requirements, or binding kinetics

## Evidence

- [intro] summarise the entire communication network: "summarise the entire communication network to show how strongly or specifically each cell type in a complex sample communicates to every other cell type, thus identifying highly communicating cell"
- [discussion] local specificity caveat: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [methods] detection threshold: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] expressed gene definition: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [readme] database grounding: "NATMI uses connectomeDB2020 (the most up-to-date manually curated ligand-receptor interaction list) and user supplied gene expression tables with cell type labels"
