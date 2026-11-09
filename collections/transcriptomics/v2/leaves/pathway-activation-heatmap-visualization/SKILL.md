---
name: pathway-activation-heatmap-visualization
description: Use when a CommPath object already contains scored pathways (scorePath)
  and differential activation results (diffAllPath), and you need to visualize the
  top differentially activated pathways per cluster as a heatmap PDF.
when_to_use_negative:
- Before pathway scores exist — the object lacks scorePath/diffAllPath results, so
  there is nothing differentially activated to display
- When the goal is to show LR pair counts or interaction intensities among clusters
  — use circosPlot (filter=FALSE or TRUE) instead of a pathway heatmap
- When pathways are only annotated (findLRpath) but not yet scored, since the heatmap
  reflects activation levels, not mere geneset overlap
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CommPath
  role: R package providing pathHeatmap to render the top differentially activated
    pathways per cluster to a PDF heatmap
  repo: https://github.com/yingyonghui/CommPath
- name: GSVA
  role: Algorithm used by scorePath to compute per-cell pathway activation scores
    that feed the differential testing visualized by the heatmap
- name: R
  role: Runtime environment in which the CommPath heatmap rendering is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:f0a5a340332ce2746f5e4b57e386bcc469a1249b98b96a63056e6168d0e0edf6
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pathway-activation-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render the differentially activated signaling pathways per cell cluster from a CommPath analysis as a heatmap (pathHeatmap), so that top pathway activation differences (e.g., KEGG pathways scored by GSVA) can be inspected alongside the acti.path.dat significance statistics.

## When to use

Use when a CommPath object already contains scored pathways (scorePath) and differential activation results (diffAllPath), and you need to visualize the top differentially activated pathways per cluster as a heatmap PDF.

## When NOT to use

- Before pathway scores exist — the object lacks scorePath/diffAllPath results, so there is nothing differentially activated to display
- When the goal is to show LR pair counts or interaction intensities among clusters — use circosPlot (filter=FALSE or TRUE) instead of a pathway heatmap
- When pathways are only annotated (findLRpath) but not yet scored, since the heatmap reflects activation levels, not mere geneset overlap

## Inputs

- CommPath S4 object (created with createCommPath) with pathway activation scores from scorePath(method='gsva', min.size=10)
- acti.path.dat differential pathway activation table from diffAllPath(only.posi=TRUE, only.sig=TRUE) containing mean.diff, t, P.val, P.val.adj

## Outputs

- pathHeatmap.pdf — heatmap of the top (e.g., 10) differentially activated pathways per cell cluster, averaged over cells (cell.aver=TRUE)

## How to apply

After running findLRpath(category='kegg') and scorePath(method='gsva', min.size=10), obtain the differential pathway activation table via diffAllPath(only.posi=TRUE, only.sig=TRUE), which yields acti.path.dat with mean.diff, t, P.val, and P.val.adj statistics plus ligand.in.path/receptor.in.path columns. Then call pathHeatmap on the CommPath object with top.n.pathway=10 and cell.aver=TRUE to plot the top differentially activated pathways per cluster, writing the figure to pathHeatmap.pdf. The heatmap rows/columns should be consistent with the significance calls in acti.path.dat under the adjusted P-value threshold implied by only.sig=TRUE, providing a visual check that pathway activity scoring and differential testing were applied coherently before downstream filterLR/filterPath screening.

## Related tools

- **CommPath** (R package providing pathHeatmap to render the top differentially activated pathways per cluster to a PDF heatmap) — https://github.com/yingyonghui/CommPath
- **GSVA** (Algorithm used by scorePath to compute per-cell pathway activation scores that feed the differential testing visualized by the heatmap)
- **R** (Runtime environment in which the CommPath heatmap rendering is executed)

## Examples

```
pdf('pathHeatmap.pdf'); pathHeatmap(object=tumor.obj, top.n.pathway=10, cell.aver=TRUE); dev.off()
```

## Evaluation signals

- pathHeatmap.pdf exists after the call and the R session ran without errors
- The pathways displayed correspond to the top rows of acti.path.dat ranked by differential activation statistics (P.val.adj threshold applied by only.sig=TRUE)
- Number of plotted pathways per cluster matches top.n.pathway=10 (or fewer if fewer significant pathways)
- Cluster labels in the heatmap match the cell identities in the CommPath object's cell.info
- With cell.aver=TRUE, heatmap values are cluster-level averages of the per-cell GSVA pathway activation scores

## Limitations

- GSVA is time-consuming on large datasets; CommPath offers UCell (PMID: 34285779) as an alternative scoring method before visualization
- The heatmap only summarizes the top N pathways; pathways pruned by filterPath (those lacking downstream-triggering ligands) will not be represented
- Only pathways passing the adjusted significance threshold (only.sig=TRUE) are candidates for display; non-significant activation differences are invisible

## Evidence

- [results] heatmaps of top activated pathways visualize pathway-associated LR interactions: [evidence span withheld — anchor status: not_found]
