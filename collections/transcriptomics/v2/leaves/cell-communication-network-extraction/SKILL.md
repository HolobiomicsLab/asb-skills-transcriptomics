---
name: cell-communication-network-extraction
description: Use when you have a CPM-normalized single-cell expression matrix plus
  a cell-annotation metafile and want to predict which ligand–receptor pairs mediate
  communication between annotated cell types.
when_to_use_negative:
- You need downstream signalling consequence prediction (NicheNet, SoptSC) or spatially
  resolved interaction inference — NATMI is not designed for these
- Your ligand–receptor pairs require heteromeric complex, heterodimerization, or co-receptor
  modelling — NATMI does not model them
- You need binding kinetics or protein-level interaction evidence — predictions rely
  on mRNA expression and are subject to dropouts
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Python toolkit whose ExtractEdges.py, DiffEdges.py, and VisInteractions.py
    scripts extract, differentially compare, and visualize cell-to-cell communication
    networks using connectomeDB2020
  repo: https://github.com/forrest-lab/NATMI/
- name: connectomeDB2020
  role: Default manually curated database of 2,293 literature-supported human ligand–receptor
    pairs used via --interDB lrc2p
- name: LRdb
  role: Alternative ligand/receptor database from SingleCellSignalR that can be supplied
    as a user-defined pair list
  repo: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:a86ae54e1c088634c022112288c126de7f55126da91dc10183971bccf6176dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - LRdb
  - NATMI
  - Python
  - connectomeDB2020
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-communication-network-extraction

## Summary

Extract and analyze cell-to-cell communication networks from single-cell (or bulk) expression data by predicting ligand–receptor interactions between cell types using NATMI with the connectomeDB2020 (lrc2p) library of 2,293 literature-supported ligand–receptor pairs.

## When to use

Use when you have a CPM-normalized single-cell expression matrix plus a cell-annotation metafile and want to predict which ligand–receptor pairs mediate communication between annotated cell types.

## When NOT to use

- You need downstream signalling consequence prediction (NicheNet, SoptSC) or spatially resolved interaction inference — NATMI is not designed for these
- Your ligand–receptor pairs require heteromeric complex, heterodimerization, or co-receptor modelling — NATMI does not model them
- You need binding kinetics or protein-level interaction evidence — predictions rely on mRNA expression and are subject to dropouts

## Inputs

- CPM-normalized gene expression matrix (expression.matrix.txt / emFile CSV; official gene symbols for human/mouse, HGNC ID also supported)
- Cell labels metafile mapping each cell barcode to a cell type (annotation.txt / annFile, single-cell analysis only)
- Ligand–receptor pair list: precompiled connectomeDB2020 (lrc2p) or a user-supplied binary pair list

## Outputs

- Edge list of predicted ligand–receptor-mediated interactions between cell types with mean-expression and specificity weight columns (e.g., 126,738 edges for the Skelly cardiac dataset matching Supplementary Data 2)
- Delta network table of edge-weight changes between two conditions (e.g., 2045 shared, 1247 3-month-only, 340 18-month-only mammary edges)
- Rendered communication/delta network plots (e.g., network_total-specificity-based_layout_circle.pdf)

## How to apply

First normalize the expression matrix to counts per million (divide each profile by total UMIs/reads and multiply by 1,000,000) and confirm cell annotations map each barcode to a cell type (e.g., the 12 Skelly cardiac cell types). Run NATMI's ExtractEdges.py with --interDB lrc2p (connectomeDB2020) and --weightType mean, applying the detection threshold of 0.2 so a gene counts as expressed only if detected in more than 20% of cells of a cell type. Each output edge carries both a mean-expression weight and a specificity weight, which can yield substantially different top-ranked pairs, so rank edges under each metric when prioritizing interactions. To compare two conditions (e.g., 3- vs 18-month Tabula Muris Senis mammary gland), run ExtractEdges.py per condition and then DiffEdges.py with --refFolder/--targetFolder to derive the delta network. Visualize results with VisInteractions.py, optionally filtering edges via --detectionThreshold, --expressionThreshold, --specificityThreshold, and --keepTopEdge.

## Related tools

- **NATMI** (Python toolkit whose ExtractEdges.py, DiffEdges.py, and VisInteractions.py scripts extract, differentially compare, and visualize cell-to-cell communication networks using connectomeDB2020) — https://github.com/forrest-lab/NATMI/
- **connectomeDB2020** (Default manually curated database of 2,293 literature-supported human ligand–receptor pairs used via --interDB lrc2p)
- **LRdb** (Alternative ligand/receptor database from SingleCellSignalR that can be supplied as a user-defined pair list) — https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt

## Examples

```
python ExtractEdges.py --species mouse --emFile 3m.upm.em.csv --annFile 3m.ann.csv --interDB lrc2p --coreNum 4 --out 3m.mg
```

## Evaluation signals

- Reproduced edge count matches the reference: 126,738 predicted edges for the Skelly cardiac dataset (Supplementary Data 2)
- Detection-threshold invariant: every retained edge has ligand and receptor detection rates >0.2 (gene detected in >20% of cells of a cell type)
- Delta network counts match reported values: 2045 edges common, 1247 only at 3 months, 340 only at 18 months in mammary gland
- Derived fractions match: 613/1247 (49.2%) of 3-month-only edges involve basal cells and 266/340 (78.2%) of 18-month-only edges involve T/B cells
- Each output edge row carries both mean-expression and specificity weight columns; top-20 rankings under the two weightings can be compared for divergence

## Limitations

- Specificity weights are dataset-specific (local), not global; global specificities based on the Human Cell Atlas are deferred to future NATMI versions
- Predictions rely on mRNA expression rather than protein concentration and are subject to dropouts; no binding-kinetics estimates
- Heteromeric complexes, heterodimerization, and co-receptor requirements are not modelled; spatial transcriptomics data is not specifically supported
- Networks depend on the original authors' cell annotations, which can affect accuracy

## Evidence

- [abstract] NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support): "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] detection threshold 20% of cells: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [discussion] ExtractEdges.py command: [evidence span withheld — anchor status: not_found]
- [methods] detectionThreshold default: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] CPM normalization: "normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000"
- [discussion] local specificity caveat: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [discussion] no heteromeric modelling: "thus we do not model them in NATMI"
