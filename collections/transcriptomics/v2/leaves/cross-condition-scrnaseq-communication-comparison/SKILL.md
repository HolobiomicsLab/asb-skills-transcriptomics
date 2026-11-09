---
name: cross-condition-scrnaseq-communication-comparison
description: Use when you have two pre-built CommPath objects from matched scRNA-seq
  conditions (e.g., tumor vs normal) and need to identify communication chains up-
  or down-regulated between them.
when_to_use_negative:
- You have only a single condition and want initial LR interaction inference — build
  objects with createCommPath, findLRmarker, findLRpairs, findLRpath, and scorePath
  first rather than comparePath
- Your data are not scRNA-seq expression matrices with cell identity labels (CommPath
  requires normalized, log-transformed gene x cell matrices and cell.info)
- Conditions lack matched cell clusters in both objects — comparePath requires comparing
  the same cluster across the two conditions
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CommPath
  role: R package providing comparePath, pathPlot.compare, and pathChainPlot.compare
    for two-condition differential pathway and communication-chain analysis
  repo: https://github.com/yingyonghui/CommPath
- name: R
  role: Runtime environment; CommPath installed via devtools::install_github('yingyonghui/CommPath')
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:aa314c094837c79534fc2ad5f2bb6648ff8df4ca619675957d37ff41226b97b7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yingyonghui/CommPath
  repo_url: https://github.com/yingyonghui/CommPath
  tools:
  - CommPath
  - R
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-condition-scrnaseq-communication-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares pathway-mediated cell-cell communication between two biological conditions (e.g., HCC tumor vs normal) using CommPath's comparePath, pathPlot.compare, and pathChainPlot.compare functions to detect differentially activated pathways in the same clusters and trace the associated ligand-receptor communication chains.

## When to use

Use when you have two pre-built CommPath objects from matched scRNA-seq conditions (e.g., tumor vs normal) and need to identify communication chains up- or down-regulated between them.

## When NOT to use

- You have only a single condition and want initial LR interaction inference — build objects with createCommPath, findLRmarker, findLRpairs, findLRpath, and scorePath first rather than comparePath
- Your data are not scRNA-seq expression matrices with cell identity labels (CommPath requires normalized, log-transformed gene x cell matrices and cell.info)
- Conditions lack matched cell clusters in both objects — comparePath requires comparing the same cluster across the two conditions

## Inputs

- Two pre-built CommPath S4 objects from distinct conditions (e.g., tumor.obj and normal.obj), each created from a library-size-normalized, log-transformed gene x cell expression matrix plus cell identity labels
- Cluster identity to compare across conditions (select.ident, e.g., 'Endothelial')
- Lists of upstream and downstream cell types (up.ident, down.ident) and pathways of interest (select.path)

## Outputs

- diff.path.dat: dataframe of differentially activated pathways per cluster between the two conditions
- pathPlot.compare line plot PDF (e.g., pathPlot.compare.Endothelial.pdf) of receptor/pathway comparison
- pathChainPlot.compare chain plot PDF (e.g., pathChainPlot.compare.Endothelial.pdf) where line widths/colors encode interaction intensity changes upregulated (red) or downregulated (blue) in tumor vs normal

## How to apply

Load two pre-processed CommPath objects (e.g., HCC.tumor.3k.RData and HCC.normal.3k.RData from Figshare) with library(CommPath). Run comparePath(object.1 = tumor.obj, object.2 = normal.obj, select.ident = <cluster>) to identify differentially activated pathways between the same cell cluster in the two conditions, storing results in diff.path.dat. Visualize receptor/pathway changes with pathPlot.compare (e.g., up.ident = c('Treg','Myeloid','Endothelial','Fibroblast','Hepatocyte')). Trace condition-specific communication chains with pathChainPlot.compare, supplying upstream (up.ident) and downstream (down.ident) cell types and select.path (e.g., 'Gastric cancer', 'Relaxin signaling pathway', 'Axon guidance'). Rationale: CommPath uniquely interprets LR interactions via activated pathways in receiver cells (scored with GSVA or UCell) and infers chains considering both upstream and downstream cells, unlike CellPhoneDB or CellChat which consider only intercellular signaling.

## Related tools

- **CommPath** (R package providing comparePath, pathPlot.compare, and pathChainPlot.compare for two-condition differential pathway and communication-chain analysis) — https://github.com/yingyonghui/CommPath
- **R** (Runtime environment; CommPath installed via devtools::install_github('yingyonghui/CommPath')) — https://github.com/yingyonghui/CommPath

## Examples

```
library(CommPath); diff.path.dat <- comparePath(object.1 = tumor.obj, object.2 = normal.obj, select.ident = 'Endothelial')
```

## Evaluation signals

- Both output PDF files (pathPlot.compare.*.pdf and pathChainPlot.compare.*.pdf) exist and open as valid PDFs
- Line widths and colors in the comparison plots encode interaction intensity changes: red for upregulated, blue for downregulated chains in tumor vs normal
- diff.path.dat contains differential pathway statistics for the selected cluster (select.ident) present in both condition objects
- Biological sanity check: in HCC application, the SPP1/PTGER4 LR pair mediating myeloid–CD8+ T cell communication should appear as significantly upregulated in tumor versus normal CD8+ T cells

## Limitations

- GSVA-based pathway scoring is time-consuming on large datasets; the UCell algorithm is offered as an alternative
- Example analyses are limited to ~3000 cells across the top 5000 highly variable genes for computational tractability
- Many competing tools (CellPhoneDB, CellChat, scConnect, NATMI, Connectome) infer LR interactions from intercellular signaling only; CommPath's chain inference depends on curated pathway databases (KEGG, WikiPathways, Reactome, GO), so pathway coverage bounds interpretation

## Evidence

- [results] two-condition comparison workflow: "we utilized CommPath to compare the communication chains involving CD8+ T cells between tumor and normal tissues"
- [abstract] chain inference rationale: [evidence span withheld — anchor status: not_found]
- [methods] comparePath parameters from source card: [evidence span withheld — anchor status: not_found]
- [readme] installation from README: "CommPath R package can be easily installed from Github using devtools:
devtools::install_github("yingyonghui/CommPath")"
- [discussion] limitations of competing tools: "many of these methods infer LR interactions only considering the intercellular signaling, including CellPhoneDB [4], CellChat [6], scConnect [7], NATMI [23], and Connectome [24]."
