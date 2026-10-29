---
name: odds-ratio-heatmap-visualization
description: Use when comparing a ligand-receptor resource's membership in biological
  annotation terms (pathways, cancer gene sets, tissue-enriched proteins) against
  a pooled background to expose enrichment bias.
when_to_use_negative:
- When the goal is to score method predictions on expression data rather than characterize
  resource content bias — this skill tests resource composition against annotations,
  not CCC inference accuracy.
- When the annotation set of interest is not defined over the resource's genes/interactions
  (no in/out-of-annotation membership can be tabulated).
- When fewer than two resources are available, since the pooled 16-resource background
  and cross-resource bias comparison are the point of the analysis.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LIANA
  role: Assembles and loads the ligand-receptor interactions, receivers, and transmitters
    from the 16 CCC resources that supply the contingency-table counts and pooled
    background
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: R client used to access OmniPath and other interaction/annotation resources
    feeding the enrichment analysis
  repo: https://github.com/saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:6aa88e1de4d726163dff5f7dfc9a8faed37300fb3a93da7bd91381efd81c8a78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - OmnipathR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# odds-ratio-heatmap-visualization

## Summary

Reproduces the Fig 4 analysis of the LIANA CCC-resource comparison: per resource and annotation term (SignaLink/NetPath pathways, CancerSEA, Human Protein Atlas tissue-enriched proteins, DisGeNet), a 2x2 Fisher's exact test against the pooled 16-resource background yields odds ratios that are BH-corrected and rendered as a significance-marked odds-ratio heatmap.

## When to use

Use when comparing a ligand-receptor resource's membership in biological annotation terms (pathways, cancer gene sets, tissue-enriched proteins) against a pooled background to expose enrichment bias.

## When NOT to use

- When the goal is to score method predictions on expression data rather than characterize resource content bias — this skill tests resource composition against annotations, not CCC inference accuracy.
- When the annotation set of interest is not defined over the resource's genes/interactions (no in/out-of-annotation membership can be tabulated).
- When fewer than two resources are available, since the pooled 16-resource background and cross-resource bias comparison are the point of the analysis.

## Inputs

- Ligand-receptor interaction lists (transmitters/receivers) from the 16 CCC resources assembled in LIANA (e.g. Ramilowski, ConnectomeDB, CellPhoneDB, CellChatDB, LRdb, OmniPath, Cellinker, CellTalkDB, iTALK, SingleCellSignalR, NATMI)
- Annotation term gene sets from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet
- Pooled collection of all 16 resources used as the Fisher's-test background

## Outputs

- Per resource-annotation pair Fisher's exact test p-values with Benjamini-Hochberg FDR correction
- Matrix of odds ratios (rows = 16 resources, columns = annotation terms)
- Odds-ratio heatmap figure (figure4_odds_ratio_heatmap.png) with FDR-significant cells marked and under/overrepresentation calls (e.g. T cell receptor pathway)

## How to apply

Load the ligand-receptor interactions, receivers, and transmitters from each of the 16 CCC resources assembled in LIANA, plus annotation sets from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet, and pool all 16 resources into a background collection. For each resource-annotation pair, build a 2x2 contingency table (in-annotation vs out-of-annotation for the resource vs the pooled background) and run Fisher's exact test per resource in R. Correct the per-resource p-values across annotation terms with Benjamini-Hochberg FDR correction, then compute odds ratios for each pair and assemble them into a matrix. Plot the odds-ratio heatmap as in Fig 4, marking cells significant after FDR correction (e.g. the T cell receptor pathway underrepresentation in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK versus overrepresentation in OmniPath and Cellinker), and save as figure4_odds_ratio_heatmap.png.

## Related tools

- **LIANA** (Assembles and loads the ligand-receptor interactions, receivers, and transmitters from the 16 CCC resources that supply the contingency-table counts and pooled background) — https://github.com/saezlab/liana
- **OmnipathR** (R client used to access OmniPath and other interaction/annotation resources feeding the enrichment analysis) — https://github.com/saezlab/OmnipathR

## Examples

```
R> tab <- matrix(c(n_rr, n_rn, n_br, n_bn), nrow = 2, dimnames = list(res = c("in", "out"), bg = c("in", "out"))); or <- (n_rr * n_bn) / (n_rn * n_br); p <- fisher.test(tab)$p.value
```

## Evaluation signals

- Each 2x2 contingency table's row sums must equal the resource's interaction count and the pooled background size (in-annotation + out-of-annotation).
- Odds ratio invariant: OR = (a*d)/(b*c); cells with OR = 1 indicate no bias relative to the pooled collection.
- BH-adjusted q-values must be computed across annotation terms within each resource, and only q < 0.05 cells marked significant on the heatmap.
- Heatmap dimensions should match 16 resources x annotation terms from SignaLink, NetPath, CancerSEA, Human Protein Atlas, and DisGeNet.
- Direction check: T cell receptor pathway should appear underrepresented (OR < 1) in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK, and overrepresented (OR > 1) in OmniPath and Cellinker.

## Limitations

- Enrichment calls reflect curation bias in the resources, not experimental truth; the paper notes 'the variable overlap between the resources, their uneven functional distributions, and the reported curation disagreements are a call for further large-scale curation efforts.'
- Odds ratios depend on the pooled 16-resource background definition; changing the resource pool changes the reference and can flip under/overrepresentation calls.
- Annotation coverage is itself uneven (e.g. restricted coverage of cytokine activity signatures and receptor proteins), so absent annotations should not be read as absent biology.

## Evidence

- [methods] resource vs background contingency design: [evidence span withheld — anchor status: not_found]
- [abstract] bias finding: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [intro] motivation for term-bias analysis: "First, we explored the degree of overlap among resources and whether certain resources are biased toward specific biological terms, such as pathways and tissue-enriched proteins."
- [readme] resource assembly via LIANA: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [discussion] curation caveat: "the variable overlap between the resources, their uneven functional distributions, and the reported curation disagreements are a call for further large-scale curation efforts."
