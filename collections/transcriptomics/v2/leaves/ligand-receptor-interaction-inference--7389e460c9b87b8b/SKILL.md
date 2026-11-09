---
name: ligand-receptor-interaction-inference
description: Use when a developmental-series scRNA-seq dataset (expression matrix
  + cell-type labels + time-point annotations) needs cell-cell interaction inference
  that distinguishes membrane- from secreted-ligand signaling and tracks dynamic interaction
  changes across time points.
when_to_use_negative:
- The dataset is a single stationary time point with no developmental series information
  — the DTW reinforcement and fuzzy clustering across time points, dsCellNet's key
  advantage, cannot be exploited
- The signaling of interest is absent from the default 5,744-pair L-R list (e.g. reported
  missing MG–Neurons interactions) and no supplementary L-R gene list can be provided
- A per-method runtime or accuracy benchmark against NATMI, CellPhoneDB or SingleCellSignalR
  is the goal rather than interaction inference itself
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: dsCellNet
  role: Primary R package performing localization-aware L-R inference with DTW and
    fuzzy clustering
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps human L-R pair lists to mouse orthologs when preparing the candidate
    pair list
- name: Mfuzz
  role: Fuzzy clustering of L-R pair expression changes into eight clusters to refine
    key time points and connections
- name: NATMI
  role: Published baseline method whose 4,071 pairs are incorporated into the default
    database and used for accuracy/runtime comparison
- name: CellPhoneDB
  role: Published baseline method whose 1,396 pairs are incorporated into the default
    database and used for comparison
- name: SingleCellSignalR
  role: Published baseline method whose 3,251 pairs are incorporated into the default
    database and used for comparison
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:b221c671512b1e906874167948bc57464bff7e9689d14f688907b20900e3fbd3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - Mfuzz
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  - homologene
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# ligand-receptor-interaction-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer cell–cell communication networks from developmental-series scRNA-seq with dsCellNet, which scores membrane- and secreted-ligand interactions separately against a 5,744-pair L-R database, then reinforces interactions with dynamic time warping and fuzzy clustering to recover key time points and mediator cell types.

## When to use

Use when a developmental-series scRNA-seq dataset (expression matrix + cell-type labels + time-point annotations) needs cell-cell interaction inference that distinguishes membrane- from secreted-ligand signaling and tracks dynamic interaction changes across time points.

## When NOT to use

- The dataset is a single stationary time point with no developmental series information — the DTW reinforcement and fuzzy clustering across time points, dsCellNet's key advantage, cannot be exploited
- The signaling of interest is absent from the default 5,744-pair L-R list (e.g. reported missing MG–Neurons interactions) and no supplementary L-R gene list can be provided
- A per-method runtime or accuracy benchmark against NATMI, CellPhoneDB or SingleCellSignalR is the goal rather than interaction inference itself

## Inputs

- Gene expression matrix (dsCellNet datainput expnam file, e.g. GSE153164 E15.5 mouse embryonic cerebral cortex, 11,670 cells)
- Cell-type label file (typnam)
- Time-point annotation file (timnam, optional)
- L-R database location (dbloc; default dsCellNet 5,744-pair list or user-supplied supplementary pairs)
- Species name for homologene ortholog mapping
- Control time-point name (controltime) and comparison target (treattime) for comparednet

## Outputs

- Per-time-point cell-type-pair interaction networks split by ligand category (membrane-ligand vs secreted-ligand mediated)
- DTW-correlated interaction clusters with identified key time points and mediator cell types
- Control-vs-treatment interaction comparison (comparednet output, e.g. variant_interaction_counts.csv)
- Incoming/outgoing/total importance scores per cell type per time point (importance_plot)

## How to apply

In R (>=4.0.3, install dsCellNetv1_1.0.tar.gz from source with igraph, homologene, Hmisc, ggpubr, pheatmap, reshape2, ggalluvial, tseries and magrittr dependencies), load expression, cell-type and time-point files via datainput(datloc, expnam, typnam, timnam, "FALSE", resloc). Build membrane/secreted ligand-receptor candidates with preparepairs(inpexp, species, dbloc, mainlab) against the default database of 5,744 interactions compiled from Ramilowski et al. (2,557), CellPhoneDB (1,396), NATMI (4,071) and SingleCellSignalR (3,251) pairs, homologene-mapped to mouse; users can supply supplementary L-R genes if the default list is limiting. Run singlenet(inpexp, inptyp, "Type", inptim, "Time", memsercpros, usedcolo, "FALSE", dbloc, mainlab) per time point, computing significance separately for membrane vs secreted ligands; then developcorrelation() applies dynamic time warping within and between adjacent time points and fuzzy clustering (eight clusters via Mfuzz) refines key time points, with comparednet(controltime, treattime, ...) for treatment-vs-control comparison.

## Related tools

- **dsCellNet** (Primary R package performing localization-aware L-R inference with DTW and fuzzy clustering) — github.com/songzh523/dsCellNet
- **homologene** (Maps human L-R pair lists to mouse orthologs when preparing the candidate pair list)
- **Mfuzz** (Fuzzy clustering of L-R pair expression changes into eight clusters to refine key time points and connections)
- **NATMI** (Published baseline method whose 4,071 pairs are incorporated into the default database and used for accuracy/runtime comparison)
- **CellPhoneDB** (Published baseline method whose 1,396 pairs are incorporated into the default database and used for comparison)
- **SingleCellSignalR** (Published baseline method whose 3,251 pairs are incorporated into the default database and used for comparison)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- The default L-R pair list retains 5,744 interactions after homologene mapping; a large deviation suggests preparepairs misconfiguration
- Runtime for singlenet on ~10,000 E15.5 cells should be around 12 minutes; Wilcoxon signed-rank comparison against NATMI/CellPhoneDB/SingleCellSignalR yields p < 0.01
- Product-of-sum aggregation variant should yield the fewest interactions and miss rare cell types (endothelial cells, OPCs); regularized localization-aware scoring recovers more secreted-ligand/membrane-receptor interactions than product-of-mean
- Expression changes of inferred L-R pairs should partition into eight Mfuzz fuzzy clusters
- Sankey plot (sankey) of cell-type proportions per time point should be inspected to confirm input cell-type and time-point labels were parsed correctly

## Limitations

- The default L-R gene list size limits detection of some interactions (e.g. MG–Neurons); users must supply supplementary L-R genes as support
- The cause of missing MG–Neurons interactions in individual samples remains unexplained, and alternative ligand-receptor genes mediating MG–Neuron interactions between AD patients are not identified
- The dsCellNet version number is not stated in the article; only dsCellNetv1_1.0.tar.gz from the README is identifiable
- SingleCellSignalR (a comparison baseline) ignored EC interactions and cannot distinguish autocrine interactions with default parameters — careful when benchmarking

## Evidence
