---
name: developmental-series-scrnaseq-analysis
description: Use when you have time-series single-cell RNA-seq spanning multiple developmental
  or aging time points (e.g., GSE153164 corticogenesis series) and need to infer L-R
  interactions and how they change across time points.
when_to_use_negative:
- Input is a single-time-point scRNA-seq dataset with no temporal (developmental series)
  dimension — dsCellNet's DTW and fuzzy clustering steps require multi-time-point
  data
- The question requires a static cell-cell interaction map only, without identifying
  key time points or dynamic interaction changes
- The L-R pairs of interest are absent from the 5,744-pair default database and no
  supplementary user-supplied L-R list is available (e.g., some MG–Neurons interactions
  were missed)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dsCellNet
  role: 'R package implementing the full pipeline: datainput, preparepairs, singlenet,
    developcorrelation, comparednet, importance_plot'
  repo: github.com/songzh523/dsCellNet
- name: Mfuzz
  role: Fuzzy c-means clustering of interaction expression changes into eight clusters
    after min.std pre-filtering; mestimate for fuzzifier estimation
- name: homologene
  role: Infers L-R pairs across species using default human/mouse homologs
- name: R
  role: Runtime environment; dsCellNet is an open-access R package requiring R >=
    4.0.3
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:ee2fe04e723d4d19b140bd7ed5385f7b6fc0241498e32fc878304bd936151ea8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - Mfuzz
  - R
  - dsCellNet
  - homologene
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# developmental-series-scrnaseq-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer dynamic cell-cell communication networks from developmental-series scRNA-seq data using dsCellNet, which identifies ligand-receptor interactions by protein localization, reinforces them with dynamic time warping (DTW), and clusters interaction expression changes with Mfuzz fuzzy c-means (eight clusters) to pinpoint key time points and mediator cell types.

## When to use

Use when you have time-series single-cell RNA-seq spanning multiple developmental or aging time points (e.g., GSE153164 corticogenesis series) and need to infer L-R interactions and how they change across time points.

## When NOT to use

- Input is a single-time-point scRNA-seq dataset with no temporal (developmental series) dimension — dsCellNet's DTW and fuzzy clustering steps require multi-time-point data
- The question requires a static cell-cell interaction map only, without identifying key time points or dynamic interaction changes
- The L-R pairs of interest are absent from the 5,744-pair default database and no supplementary user-supplied L-R list is available (e.g., some MG–Neurons interactions were missed)

## Inputs

- Gene expression matrix file (filename given via expnam)
- Cell-type label file (filename given via typnam)
- Time-point information file (filename given via timnam, optional)
- Species name for L-R database homolog mapping (e.g., human/mouse via homologene)
- Control time-point name and comparison target for comparednet

## Outputs

- Per-time-point cell-cell interaction networks with significance scores for membrane and secreted L-R genes
- Developmental-series interaction correlation matrix from developcorrelation
- Mfuzz fuzzy clustering membership/assignment table (e.g., mfuzz_cluster_assignments.csv) partitioning L-R pairs into eight clusters
- Cell-type importance scores (incoming/outgoing/total) per time point
- Sankey plots and cell-type proportion color definitions (definecolo, sankey)

## How to apply

Provide three input files (gene expression, cell-type labels, time-point information) via dsCellNet's datainput, then run preparepairs to build the multi-species supported membrane/secreted L-R database (5,744 default interactions from Ramilowski, CellPhoneDB, NATMI, SingleCellSignalR) and singlenet to infer per-time-point networks by separately computing significance of membrane vs secreted L-R genes. Run developcorrelation to compute DTW-based correlation of interaction expression changes within each time point and between adjacent time points, which filters false positives and highlights developmentally dynamic interactions. Standardize the interaction expression-change matrix, pre-filter low-variance L-R pairs with Mfuzz's min.std, estimate the fuzzifier with mestimate, and run Mfuzz fuzzy c-means with eight clusters; assign each L-R pair to its highest-membership cluster. Optionally use comparednet against a control time point and importance_plot to score incoming/outgoing/total activity of each cell type per time point.

## Related tools

- **dsCellNet** (R package implementing the full pipeline: datainput, preparepairs, singlenet, developcorrelation, comparednet, importance_plot) — github.com/songzh523/dsCellNet
- **Mfuzz** (Fuzzy c-means clustering of interaction expression changes into eight clusters after min.std pre-filtering; mestimate for fuzzifier estimation)
- **homologene** (Infers L-R pairs across species using default human/mouse homologs)
- **R** (Runtime environment; dsCellNet is an open-access R package requiring R >= 4.0.3) — github.com/songzh523/dsCellNet

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab); developcorrelation(inptim,usedcolo,resloc)
```

## Evaluation signals

- Mfuzz output partitions the L-R pairs into exactly eight distinct clusters
- Every candidate L-R pair is assigned to its maximum-membership cluster in the membership/assignment table
- Cluster-to-time-point interpretation is coherent (e.g., cluster 1 genes highly expressed at E12.5) and consistent with the DTW-correlation interaction matrix from developcorrelation
- The default L-R database contains 5,744 interactions (2,557 Ramilowski + 1,396 CellPhoneDB + 4,071 NATMI + 3,251 SingleCellSignalR after merging)
- definecolo/sankey outputs show sensible cell-type proportions per time point from the cell-type label file

## Limitations

- dsCellNet version number is not stated in the article
- Detection of some interactions is limited by the size of the default L-R gene list; cause of missing MG–Neurons interactions in individual samples remains unexplained, though users can supply supplementary L-R genes
- Published methods, including dsCellNet's context, cannot distinguish membrane-ligand vs secreted-ligand receptor interaction expression profiles without the separate significance calculations dsCellNet adds
- Requires R >= 4.0.3 with specific dependency versions (igraph >= 1.2.6, homologene >= 1.4.68, Hmisc >= 4.5.0, ggpubr >= 0.4.0, etc.)

## Evidence
