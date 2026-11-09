---
name: one-sided-wilcoxon-rank-sum-testing
description: Use when validating whether cells predicted as primed for a lineage (e.g.
  GP/MEMP MPPs or HS/IRS TACs) show significantly higher fate probability, TF motif
  activity, or chromatin accessibility than unprimed or opposite-primed cells.
when_to_use_negative:
- When comparing more than two ordered groups or needing effect-size estimates beyond
  rank ordering — the one-sided rank-sum test only asserts directional elevation of
  one group.
- When the lineage TF lacks a DNA-binding motif (e.g. UHRF1), TF-activity tests cannot
  be applied and only accessibility-based comparisons are valid.
- When cells cannot be uniquely assigned to one lineage (overlapping primed assignments),
  since the test requires mutually exclusive groups — exclude overlaps via intersect/setdiff
  first.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: DICE
  role: Provides FindPrimedCells and FindCandidateFactors outputs whose primed-cell
    groups are contrasted by one-sided Wilcoxon tests, and defines Wilcoxon-derived
    marker genes
  repo: https://github.com/aet21/DICE
- name: Seurat
  role: Supplies the scRNA-Seq object context for Wilcoxon rank-sum marker-gene identification
    and log-normalization of the gene accessibility matrix
- name: Palantir
  role: Provides cell-fate probabilities per lineage that are compared between primed
    and unprimed groups via one-sided Wilcoxon tests
  repo: https://github.com/dpeerlab/Palantir
- name: chromVAR
  role: Produces TF motif activity scores (e.g. GATA1, KLF1, IRF8) that are summed
    per cell and tested with one-sided Wilcoxon rank-sum tests
  repo: https://github.com/GreenleafLab/chromVAR
- name: ggpubr
  role: Renders the boxplots with geom_signif one-sided Wilcoxon P-value annotations
    on fate probability and TFA comparisons
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:91a81088b82adc3dd4713a01fc5ace9493c855ba84b221853cbd3618db6f6ac8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/aet21/DICE
  tools:
  - DICE
  - Palantir
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggpubr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# one-sided-wilcoxon-rank-sum-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply one-sided Wilcoxon rank-sum tests (alternative='greater') to compare primed versus unprimed cell groups in the DICE framework, both to derive marker genes of primed cells from scRNA-Seq data and to statistically validate priming in matched scATAC-Seq readouts (Palantir fate probability, chromVAR TF activity, gene-level accessibility).

## When to use

Use when validating whether cells predicted as primed for a lineage (e.g. GP/MEMP MPPs or HS/IRS TACs) show significantly higher fate probability, TF motif activity, or chromatin accessibility than unprimed or opposite-primed cells.

## When NOT to use

- When comparing more than two ordered groups or needing effect-size estimates beyond rank ordering — the one-sided rank-sum test only asserts directional elevation of one group.
- When the lineage TF lacks a DNA-binding motif (e.g. UHRF1), TF-activity tests cannot be applied and only accessibility-based comparisons are valid.
- When cells cannot be uniquely assigned to one lineage (overlapping primed assignments), since the test requires mutually exclusive groups — exclude overlaps via intersect/setdiff first.

## Inputs

- DICE-derived primed cell assignments (e.g. primedGP/primedMEMP MPPs from FindPrimedCells, or HS/IRS-primed TACs)
- Gene-level accessibility matrix from matched scATAC-Seq (e.g. ranzoni.gene.access.m; peaks 3kb upstream + gene body, log-normalized with Seurat)
- Precomputed chromVAR TF activity matrix (e.g. ranzoni.TFA.m)
- Palantir cell-fate probabilities toward the competing lineages
- Wilcoxon-derived marker-gene lists of primed cells (e.g. primedCellMarker.l)
- cisTopic topic scores and marker-gene-associated peak accessibility (SHARE-Seq TAC validation)

## Outputs

- One-sided Wilcoxon rank-sum P-values for each group comparison (primed vs opposite-primed, primed vs unprimed)
- Marker gene lists per primed lineage (significantly higher expression in primed cells)
- ggplot2/ggpubr boxplots with geom_signif annotations, e.g. tfa_validation_boxplots.png and tac_priming_atac_validation.png

## How to apply

First run Wilcoxon rank-sum tests on scRNA-Seq expression to obtain marker genes with significantly higher expression in primedMEMPs or primedGPs, restricting candidates to genes present in the scATAC-Seq accessibility matrix. In the Ranzoni validation workflow, intersect marker lists with rownames of ranzoni.gene.access.m, sum accessibility per cell, define ATAC-side primed cells as the top-10% (quantile 0.9) by summed marker accessibility, and exclude cells primed for both lineages via intersect/setdiff. Then apply one-sided Wilcoxon rank-sum tests with alternative='greater' comparing, e.g., primedMEMP vs primedGP and primedMEMP vs unprimed on summed chromVAR TFA of GATA1/KLF1 (MEMP) or IRF8 alone (GP, since UHRF1 has no motif activity). For SHARE-Seq TAC validation, test Palantir fate probability and Runx2/Zfhx3/Foxo3/Lmo7 TF activity of primed versus unprimed TACs the same way. Visualize group differences with ggplot2/ggpubr boxplots annotated using geom_signif, saving figures such as tfa_validation_boxplots.png or tac_priming_atac_validation.png.

## Related tools

- **DICE** (Provides FindPrimedCells and FindCandidateFactors outputs whose primed-cell groups are contrasted by one-sided Wilcoxon tests, and defines Wilcoxon-derived marker genes) — https://github.com/aet21/DICE
- **Seurat** (Supplies the scRNA-Seq object context for Wilcoxon rank-sum marker-gene identification and log-normalization of the gene accessibility matrix)
- **Palantir** (Provides cell-fate probabilities per lineage that are compared between primed and unprimed groups via one-sided Wilcoxon tests) — https://github.com/dpeerlab/Palantir
- **chromVAR** (Produces TF motif activity scores (e.g. GATA1, KLF1, IRF8) that are summed per cell and tested with one-sided Wilcoxon rank-sum tests) — https://github.com/GreenleafLab/chromVAR
- **ggpubr** (Renders the boxplots with geom_signif one-sided Wilcoxon P-value annotations on fate probability and TFA comparisons)

## Examples

```
wilcox.test(tfa.m[grep('GATA1|KLF1',rownames(tfa.m)),] |> colSums() ~ group, alternative='greater'); ggboxplot(df, x='group', y='tfa', add='jitter') + geom_signif(comparisons=list(c('primedMEMP','unprimed')), test='wilcox.test', alternative='greater')
```

## Evaluation signals

- PrimedMEMP cells show significantly higher summed GATA1/KLF1 chromVAR TFA than primedGP and unprimed cells (one-sided Wilcoxon P < 0.05, alternative='greater').
- PrimedGP cells show significantly higher IRF8 TFA than primedMEMP and unprimed cells.
- In SHARE-Seq TAC validation, primed TACs show significantly higher Palantir fate probability toward their assigned lineage and higher chromVAR TF activity of Runx2/Zfhx3 (HS) or Foxo3/Lmo7 (IRS).
- ATAC-side primed cells are reproducibly the top-10% (quantile 0.9) by summed marker-gene accessibility, with no cell assigned to both lineages after overlap exclusion.
- Figure output (tfa_validation_boxplots.png / tac_priming_atac_validation.png) renders with geom_signif P-value annotations matching the wilcox.test calls.

## Limitations

- Validation of scRNA-Seq-predicted primed states in scATAC-Seq relied on an invented, approximate method for assigning corresponding primed states (top-10% marker accessibility), so test results inherit that assignment's assumptions.
- One-sided tests presuppose a directional hypothesis (primed > comparison group); using them without a prior lineage assignment direction would be inappropriate.
- In the real datasets analyzed, covariation patterns did not deviate strongly from monotonic linear dependencies, so the downstream group comparisons may not exercise non-linear priming signals.
- UHRF1 is not a TF and has no chromVAR motif activity, limiting the GP-side validation to IRF8 alone.

## Evidence

- [other] one-sided Wilcoxon rank-sum tests (alternative='greater') on fate probability and TF activity: [evidence span withheld — anchor status: not_found]
- [other] For the MEMP lineage, compute the summed chromVAR TFA of GATA1 and KLF1 per cell; for the GP lineage, use the TFA of IRF8 alone (UHRF1 is not a TF and has no motif activity): [evidence span withheld — anchor status: not_found]
- [other] Build ggplot2 boxplots with ggpubr geom_signif one-sided Wilcoxon rank-sum tests (alternative='greater') comparing primedMEMP vs primedGP and primedMEMP vs unprimed: [evidence span withheld — anchor status: not_found]
- [other] the figure must show significantly higher summed GATA1/KLF1 TFA in primedMEMP cells (one-sided Wilcoxon P < 0.05 vs primedGP and vs unprimed): [evidence span withheld — anchor status: not_found]
