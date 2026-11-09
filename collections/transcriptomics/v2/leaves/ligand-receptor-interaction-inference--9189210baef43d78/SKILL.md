---
name: ligand-receptor-interaction-inference
description: Use when the goal is to infer cell-cell communication from normalized
  single-cell RNA-seq expression matrices plus cell labels, especially to identify
  pathway-mediated LR interactions (e.g., CXCL12/CXCR4 between CD8+ T cells and endothelial
  cells/fibroblasts in HCC).
when_to_use_negative:
- The expression matrix is raw counts and has not been normalized by library size
  and log-transformed, since CommPath expects normalized expression values
- The goal is intercellular signaling inference only, without examining activated
  pathways in receiver cells or downstream communication chains, where lighter tools
  like CellPhoneDB or CellChat suffice
- The organism is not among the species for which CommPath provides LR and pathway
  databases (human, mouse, rat, zebrafish, fruitfly, worm)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CommPath
  role: Primary R package used to create CommPath objects, identify LR markers, infer
    significant LR pairs, score pathways, filter functional interactions, and visualize
    communication networks and chains
  repo: https://github.com/yingyonghui/CommPath
- name: GSVA
  role: Algorithm invoked by scorePath to compute per-cell pathway activation scores
    (min.size = 10); UCell is an alternative
- name: R
  role: Runtime environment; CommPath is installed from GitHub via devtools::install_github
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:887bbdcc044d88782da6bbc784ec954c0ec85bc0c99d6853b58bac01f5ddb77e
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ligand-receptor-interaction-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer and visualize pathway-mediated ligand-receptor interactions and cell-cell communication chains from scRNA-seq data using the CommPath R package, which prioritizes functional LR interactions by considering signaling pathway activation in receiver cells.

## When to use

Use when the goal is to infer cell-cell communication from normalized single-cell RNA-seq expression matrices plus cell labels, especially to identify pathway-mediated LR interactions (e.g., CXCL12/CXCR4 between CD8+ T cells and endothelial cells/fibroblasts in HCC).

## When NOT to use

- The expression matrix is raw counts and has not been normalized by library size and log-transformed, since CommPath expects normalized expression values
- The goal is intercellular signaling inference only, without examining activated pathways in receiver cells or downstream communication chains, where lighter tools like CellPhoneDB or CellChat suffice
- The organism is not among the species for which CommPath provides LR and pathway databases (human, mouse, rat, zebrafish, fruitfly, worm)

## Inputs

- Normalized (library-size normalized and log-transformed) gene x cell expression matrix (e.g., tumor.expr from HCC_tumor_normal_3k.RData)
- Vector of cell identity labels matching cell order (e.g., tumor.label)
- CommPath LR and pathway databases for the chosen species (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegus)

## Outputs

- CommPath S4 object with slots data, cell.info, meta.info, LR.marker, interact, interact.filter, pathway, and pathway.net
- Significant LR pair statistics (tumor.obj@interact[['InteractNumer']] and [['InteractGene']])
- Circos plots of LR interaction counts and intensities (e.g., circosPlot.count.nonfiltered.pdf)
- Network graphs, chain plots, and heatmaps of top differentially activated pathways (e.g., pathNet.upstream.Endothelial.1.pdf, pathHeatmap with top.n.pathway = 10, cell.aver = TRUE)

## How to apply

Create a CommPath object from a library-size-normalized, log-transformed gene-by-cell expression matrix and matching cell labels with createCommPath (species = 'hsapiens'). Identify marker ligands and receptors per cluster using findLRmarker with the Wilcoxon rank-sum test, then call findLRpairs with logFC.thre = 0 and p.thre = 0.01 to obtain statistically significant LR pairs. Run pathway enrichment with findLRpath (category = 'kegg'), score per-cell pathway activation with scorePath (method = 'gsva', min.size = 10), and obtain significantly up-regulated pathways per cluster with diffAllPath (only.posi = TRUE, only.sig = TRUE). Filter functional LR interactions with filterLR and pathways with filterPath, integrate statistics with pathNet, and visualize with circosPlot (counts and intensities, filtered and unfiltered), pathNetPlot, pathPlot, and pathChainPlot; compare tumor vs. normal communication with comparePath and pathPlot.compare. The rationale is that CommPath screens pathway-associated LR interactions by considering pathway activities in receiver cells, unlike tools such as CellPhoneDB or CellChat that consider only intercellular signaling.

## Related tools

- **CommPath** (Primary R package used to create CommPath objects, identify LR markers, infer significant LR pairs, score pathways, filter functional interactions, and visualize communication networks and chains) — https://github.com/yingyonghui/CommPath
- **GSVA** (Algorithm invoked by scorePath to compute per-cell pathway activation scores (min.size = 10); UCell is an alternative)
- **R** (Runtime environment; CommPath is installed from GitHub via devtools::install_github)

## Examples

```
tumor.obj <- createCommPath(expr.mat = tumor.expr, cell.info = tumor.label, species = "hsapiens"); tumor.obj <- findLRmarker(object = tumor.obj, method = "wilcox.test"); tumor.obj <- findLRpairs(object = tumor.obj, logFC.thre = 0, p.thre = 0.01)
```

## Evaluation signals

- Verify that expected PDF plot files are produced (e.g., circosPlot.count.nonfiltered.pdf, pathNet.upstream.Endothelial.1.pdf, pathChainPlot.Endothelial.pdf)
- Confirm the CXCL12/CXCR4 pair emerges as the highest-intensity LR interaction between CD8+ T cells and their upstream endothelial cells and fibroblasts
- Check that rendered figures qualitatively match reported Fig. 2-style LR interaction patterns among cell types
- Confirm significant LR pairs are stored in tumor.obj@interact[['InteractNumer']] and [['InteractGene']] with p < 0.01 and logFC.thre = 0
- Confirm that diffAllPath returns only significantly up-regulated pathways per cluster (only.posi = TRUE, only.sig = TRUE)

## Limitations

- The GSVA algorithm is time-consuming on large datasets; UCell or average-based assessment are alternatives
- Supplementary artifacts (Supplementary Figure 7A/B/C and Supplementary Table S1) are referenced but their content is not included in the provided text
- No version numbers are stated for CommPath or any of the compared tools, and no changelog is available for the repository

## Evidence
