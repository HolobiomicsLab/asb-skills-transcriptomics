---
name: membrane-secreted-ligand-classification
description: Use when inferring ligand-receptor cell-cell interactions from scRNA-seq
  and you need to distinguish membrane-ligand vs secreted-ligand mediated receptor
  interactions, e.g. because product-of-sum aggregation misses rare cell types like
  endothelial cells or OPCs.
when_to_use_negative:
- When the L-R database lacks protein localization annotations for the ligands of
  interest, since the localization-aware significance cannot be computed
- When a single stationary-condition interaction map suffices and no distinction between
  membrane- and secreted-ligand mediation is needed
- When interactions involve ligands absent from the default L-R gene list (e.g. some
  MG–Neurons interactions), which may require user-supplied supplementary L-R genes
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: dsCellNet
  role: R package providing preparepairs, singlenet, and comparednet for localization-aware
    L-R pair classification and interaction inference
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps human L-R pairs from the compiled databases to mouse homologs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:5bd879634713a2d6b7e54d18741baca184e15b68d3b9435fe08a9aa0b52c3af9
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# membrane-secreted-ligand-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A strategy from dsCellNet that classifies ligand-receptor candidate pairs by protein localization (membrane-bound vs secreted ligand) and computes interaction significance separately for each class, improving sensitivity over product-of-mean or product-of-sum expression aggregation.

## When to use

Use when inferring ligand-receptor cell-cell interactions from scRNA-seq and you need to distinguish membrane-ligand vs secreted-ligand mediated receptor interactions, e.g. because product-of-sum aggregation misses rare cell types like endothelial cells or OPCs.

## When NOT to use

- When the L-R database lacks protein localization annotations for the ligands of interest, since the localization-aware significance cannot be computed
- When a single stationary-condition interaction map suffices and no distinction between membrane- and secreted-ligand mediation is needed
- When interactions involve ligands absent from the default L-R gene list (e.g. some MG–Neurons interactions), which may require user-supplied supplementary L-R genes

## Inputs

- Gene expression matrix from scRNA-seq (e.g. 11,670 E15.5 mouse embryonic cerebral cortex single cells from GSE153164)
- Cell-type label file
- Time-point annotation file (optional)
- dsCellNet default L-R database (dbloc; 5,744 pairs with protein localization annotations)

## Outputs

- Per cell-type-pair inferred interactions split by L-R category (membrane-ligand vs secreted-ligand mediated)
- Ligand-receptor candidate pair list partitioned into membrane and secreted classes (preparepairs output)
- Comparison tables of interaction counts across aggregation variants (e.g. variant_interaction_counts.csv)

## How to apply

Build candidate L-R pairs with dsCellNet's preparepairs against the default database of 5,744 interactions compiled from Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396), NATMI (4,071), and SingleCellSignalR (3,251), homologene-mapped to mouse. Classify each ligand as membrane or secreted using public protein localization information, then run singlenet to compute the significance of membrane vs secreted L-R genes separately rather than applying a single product-of-mean or product-of-sum aggregation. This localization-aware calculation recovers more secreted-ligand/membrane-receptor interactions than product-of-mean and avoids the product-of-sum failure mode that misses rare cell types. Optionally reinforce significance via dynamic time warping within and between adjacent time points before comparing variants with comparednet.

## Related tools

- **dsCellNet** (R package providing preparepairs, singlenet, and comparednet for localization-aware L-R pair classification and interaction inference) — github.com/songzh523/dsCellNet
- **homologene** (Maps human L-R pairs from the compiled databases to mouse homologs)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- The product-of-sum variant yields the fewest inferred interactions and misses endothelial cell (EC) and OPC interactions (Fig. 3A/3C reference pattern)
- The localization-aware regularized dsCellNet run captures more secreted-ligand/membrane-receptor interactions than the product-of-mean variant
- Every inferred interaction in the output is labeled as membrane-ligand or secreted-ligand mediated with no unclassified entries
- Default L-R candidate list resolves to 5,744 interactions after homologene mouse mapping

## Limitations

- The default L-R gene list size limits detection of some interactions (e.g. MG–Neurons); users can supply supplementary L-R genes
- Published methods cannot distinguish membrane-ligand vs secreted-ligand mediated receptor interactions; this classification depends on public protein localization data being available and correct
- Significance calculations are per time point; dynamic reinforcement requires developmental series time-point annotations

## Evidence
