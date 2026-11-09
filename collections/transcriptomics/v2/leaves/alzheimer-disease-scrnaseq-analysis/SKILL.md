---
name: alzheimer-disease-scrnaseq-analysis
description: Use when analyzing two-group (case vs control) single-cell RNA-seq data
  — e.g., late-stage AD vs healthy human brain — to infer differential cell-cell communication
  networks and rank the most active communicating cell types.
when_to_use_negative:
- Input is developmental series scRNA-seq (multiple sequential time points) where
  the goal is temporal trajectory analysis via developcorrelation and fuzzy clustering
  rather than a two-group case/control comparison
- The dataset is not membrane/secreted protein-mediated communication analysis — e.g.,
  intracellular signaling or gene regulatory network inference, since dsCellNet identifies
  interactions according to protein localizations
- Only a single condition is available and no control/treatment time-point pair exists
  for comparednet
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: dsCellNet
  role: Core R package providing datainput, definecolo, preparepairs, singlenet, comparednet,
    and importance_plot to infer and compare AD-vs-control cell-cell communication
    networks
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Runtime environment for the dsCellNet package (R>=4.0.3) with dependencies
    igraph, homologene, Hmisc, ggpubr, pheatmap, reshape2, ggalluvial, tseries, magrittr
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:2de43cc03b58616531c7b7909015c9bcbde73600900d426863da3fa66ed54b5e
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
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# alzheimer-disease-scrnaseq-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply the dsCellNet R package's two-group comparison (comparednet) to human brain scRNA-seq from six late-stage Alzheimer's disease patients and six healthy controls (GSE174367) to identify which immune cell types drive interactions increased in AD brains.

## When to use

Use when analyzing two-group (case vs control) single-cell RNA-seq data — e.g., late-stage AD vs healthy human brain — to infer differential cell-cell communication networks and rank the most active communicating cell types.

## When NOT to use

- Input is developmental series scRNA-seq (multiple sequential time points) where the goal is temporal trajectory analysis via developcorrelation and fuzzy clustering rather than a two-group case/control comparison
- The dataset is not membrane/secreted protein-mediated communication analysis — e.g., intracellular signaling or gene regulatory network inference, since dsCellNet identifies interactions according to protein localizations
- Only a single condition is available and no control/treatment time-point pair exists for comparednet

## Inputs

- Gene expression matrix file (human brain scRNA-seq, six late-stage AD patients and six healthy controls, GSE174367)
- Cell-type label file (Type column)
- Time-point / group label file (Time column encoding AD vs control)
- Human membrane/secreted ligand-receptor database path (dbloc)
- Result directory path (resloc) and result prefix (mainlab)

## Outputs

- Per-group cell-cell interaction networks inferred by singlenet
- AD-vs-control differential interaction comparison from comparednet (interactions increased in AD brains)
- Incoming, outgoing, and total per-cell-type communication scores (importance plots)
- cell_importance_table.csv ranking cell types by communication score
- GO enrichment of interactions increased in AD identifying immune-cell contributions to cytokine production and glutamate secretion

## How to apply

Load the AD/control expression, cell-type, and time-point files with datainput(datloc, expnam, typnam, timnam, "FALSE", resloc), coercing outputs to data frames (inpexp, inptyp, inptim). Define a consistent cell-type color mapping with definecolo(tsccnetcol, inptyp$Type, resloc, mainlab) and optionally summarize per-time-point cell-type proportions with sankey. Build the membrane/secreted ligand-receptor pair set from the expression matrix via preparepairs(inpexp, species, dbloc, mainlab) with species set to human, then infer single-time-point cell-cell interaction networks for each group with singlenet(...). Run the two-group comparison with comparednet(controltime, treattime, usedcolo, resloc), setting the healthy-control time-point as controltime and the AD group as treattime. Finally, compute incoming/outgoing/total per-cell-type communication scores with importance_plot(resloc, "row", scoring, "None") for each score type and compile cell_importance_table.csv; the analysis is judged correct when MG and ASCs rank as the top communicating immune cell types and the AD interaction list contains IL8, IL18, NRP1, CX3CR1, and TLR1/2.

## Related tools

- **dsCellNet** (Core R package providing datainput, definecolo, preparepairs, singlenet, comparednet, and importance_plot to infer and compare AD-vs-control cell-cell communication networks) — github.com/songzh523/dsCellNet
- **R** (Runtime environment for the dsCellNet package (R>=4.0.3) with dependencies igraph, homologene, Hmisc, ggpubr, pheatmap, reshape2, ggalluvial, tseries, magrittr) — github.com/songzh523/dsCellNet

## Examples

```
out<-datainput(datloc,expnam,typnam,timnam,"FALSE",resloc); comparednet(controltime,treattime,usedcolo,resloc); importance_plot(resloc,"row","total","None")
```

## Evaluation signals

- The AD importance table (cell_importance_table.csv) ranks microglia (MG) and astrocytes (ASCs) as the top communicating immune cell types
- The inferred AD interaction list contains IL8, IL18, NRP1, CX3CR1, and TLR1/2
- GO enrichment of AD-increased interactions recovers cytokine production and glutamate secretion terms consistent with neuroinflammation
- datainput returns three data frames (inpexp, inptyp, inptim) whose dimensions match the input expression, cell-type, and time-point files
- comparednet runs with controltime set to the healthy-control time-point and treattime set to the AD group, producing non-empty differential interaction output

## Limitations

- The default L-R gene list size limits detection of some interactions (e.g., MG–Neurons); users can supply supplementary L-R genes
- The cause of missing MG–Neurons interactions in individual samples remains unexplained
- Alternative ligand-receptor genes mediating MG–Neuron interactions between AD patients are not identified, implying complex mechanisms among AD patients
- dsCellNet version number is not stated in the article

## Evidence

- [other] GO enrichment of interactions increased in Alzheimer's brains demonstrates that microglia (MG) and astrocytes (ASCs) are the primary immune cells contributing to neuroinflammation involving cytokine production and glutamate secretion in AD.: [evidence span withheld — anchor status: not_found]
- [results] Article results describing the AD dataset.: "we used scRNA-seq data from human brains of six late-stage Alzheimer’s disease patients and six healthy controls"
- [readme] README example invocation for the comparison step.: "comparednet(controltime,treattime,usedcolo,resloc)"
- [readme] README example invocation for importance scoring.: "importance_plot(resloc,"row","incoming","None")"
- [discussion] Discussion limitation on default L-R gene list.: "This maybe owing to the limitation that the size of default L-R genes, users can support their L-R genes as supplementary"
