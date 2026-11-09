---
name: scrna-seq-cell-type-aggregation
description: Use when scRNA-seq expression with cell-type labels must be collapsed
  to per-cell-type profiles to score ligand-receptor interactions between sender and
  receiver cell types at a single time point.
when_to_use_negative:
- You need dynamic interaction changes across a developmental series rather than a
  single-time-point network — use the DTW/fuzzy-clustering stages (developcorrelation)
  instead
- Ligand-receptor pairs of interest are absent from dsCellNet's default ~5,744-pair
  list and no user-supplied supplementary L-R list is provided
- Input is bulk or pseudobulk expression already aggregated per condition, so label
  permutation of single-cell type assignments is meaningless
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: dsCellNet
  role: R package providing datainput, preparepairs and singlenet for cell-type aggregation
    and per-time-point network inference
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps gene identifiers between human and mouse when building the membrane/secreted
    L-R database
- name: R
  role: Implementation environment (R >= 4.0.3 with igraph, Hmisc and other dependencies)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:e3a870d33c2f76171d4079b9504fd6388ad5a78c73a8f0040c1fc61643ee8ebc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - R
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# scrna-seq-cell-type-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregate single-cell expression per cell type (within a time point) so ligand-receptor edge weights can be computed as products of cell-type-level means/sums, and assess significance by permuting cell-type labels 100 times, retaining interactions with p < 0.05.

## When to use

Use when scRNA-seq expression with cell-type labels must be collapsed to per-cell-type profiles to score ligand-receptor interactions between sender and receiver cell types at a single time point.

## When NOT to use

- You need dynamic interaction changes across a developmental series rather than a single-time-point network — use the DTW/fuzzy-clustering stages (developcorrelation) instead
- Ligand-receptor pairs of interest are absent from dsCellNet's default ~5,744-pair list and no user-supplied supplementary L-R list is provided
- Input is bulk or pseudobulk expression already aggregated per condition, so label permutation of single-cell type assignments is meaningless

## Inputs

- scRNA-seq gene expression matrix (e.g. E15.5 mouse embryonic cerebral cortex, 11,670 single cells)
- Cell-type label table (Type column)
- Time-point annotation table (Time column)
- Ligand-receptor pair database classified by protein localization (membrane-ligand, secreted-ligand, membrane-receptor)
- homologene human/mouse homolog mapping

## Outputs

- Per-cell-type aggregated expression profiles
- Sender-receiver edge list with L-R pair, category, weight and p-value (lr_interactions_E15_5.csv)
- Set of significant cell-cell interactions (p < 0.05) for the time point

## How to apply

Load the expression matrix and cell-type label table with datainput(datloc, expnam, typnam, ...) from the dsCellNetv1 R package, then split L-R pairs into membrane-ligand, secreted-ligand and membrane-receptor classes with preparepairs(inpexp, species, dbloc, mainlab), mapping identifiers via the homologene human/mouse homolog database. Aggregate expression per cell type and compute edge weights as W = Mean(Ligand) x Mean(Receptor) for membrane-ligand pairs and W = Sum(Ligand) x Mean(Receptor) for secreted-ligand pairs — the localization-aware distinction is the rationale, since membrane and secreted ligands have different expression profiles. Run singlenet(inpexp, inptyp, "Type", inptim, "Time", memsercpros, usedcolo, "FALSE", dbloc, mainlab) restricted to the target time point (e.g. E15.5) to enumerate candidate sender-receiver interactions. Permute cell-type labels 100 times, recompute weights per permutation, assign each observed interaction a permutation-tail p-value, and keep only significant interactions (p < 0.05) when constructing the communication network.

## Related tools

- **dsCellNet** (R package providing datainput, preparepairs and singlenet for cell-type aggregation and per-time-point network inference) — github.com/songzh523/dsCellNet
- **homologene** (Maps gene identifiers between human and mouse when building the membrane/secreted L-R database)
- **R** (Implementation environment (R >= 4.0.3 with igraph, Hmisc and other dependencies))

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- lr_interactions_E15_5.csv exists and contains all six required columns: sender, receiver, L-R pair, category, weight, p-value
- Every retained row has p < 0.05 and all p-values lie in [0,1]
- All retained edge weights are positive finite numbers
- Permutation count equals 100 label shuffles and p-values correspond to permutation tail probabilities
- Membrane-ligand edges use Mean(Ligand) x Mean(Receptor) while secreted-ligand edges use Sum(Ligand) x Mean(Receptor), consistent with the localization categories

## Limitations

- Default L-R gene list size limits detection of some interactions (e.g. MG–Neurons); users can supply L-R genes as supplementary
- Published-method comparison notes SingleCellSignalR missed EC interactions and autocrine interactions — localization-aware significance aims to address such false positives, but the default DB still constrains coverage
- Significance depends on the 100-permutation resolution, so the smallest attainable non-zero p-value is bounded by permutation count

## Evidence

- [other] dsCellNet computes p values for interactions between two cell types by randomly shuffling cell type labels 100 times and retains only significant interactions (p < 0.05) for constructing the communication network.: [evidence span withheld — anchor status: not_found]
