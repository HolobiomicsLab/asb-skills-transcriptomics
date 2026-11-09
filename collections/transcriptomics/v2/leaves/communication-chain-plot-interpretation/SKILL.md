---
name: communication-chain-plot-interpretation
description: Use when a user has CommPath objects (e.g., tumor.obj / normal.obj from
  HCC scRNA-seq) and needs to identify or visualize which LR pairs and ligand→receptor→pathway→downstream-cell
  chains, or their tumor-vs-normal changes, explain signaling through a target cluster.
when_to_use_negative:
- Only global LR-pair counts or interaction intensities between clusters are needed,
  with no pathway mediation — use circosPlot(object, filter=FALSE) instead of chain
  plots
- The receiver-cell pathway activation has not been scored (no scorePath/gsva run),
  so chains would not be grounded in actual pathway activity
- The question concerns intracellular TF-target or ligand-target regulatory potential
  rather than pathway-mediated communication chains (tools like NicheNet or scSeqComm
  are better suited)
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: CommPath
  role: R package providing comparePath, pathPlot.compare, pathChainPlot.compare,
    pathNet, findLigand, findReceptor, circosPlot for inferring and visualizing pathway-mediated
    communication chains
  repo: https://github.com/yingyonghui/CommPath
- name: R
  role: Runtime environment; CommPath installed via devtools::install_github("yingyonghui/CommPath")
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:d78273dcaf8758b716201703366bfd7417fa2860e751a4bcf8446f160cdc4fed
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# communication-chain-plot-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for building and interpreting CommPath's pathway-mediated cell-cell communication chains — reading network/chain plots that connect upstream cell types and LR pairs through activated receiver-cell pathways to downstream responding cell types, and comparing those chains between conditions (e.g., HCC tumor vs normal).

## When to use

Use when a user has CommPath objects (e.g., tumor.obj / normal.obj from HCC scRNA-seq) and needs to identify or visualize which LR pairs and ligand→receptor→pathway→downstream-cell chains, or their tumor-vs-normal changes, explain signaling through a target cluster.

## When NOT to use

- Only global LR-pair counts or interaction intensities between clusters are needed, with no pathway mediation — use circosPlot(object, filter=FALSE) instead of chain plots
- The receiver-cell pathway activation has not been scored (no scorePath/gsva run), so chains would not be grounded in actual pathway activity
- The question concerns intracellular TF-target or ligand-target regulatory potential rather than pathway-mediated communication chains (tools like NicheNet or scSeqComm are better suited)

## Inputs

- Processed CommPath S4 objects (e.g., HCC.tumor.3k.RData and HCC.normal.3k.RData from Figshare) already run through findLRmarker, findLRpairs, findLRpath, scorePath, diffAllPath, filterLR, filterPath
- Library-size normalized, log-transformed gene × cell expression matrices with matching cell identity label vectors
- For two-condition comparison: diff.path.dat from comparePath
- User-selected target cluster (select.ident), upstream/downstream identity vectors (up.ident, down.ident), and pathway names (select.path)

## Outputs

- pathChainPlot.compare.<cluster>.pdf — communication-chain comparison plot with red (upregulated) / blue (downregulated) line widths encoding tumor-vs-normal intensity changes
- pathPlot.compare.<cluster>.pdf — receptor/pathway comparison line plot
- pathway.net slot — integrated statistics of filtered activated pathways and associated LR interactions
- Identified LR pairs and ligands/receptors mediating condition-specific chains (e.g., SPP1/PTGER4 in myeloid→CD8+ T cell communication)

## How to apply

CommPath builds communication chains by integrating, via pathNet, the statistics of filtered activated pathways and their associated LR interactions, then tracing both upstream cell types/ligands (findLigand) and downstream cells/receptors (findReceptor) around a user-selected cluster. For condition comparisons, run comparePath(object.1 = tumor.obj, object.2 = normal.obj, select.ident = <cluster>) to get differentially activated pathways (diff.path.dat), then plot with pathPlot.compare and pathChainPlot.compare, supplying up.ident and down.ident cell-type vectors and select.path pathway names (e.g., 'Gastric cancer', 'Relaxin signaling pathway', 'Axon guidance'). Interpretation rule: in the comparison plots, red line width/color encodes chains upregulated and blue encodes chains downregulated in tumor versus normal, with width proportional to interaction intensity change. The rationale is that unlike LR-only tools (CellPhoneDB, CellChat, scConnect, NATMI, Connectome), chains are anchored to pathway activation scores in receiver cells (gsva via scorePath, or UCell), so a reported chain implies the receptor-proximal pathway is actually active and transduced to downstream cells.

## Related tools

- **CommPath** (R package providing comparePath, pathPlot.compare, pathChainPlot.compare, pathNet, findLigand, findReceptor, circosPlot for inferring and visualizing pathway-mediated communication chains) — https://github.com/yingyonghui/CommPath
- **R** (Runtime environment; CommPath installed via devtools::install_github("yingyonghui/CommPath")) — https://github.com/yingyonghui/CommPath

## Examples

```
pathChainPlot.compare(diff.path.dat, up.ident = c('Myeloid','Endothelial','Fibroblast','Hepatocyte'), down.ident = c('CD8 T','CD4 T','Treg','NK','Endothelial','Fibroblast','Hepatocyte'), select.path = c('Gastric cancer','Relaxin signaling pathway','Axon guidance'))
```

## Evaluation signals

- Both output PDF files (e.g., pathChainPlot.compare.Endothelial.pdf) exist and open as valid PDFs
- Line widths/colors in the comparison plot correctly encode upregulated (red) or downregulated (blue) interaction intensity in tumor versus normal, as described in the methods
- The chains shown pass through pathways from select.path (e.g., 'Relaxin signaling pathway') and the reported LR pairs connect a ligand in an up.ident cluster to a receptor whose pathway activates a down.ident cluster
- Condition-specific findings are biologically consistent — e.g., SPP1/PTGER4 myeloid→CD8+ T cell communication should surface as enhanced in tumor samples
- diff.path.dat from comparePath contains significant differential-pathway statistics for the same cluster (select.ident) across both CommPath objects

## Limitations

- GSVA-based scorePath is time-consuming on large datasets; UCell or average-based activity scoring are offered as alternatives
- Many compared tools (CellPhoneDB, CellChat, scConnect, NATMI, Connectome) infer LR interactions considering only intercellular signaling, so chain-level comparisons are only meaningful within CommPath's pathway-filtered framework
- Chain results depend on pathway database choice (KEGG, WikiPathways, Reactome, GO) and on filterPath removing pathways containing only marker ligands that trigger no pathway in downstream clusters
- Example data are subsampled (3000 cells, top 5000 highly variable genes from tumor and normal tissues), so absolute interaction counts may not generalize to full datasets

## Evidence

- [abstract] CommPath determines cell-cell communication chain by considering both the upstream and downstream cells of user-defined cell populations.: "CommPath determines cell-cell communication chain by considering both the upstream and downstream cells of user-defined cell populations."
- [discussion] Besides the circos plots, CommPath utilizes network graphs and chain plots to illustrate the communication chains between selected cell types: "Besides the circos plots to show the LR interaction patterns among all cell types, CommPath utilizes network graphs and chain plots to illustrate the communication chains between selected cell types"
- [methods] pathNet: First to integrate the statistics of filtered activated pathways and their associated LR interactions: "First to integrate the statistics of filtered activated pathways and their associated LR interactions"
- [other] Run comparePath(object.1 = tumor.obj, object.2 = normal.obj, select.ident = 'Endothelial') to identify differentially activated pathways: [evidence span withheld — anchor status: not_found]
- [other] the LR pair SPP1/PTGER4 mediating myeloid-CD8+ T cell communication was enhanced in tumor samples: [evidence span withheld — anchor status: not_found]
