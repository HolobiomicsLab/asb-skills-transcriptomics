---
name: membrane-versus-secreted-ligand-weight-computation
description: Use when constructing a single-time-point cell-cell communication network
  from scRNA-seq data and you need to score L-R candidate edges differently depending
  on whether the ligand is membrane-bound or secreted.
when_to_use_negative:
- You only have a single-cell dataset without cell-type labels or a localization-annotated
  L-R database — preparepairs and the permutation test cannot be run meaningfully.
- You need dynamic, cross-time-point interaction analysis (DTW correlation and fuzzy
  clustering via developcorrelation) rather than a single-time-point network.
- Your gene identifiers cannot be mapped through the homologene human/mouse homolog
  database and you have no species-supported L-R list.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: dsCellNet
  role: R package providing datainput, preparepairs, and singlenet functions that
    implement localization-aware weighting and the permutation significance test
  repo: https://github.com/songzh523/dsCellNet
- name: R
  role: Implementation language; dsCellNet is distributed as an open-access R package
    (dsCellNetv1_1.0.tar.gz, R >= 4.0.3)
- name: homologene
  role: Maps ligand-receptor gene identifiers to the default human/mouse homologs
    when classifying L-R pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:7e11be9991741ac68b5e557d9b64941e26fba79e0dc4f07ae553810c72b5bec2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  repo_url: https://github.com/songzh523/dsCellNet
  tools:
  - R
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# membrane-versus-secreted-ligand-weight-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

dsCellNet's localization-aware edge-weighting scheme that computes ligand-receptor interaction weights separately for membrane-ligand pairs (W = Mean(Ligand) x Mean(Receptor)) and secreted-ligand pairs (W = Sum(Ligand) x Mean(Receptor)), followed by a 100-permutation label-shuffling test retaining only significant interactions (p < 0.05).

## When to use

Use when constructing a single-time-point cell-cell communication network from scRNA-seq data and you need to score L-R candidate edges differently depending on whether the ligand is membrane-bound or secreted.

## When NOT to use

- You only have a single-cell dataset without cell-type labels or a localization-annotated L-R database — preparepairs and the permutation test cannot be run meaningfully.
- You need dynamic, cross-time-point interaction analysis (DTW correlation and fuzzy clustering via developcorrelation) rather than a single-time-point network.
- Your gene identifiers cannot be mapped through the homologene human/mouse homolog database and you have no species-supported L-R list.

## Inputs

- scRNA-seq gene expression matrix (e.g. E15.5 mouse embryonic cerebral cortex, 11,670 single cells)
- Cell-type label table (Type column)
- Time-point label table (Time column, optional)
- Ligand-receptor pair database classified by protein localization (from preparepairs, using homologene human/mouse homologs)

## Outputs

- Weighted, significance-filtered edge list (lr_interactions_E15_5.csv) with columns: sender, receiver, L-R pair, category, weight, p-value
- memsercpros object classifying L-R pairs into membrane-ligand, secreted-ligand, and membrane-receptor classes

## How to apply

Load the scRNA-seq expression matrix and cell-type labels with datainput(datloc, expnam, typnam, ...) from the dsCellNetv1 R package, then run preparepairs(inpexp, species, dbloc, mainlab) to classify ligand-receptor pairs into membrane-ligand, secreted-ligand and membrane-receptor classes, mapping identifiers via the homologene human/mouse homolog database. Aggregate expression per cell type and compute edge weights as W = Mean(Ligand) x Mean(Receptor) for membrane-ligand pairs versus W = Sum(Ligand) x Mean(Receptor) for secreted-ligand pairs; the rationale is that secreted ligands act at a distance, so their summed expression better reflects sender potential. Call singlenet(inpexp, inptyp, "Type", inptim, "Time", memsercpros, usedcolo, "FALSE", dbloc, mainlab) restricted to the target time point (e.g. E15.5) to enumerate candidate sender-receiver interactions. Permute cell-type labels 100 times, recompute the weights per permutation, and assign each observed interaction a p-value equal to the permutation tail probability; keep only interactions with p < 0.05.

## Related tools

- **dsCellNet** (R package providing datainput, preparepairs, and singlenet functions that implement localization-aware weighting and the permutation significance test) — https://github.com/songzh523/dsCellNet
- **R** (Implementation language; dsCellNet is distributed as an open-access R package (dsCellNetv1_1.0.tar.gz, R >= 4.0.3))
- **homologene** (Maps ligand-receptor gene identifiers to the default human/mouse homologs when classifying L-R pairs)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- lr_interactions_E15_5.csv exists and contains all six required columns: sender, receiver, L-R pair, category, weight, p-value
- Every retained row has p < 0.05 and all p-values lie in [0,1]
- All retained edge weights are positive finite numbers
- Category column values are consistent with the preparepairs classification (membrane-ligand, secreted-ligand, membrane-receptor)
- The permutation procedure ran the stated 100 label shuffles before p-value assignment

## Limitations

- The default L-R gene list size can cause some interactions to be missed (e.g. MG–Neurons interactions in individual samples); users can supply supplementary L-R genes.
- The default list contains 5,744 interactions (5,684 listed in the tools comparison table), so interactions absent from Ramilowski et al. 2015, CellPhoneDB, NATMI and SingleCellSignalR compilations will not be detected.
- The article does not state a dsCellNet version number for reproducibility.

## Evidence
