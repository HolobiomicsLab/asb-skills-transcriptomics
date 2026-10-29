---
name: ligand-receptor-interaction-pathway-filtering
description: Use when scRNA-seq data (e.g., library-size normalized, log-transformed
  gene-by-cell matrix plus cell labels) has candidate LR pairs and you must prioritize
  LR interactions by downstream pathway activation in receiver clusters.
when_to_use_negative:
- You only need expression-based LR inference without functional prioritization —
  plain findLRpairs results (filter=FALSE circos plots) suffice, as done by CellPhoneDB/CellChat-style
  intercellular-only inference
- Your goal is inferring ligand-target regulatory potential from prior knowledge (NicheNet-style)
  rather than pathway-activity-based screening in receiver cells
- The expression matrix is not library-size normalized and log-transformed, or cell
  labels do not match cell order — CommPath input assumptions would be violated
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CommPath
  role: R package implementing marker LR identification (findLRmarker), LR pair inference
    (findLRpairs), pathway assignment (findLRpath), pathway scoring (scorePath), differential
    pathway activation (diffAllPath), and LR/pathway filtering (filterLR, filterPath)
  repo: https://github.com/yingyonghui/CommPath
- name: GSVA
  role: Algorithm used by scorePath(method='gsva', min.size=10) to compute per-cell
    pathway activation scores
  repo: https://www.bioconductor.org/packages/release/bioc/html/GSVA.html
- name: R
  role: Runtime environment; CommPath is installed via devtools::install_github('yingyonghui/CommPath')
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:4f8c02fedf351215a060402693ddc812c1cd61d5fc2003af75dad24bd564b669
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
  - GSVA
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ligand-receptor-interaction-pathway-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A CommPath-based skill that screens statistically inferred ligand-receptor (LR) associations against pathway activation in receiver cells, retaining only LR interactions tied to significantly activated pathways (KEGG, WikiPathways, Reactome, GO) to prioritize functional signaling.

## When to use

Use when scRNA-seq data (e.g., library-size normalized, log-transformed gene-by-cell matrix plus cell labels) has candidate LR pairs and you must prioritize LR interactions by downstream pathway activation in receiver clusters.

## When NOT to use

- You only need expression-based LR inference without functional prioritization — plain findLRpairs results (filter=FALSE circos plots) suffice, as done by CellPhoneDB/CellChat-style intercellular-only inference
- Your goal is inferring ligand-target regulatory potential from prior knowledge (NicheNet-style) rather than pathway-activity-based screening in receiver cells
- The expression matrix is not library-size normalized and log-transformed, or cell labels do not match cell order — CommPath input assumptions would be violated

## Inputs

- Library-size normalized, log-transformed expression matrix (gene x cell), e.g. tumor.expr from HCC.tumor.3k.RData (GSE156337 HCC scRNA-seq)
- Vector of cell identity labels matching cell order, e.g. tumor.label
- Pathway annotations database choice: 'kegg', wikipathways, reactome, or GO

## Outputs

- acti.path.dat dataframe with mean.diff, t, P.val, P.val.adj and ligand.in.path/receptor.in.path columns per cluster
- Filtered LR interactions (tumor.obj@interact.filter) associated with significantly activated pathways
- Pathway-LR network statistics (tumor.obj@pathway.net) and visualization PDFs such as pathHeatmap.pdf and circos plots

## How to apply

Create a CommPath object with createCommPath(species='hsapiens') from a normalized gene-by-cell matrix and matching cell labels, identify marker ligands/receptors with findLRmarker (e.g., method='wilcox.test'), and infer significant LR pairs with findLRpairs(logFC.thre=0, p.thre=0.01). Assign pathway genesets overlapping the marker ligands/receptors via findLRpath(category='kegg') and quantify per-cell pathway activation with scorePath(method='gsva', min.size=10) (UCell is an alternative when GSVA is too slow). Run diffAllPath(only.posi=TRUE, only.sig=TRUE) to obtain acti.path.dat with mean.diff, t, P.val, and P.val.adj statistics versus other user-defined clusters, then use filterLR to retain LR interactions associated with activated pathways. Finally apply filterPath to remove pathways containing only specific marker ligands that do not trigger any pathway in downstream clusters; visualize with pathHeatmap(top.n.pathway=10, cell.aver=TRUE) or circosPlot(filter=TRUE).

## Related tools

- **CommPath** (R package implementing marker LR identification (findLRmarker), LR pair inference (findLRpairs), pathway assignment (findLRpath), pathway scoring (scorePath), differential pathway activation (diffAllPath), and LR/pathway filtering (filterLR, filterPath)) — https://github.com/yingyonghui/CommPath
- **GSVA** (Algorithm used by scorePath(method='gsva', min.size=10) to compute per-cell pathway activation scores) — https://www.bioconductor.org/packages/release/bioc/html/GSVA.html
- **R** (Runtime environment; CommPath is installed via devtools::install_github('yingyonghui/CommPath')) — https://github.com/yingyonghui/CommPath

## Examples

```
tumor.obj <- findLRpath(object=tumor.obj, category="kegg"); tumor.obj <- scorePath(object=tumor.obj, method="gsva", min.size=10); tumor.obj <- diffAllPath(object=tumor.obj, only.posi=TRUE, only.sig=TRUE)
```

## Evaluation signals

- acti.path.dat contains mean.diff, t, P.val and P.val.adj columns and only significant (only.sig=TRUE) positive (only.posi=TRUE) pathways per cluster
- pathHeatmap.pdf exists, runs without error, and displays top differentially activated pathways per cluster consistent with acti.path.dat significance calls
- Every retained LR interaction in tumor.obj@interact.filter maps to a pathway with significant activation in the receiver cluster (ligand.in.path/receptor.in.path populated)
- filterPath removes pathways lacking downstream-triggering ligands, so no retained pathway contains only specific marker ligands that trigger nothing in downstream clusters
- Pathway scoring succeeded for genesets of size >= min.size=10 (GSVA on min-size-filtered genesets)

## Limitations

- GSVA pathway scoring is time-consuming on large datasets; UCell (PMID: 34285779) or average-based assessment are offered as alternatives
- Many competing tools (CellPhoneDB, CellChat, scConnect, NATMI, Connectome) infer LR interactions only from intercellular signaling, so comparison outputs may lack receiver-cell pathway context
- Paths can contain pathways whose only ligands are specific markers that trigger no pathway downstream; filterPath must be applied to prune these
- No version numbers are stated for CommPath or compared tools, and no changelog is available for the repository

## Evidence
