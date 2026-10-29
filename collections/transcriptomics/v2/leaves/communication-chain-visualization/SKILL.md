---
name: communication-chain-visualization
description: Use when a CommPath object with filtered LR interactions and differentially
  activated pathways exists and you need to display the Upstream-Receptor-Pathway-Ligand-Downstream
  chain through a chosen central cluster.
when_to_use_negative:
- When only unfiltered LR interactions exist (filter=FALSE circos plots) and no pathway
  filtering (filterLR/filterPath) has been performed — chain inference requires pathways
  activated in receiver cells
- When the goal is whole-cohort LR interaction counts/intensity summaries rather than
  a chain through a specific central cluster — circosPlot is the appropriate visual
  instead
- When the input is not a CommPath S4 object with pathway statistics integrated (pathway.net
  slot); CommPath chain functions cannot run on raw expression matrices alone
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0089
tools:
- name: CommPath
  role: R package supplying pathPlot and pathChainPlot to render pathway-mediated
    communication chains (Upstream-Receptor-Pathway-Ligand-Downstream) from the filtered
    pathway/LR statistics in the CommPath object
  repo: https://github.com/yingyonghui/CommPath
- name: R
  role: Runtime environment for CommPath; pdf()/dev.off() used to write the chain
    plot to a sized PDF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:5dedfbc188615b4b117e461ff57e89377a5158e7c72b66b669449c8421a998b8
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
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# communication-chain-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize the pathway-mediated cell-cell communication chain inferred by CommPath, rendering upstream ligand→central receptor→activated pathway→central ligand→downstream receptor links as chain plots for a user-defined central cell cluster.

## When to use

Use when a CommPath object with filtered LR interactions and differentially activated pathways exists and you need to display the Upstream-Receptor-Pathway-Ligand-Downstream chain through a chosen central cluster.

## When NOT to use

- When only unfiltered LR interactions exist (filter=FALSE circos plots) and no pathway filtering (filterLR/filterPath) has been performed — chain inference requires pathways activated in receiver cells
- When the goal is whole-cohort LR interaction counts/intensity summaries rather than a chain through a specific central cluster — circosPlot is the appropriate visual instead
- When the input is not a CommPath S4 object with pathway statistics integrated (pathway.net slot); CommPath chain functions cannot run on raw expression matrices alone

## Inputs

- Pre-processed CommPath S4 object (e.g., tumor.obj loaded from HCC.tumor.3k.RData) with filtered LR interactions (interact.filter) and differentially activated pathways (acti.path.filtered.dat)
- User-defined central cluster identity (select.ident, e.g., 'Endothelial')
- Vectors of upstream (up.ident) and downstream (down.ident) cell clusters, e.g., Treg/Myeloid/Endothelial/Fibroblast/Hepatocyte and B/Myeloid/Endothelial/Fibroblast/Hepatocyte
- Named pathways of interest (select.path), e.g., KEGG JAK-STAT, MAPK, Ras, PI3K-Akt, AMPK signaling pathway

## Outputs

- pathPlot figure showing upstream clusters connected via receptors and differentially activated pathways in the central cluster
- pathChainPlot figure with Upstream-Receptor-Pathway-Ligand-Downstream columns and line widths proportional to interaction intensity
- PDF file (e.g., pathChainPlot.Endothelial.pdf, 6x14 inches) written via pdf()/dev.off()

## How to apply

Load the pre-processed CommPath object (e.g., HCC.tumor.3k.RData from the figshare download of GSE156337-derived HCC scRNA-seq data), which already contains filtered LR interactions and differentially activated pathways (acti.path.filtered.dat). Run CommPath::pathPlot on the central cluster (select.ident, e.g., 'Endothelial') to show upstream clusters connected via receptors and their differentially activated pathways. Then run CommPath::pathChainPlot with the central cluster, explicit up.ident and down.ident vectors, and select.path naming pathways of interest (e.g., JAK-STAT, MAPK, Ras, PI3K-Akt, AMPK signaling) to render the full chain. Write the plot to a sized PDF (e.g., 6x14 inches via pdf()/dev.off()); line widths are proportional to overall interaction intensity as specified by the pathChainPlot legend. The rationale is that CommPath considers both upstream LR pairs provoking pathways in the central cells and downstream cells responding to those same pathways, so chains must be visualized after filterLR/filterPath steps.

## Related tools

- **CommPath** (R package supplying pathPlot and pathChainPlot to render pathway-mediated communication chains (Upstream-Receptor-Pathway-Ligand-Downstream) from the filtered pathway/LR statistics in the CommPath object) — https://github.com/yingyonghui/CommPath
- **R** (Runtime environment for CommPath; pdf()/dev.off() used to write the chain plot to a sized PDF)

## Examples

```
pdf('pathChainPlot.Endothelial.pdf',height=6,width=14); pathChainPlot(object=tumor.obj, select.ident='Endothelial', up.ident=c('Treg','Myeloid','Endothelial','Fibroblast','Hepatocyte'), down.ident=c('B','Myeloid','Endothelial','Fibroblast','Hepatocyte'), select.path=c('JAK-STAT','MAPK','Ras','PI3K-Akt','AMPK signaling pathway')); dev.off()
```

## Evaluation signals

- The output PDF (pathChainPlot.Endothelial.pdf, 6x14 inches) exists after dev.off()
- The rendered figure depicts the Upstream-Receptor-Pathway-Ligand-Downstream columns
- Line widths in the chain plot are proportional to overall interaction intensity, matching the pathChainPlot legend
- Only pathways named in select.path (e.g., JAK-STAT, MAPK, Ras, PI3K-Akt, AMPK) appear as mediating pathways in the chain
- Upstream and downstream clusters shown match the up.ident and down.ident vectors passed to pathChainPlot

## Limitations

- Chain visualization depends on prior pathway filtering: pathways containing only specific marker ligands that do not trigger any pathway in downstream clusters are removed by filterPath
- Many compared tools (CellPhoneDB, CellChat, scConnect, NATMI, Connectome) infer LR interactions only considering intercellular signaling and cannot infer the communication chain among further downstream cells — this skill is CommPath-specific
- GSVA-based pathway scoring is time-consuming on large datasets (UCell is offered as an alternative), which limits how large the input object can be before visualization
- No version numbers are stated for CommPath, so function signatures/arguments may vary across installs

## Evidence

- [discussion] CommPath utilizes network graphs and chain plots to illustrate the communication chains between selected cell types: "CommPath utilizes network graphs and chain plots to illustrate the communication chains between selected cell types"
- [intro] it infers pathway-mediated cell-cell communication chain by considering not only the upstream cell types and LR pairs provoking the specific pathways: "it infers pathway-mediated cell-cell communication chain by considering not only the upstream cell types and LR pairs provoking the specific pathways"
- [methods] we next filter the pathways by removing those pathways containing only specific marker ligands which do not triger any pathway in the downstream clusters: "we next filter the pathways by removing those pathways containing only specific marker ligands which do not triger any pathway in the downstream clusters"
- [readme] load("path_to_download/HCC.tumor.3k.RData"): "load("path_to_download/HCC.tumor.3k.RData")"
- [readme] The example data are available in figshare (HCC_tumor_normal_3k_RData/19090553): "The example data are available in [figshare](https://figshare.com/articles/dataset/HCC_tumor_normal_3k_RData/19090553)"
