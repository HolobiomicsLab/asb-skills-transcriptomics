---
name: differential-pathway-activation-analysis
description: Use when scRNA-seq clusters and candidate LR pairs are available and
  the goal is to determine which intracellular signaling pathways are significantly
  activated per cluster and which LR interactions are functionally associated with
  them.
when_to_use_negative:
- When only intercellular LR pair expression statistics are needed without interpreting
  downstream pathway activity in receiver cells (e.g. a CellPhoneDB-style first-pass
  screen).
- When the expression matrix is raw counts not yet normalized by library-size and
  log-transformed, since CommPath expects normalized input.
- When the biological question concerns differential communication between two conditions
  (tumor vs normal) — use CommPath's comparePath workflow instead.
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CommPath
  role: R package providing findLRpath, scorePath, diffAllPath, filterLR, filterPath,
    and pathHeatmap to score pathway activation, test differential activation per
    cluster, and link activated pathways to LR interactions
  repo: https://github.com/yingyonghui/CommPath
- name: GSVA
  role: Algorithm used by scorePath (min.size = 10) to compute per-cell pathway activation
    scores; UCell is provided as a faster alternative
  repo: https://www.bioconductor.org/packages/release/bioc/html/GSVA.html
- name: R
  role: Execution environment; CommPath is installed via devtools::install_github("yingyonghui/CommPath")
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:e3e484bc66c3f4a74604e8b2ff76a2acb5c90246b78f14910c06378ebf2457bf
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
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# differential-pathway-activation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A CommPath-based procedure for scoring per-cell signaling pathway activities (GSVA/UCell) and testing which pathways are significantly activated in each cell cluster versus other user-defined clusters, then tying those activated pathways back to ligand-receptor interactions.

## When to use

Use when scRNA-seq clusters and candidate LR pairs are available and the goal is to determine which intracellular signaling pathways are significantly activated per cluster and which LR interactions are functionally associated with them.

## When NOT to use

- When only intercellular LR pair expression statistics are needed without interpreting downstream pathway activity in receiver cells (e.g. a CellPhoneDB-style first-pass screen).
- When the expression matrix is raw counts not yet normalized by library-size and log-transformed, since CommPath expects normalized input.
- When the biological question concerns differential communication between two conditions (tumor vs normal) — use CommPath's comparePath workflow instead.

## Inputs

- Library-size normalized, log-transformed gene-by-cell expression matrix (e.g. tumor.expr from HCC.tumor.3k.RData)
- Vector of cell identity labels matching cell order (e.g. tumor.label)
- CommPath object created with createCommPath(species = "hsapiens") after findLRmarker and findLRpairs
- Pathway annotation category choice (KEGG, WikiPathways, Reactome, GO)

## Outputs

- acti.path.dat dataframe of significantly up-regulated pathways per cluster with mean.diff, t/W, P.val, and P.val.adj plus ligand.in.path/receptor.in.path columns
- Filtered pathway-associated LR interactions (object@interact.filter via filterLR and filterPath)
- pathHeatmap.pdf heatmap of top differentially activated pathways per cluster

## How to apply

After finding marker ligands/receptors (findLRmarker, e.g. wilcox.test) and significant LR pairs (findLRpairs with logFC.thre = 0, p.thre = 0.01), assign marker genes to overlapping pathway genesets with findLRpath(category = "kegg") using KEGG, WikiPathways, Reactome, or GO annotations. Score per-cell pathway activation with scorePath(method = "gsva", min.size = 10) (UCell or mean-based scoring are faster alternatives). Run diffAllPath(only.posi = TRUE, only.sig = TRUE) to obtain acti.path.dat with mean.diff, t/W statistics, P.val, and P.val.adj comparing each cluster against other user-defined clusters. Screen functional LR interactions tied to activated pathways with filterLR and prune pathways whose marker ligands do not trigger any downstream pathway with filterPath; visualize the top differentially activated pathways with pathHeatmap(top.n.pathway = 10, cell.aver = TRUE).

## Related tools

- **CommPath** (R package providing findLRpath, scorePath, diffAllPath, filterLR, filterPath, and pathHeatmap to score pathway activation, test differential activation per cluster, and link activated pathways to LR interactions) — https://github.com/yingyonghui/CommPath
- **GSVA** (Algorithm used by scorePath (min.size = 10) to compute per-cell pathway activation scores; UCell is provided as a faster alternative) — https://www.bioconductor.org/packages/release/bioc/html/GSVA.html
- **R** (Execution environment; CommPath is installed via devtools::install_github("yingyonghui/CommPath")) — https://github.com/yingyonghui/CommPath

## Examples

```
tumor.obj <- findLRpath(object = tumor.obj, category = "kegg"); tumor.obj <- scorePath(object = tumor.obj, method = "gsva", min.size = 10); acti.path.dat <- diffAllPath(object = tumor.obj, only.posi = TRUE, only.sig = TRUE)
```

## Evaluation signals

- acti.path.dat contains per-pathway mean.diff, t (or W), P.val, and P.val.adj columns for each cluster, with only.posi = TRUE restricting to up-regulated pathways.
- pathHeatmap.pdf exists, opens as a valid PDF, and displays the top (top.n.pathway = 10) differentially activated pathways per cluster consistent with acti.path.dat significance calls (P.val.adj threshold applied by only.sig = TRUE).
- After filterLR, retained LR interactions map to genesets of pathways that were significantly activated in receiver clusters; filterPath removes pathways containing only marker ligands that trigger no downstream pathway.
- scorePath with method = "gsva" completes on the genesets (min.size = 10) without error, or a UCell/mean-based alternative is used on large datasets where GSVA is time-consuming.

## Limitations

- The GSVA algorithm is time-consuming on large datasets; CommPath offers UCell (PMID: 34285779) or average-based assessment as alternatives.
- Pathway genesets are limited to the curated KEGG, WikiPathways, Reactome, and GO annotations bundled with CommPath for the chosen species (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegrix/celegans).
- filterPath removes pathways whose marker ligands do not trigger any pathway in downstream clusters, so pathways lacking downstream-triggering ligands are pruned from chain inference.

## Evidence

- [methods] CommPath assigns upregulated ligands or receptors to signaling pathways by searching KEGG, WikiPathways, Reactome, and/or GO terms, then performs differential activation analysis: [evidence span withheld — anchor status: not_found]
