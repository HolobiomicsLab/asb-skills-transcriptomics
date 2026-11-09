---
name: autocrine-signalling-specificity-ranking
description: Use when a single-cell expression matrix with cell-type annotations should
  be queried for within-cell-type co-expression of ligand–receptor pairs (self-signalling
  potential) and autocrine edges need to be ranked against intra-/inter-organ edges
  by specificity.
when_to_use_negative:
- You need downstream signalling consequences or pathway inference (NATMI does not
  predict downstream signalling; use tools like NicheNet/SoptSC)
- Spatial transcriptomics data where ligand–receiver spatial relationships matter
  (NATMI has not been specifically designed for spatial data)
- Ligand–receptor pairs require heteromeric complex or co-receptor modelling (NATMI
  does not model heterodimerization or co-receptor requirements)
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: NATMI
  role: Constructs the cell-to-cell communication network from the expression matrix
    and cell-type annotations via ExtractEdges.py, applies detection/expression thresholds,
    and supplies edge weights for autocrine specificity ranking
  repo: https://github.com/forrest-lab/NATMI/
- name: connectomeDB2020
  role: Default curated ligand–receptor pair database (2,293 pairs with literature
    support) used as --interDB lrc2p
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:c824ad8b09179c40cd3a9518ad70f9ad406204915cc9b2f1520125de79dd8b20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  - connectomeDB2020
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# autocrine-signalling-specificity-ranking

## Summary

Rank self-signalling (autocrine) ligand–receptor edges from a single-cell atlas by summed specificity, testing whether cognate ligands and receptors are co-expressed within individual cell types. Applied with NATMI on Tabula Muris, it showed autocrine signalling is a major feature of cell-to-cell communication networks (719/2293 connectomeDB2020 pairs co-detected; median 54 co-detected pairs per cell type).

## When to use

Use when a single-cell expression matrix with cell-type annotations should be queried for within-cell-type co-expression of ligand–receptor pairs (self-signalling potential) and autocrine edges need to be ranked against intra-/inter-organ edges by specificity.

## When NOT to use

- You need downstream signalling consequences or pathway inference (NATMI does not predict downstream signalling; use tools like NicheNet/SoptSC)
- Spatial transcriptomics data where ligand–receiver spatial relationships matter (NATMI has not been specifically designed for spatial data)
- Ligand–receptor pairs require heteromeric complex or co-receptor modelling (NATMI does not model heterodimerization or co-receptor requirements)

## Inputs

- Single-cell gene expression matrix (gene symbols, e.g. mouse official gene symbols; UMI counts)
- Cell-type annotation metafile mapping each cell to one of the 117 Tabula Muris cell types
- connectomeDB2020 (lrc2p) ligand–receptor interaction list (2,293 literature-supported pairs, bundled with NATMI/Docker image)

## Outputs

- Per-cell-type autocrine (self-signalling) edge table with co-detected ligand–receptor pairs
- Summed-specificity weights and specificity ranks of autocrine edges versus intra-organ and inter-organ edges
- Aggregate statistics: median co-detected pairs per cell type and total co-detected pair count (expected ~54 and 719/2293 ≈ 31%)

## How to apply

Load the single-cell expression matrix and cell-type annotation metafile, then normalise each expression profile to counts per million by dividing by total UMI counts and multiplying by 1,000,000. Run NATMI's ExtractEdges.py with the organism-appropriate --species (e.g. mouse via NCBI HomoloGene homology matching) and the --interDB lrc2p connectomeDB2020 list of 2,293 literature-supported ligand–receptor pairs. Retain edges using NATMI's defaults: detectionThreshold 0.2 (a gene counts as expressed if detected in more than 20% of cells of a cell type) plus an expression threshold of 10 CPM. Aggregate autocrine (self-signalling) edges per cell type and compute the median number of co-detected pairs per cell type and the fraction of connectomeDB2020 pairs co-detected in at least one cell type. Compute summed-specificity weights for each edge — noting these specificities are local, calculated from the input dataset only — and rank autocrine edges against intra-organ and inter-organ edges.

## Related tools

- **NATMI** (Constructs the cell-to-cell communication network from the expression matrix and cell-type annotations via ExtractEdges.py, applies detection/expression thresholds, and supplies edge weights for autocrine specificity ranking) — https://github.com/forrest-lab/NATMI/
- **connectomeDB2020** (Default curated ligand–receptor pair database (2,293 pairs with literature support) used as --interDB lrc2p)

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --species mouse --interDB lrc2p --detectionThreshold 0.2 --expressionThreshold 10 --coreNum 4 --out autocrine_edges
```

## Evaluation signals

- Median number of co-detected ligand–receptor pairs per cell type is ~54
- Total co-detected pair count is 719 of 2,293 connectomeDB2020 pairs (~31%) at the 10 CPM threshold
- Autocrine edges show higher summed-specificity ranks than intra-organ and inter-organ edges
- Only pairs whose ligand and receptor pass the 0.2 detection threshold (non-zero counts in >20% of a cell type's cells) and 10 CPM expression threshold are retained
- Edge weights are summed specificities computed from the input dataset (local, dataset-specific)

## Limitations

- Specificity weights are dataset-specific (local), not global; global specificities based on resources like the Human Cell Atlas are deferred to future NATMI versions
- Predictions rely on the original authors' cell annotations and on mRNA expression rather than protein concentration, subject to dropouts
- Heteromeric complexes, heterodimerization and co-receptor requirements are not modelled; no binding-kinetics estimates are provided
- Edges highlighted without experimental replicates carry extra uncertainty

## Evidence

- [abstract] autocrine signalling is a major feature of cell-to-cell communication networks: "autocrine signalling is a major feature of cell-to-cell communication networks"
- [abstract] self-signalling potential from co-expressed ligand-receptor pairs: "hundreds of ligands and their cognate receptors are co-expressed in individual cells suggesting a substantial potential for self-signalling"
- [methods] detection threshold 0.2 definition: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] specificity threshold filter: "--specificityThreshold SPECIFICITYTHRESHOLD
                        do not draw the edges whose specificities are not greater than the threshold (default 0)"
- [discussion] local specificity weights caveat: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [readme] NATMI/Tabula Muris demonstration in README: "we confirm our previous predictions from bulk data that autocrine signalling is a major feature of cell-to-cell communication networks and for the first time ever show a substantial potential for"
