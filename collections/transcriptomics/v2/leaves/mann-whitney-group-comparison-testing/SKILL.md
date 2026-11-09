---
name: mann-whitney-group-comparison-testing
description: Use when a continuous per-cell score such as ORIGINS2 PPIN activity must
  be compared between two cell-type-labeled groups (e.g., basal vs L2) with a non-parametric
  two-sided test, or when testing whether a significance call depends on a network-construction
  choice.
when_to_use_negative:
- The expression matrix is raw counts rather than the normalized expression matrix
  that ORIGINS2's activity() expects (rows = genes, columns = cells).
- The comparison involves more than two cell populations or a paired/repeated-measures
  design — this skill is a two-sample two-sided test of basal vs L2-style groups.
- 'The goal is comparing activity magnitudes across PPINs of different sizes: this
  workflow uses normalization=FALSE (raw edge sums), whereas normalization=TRUE would
  divide by the number of PPIN edges.'
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: ORIGINS2
  role: Provides activity() to compute per-cell PPIN activity (normalization=FALSE
    returns the unnormalized sum of all PPIN edges) and build_ppin() to construct
    the proliferation PPIN from genes_bp and full_human_ppin
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R (stats::wilcox.test)
  role: Runs the two-sided Mann-Whitney U test on basal vs L2 activity distributions
    for each network variant
- name: dplyr
  role: Data manipulation for grouping per-cell activity values by cell-type labels
    (basal vs L2) and handling the negative-regulator-filtered edge copy
- name: magrittr
  role: Pipe operator used to chain the R grouping/testing workflow steps (ORIGINS2
    Imports)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:6dc88c5dee29f23c8314d43b98ecef625d8c8919586aab78adef89c3220dcda5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielasenraoka/ORIGINS2
  repo_url: https://github.com/danielasenraoka/ORIGINS2
  tools:
  - ORIGINS2
  - R
  - R (stats::wilcox.test)
  - dplyr
  - magrittr
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mann-whitney-group-comparison-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Two-sided Mann-Whitney U testing (stats::wilcox.test in R) over per-cell PPIN activity scores computed with ORIGINS2's activity() function. Used to decide whether annotated cell populations, e.g., basal vs L2 cells in the healthy breast sample, differ at the 0.01 significance level in activity for a process such as cell proliferation (GO:0008283), and whether that significance depends on retaining negative-regulator edges in the PPIN.

## When to use

Use when a continuous per-cell score such as ORIGINS2 PPIN activity must be compared between two cell-type-labeled groups (e.g., basal vs L2) with a non-parametric two-sided test, or when testing whether a significance call depends on a network-construction choice.

## When NOT to use

- The expression matrix is raw counts rather than the normalized expression matrix that ORIGINS2's activity() expects (rows = genes, columns = cells).
- The comparison involves more than two cell populations or a paired/repeated-measures design — this skill is a two-sample two-sided test of basal vs L2-style groups.
- The goal is comparing activity magnitudes across PPINs of different sizes: this workflow uses normalization=FALSE (raw edge sums), whereas normalization=TRUE would divide by the number of PPIN edges.

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells), e.g., the healthy breast sample matrix
- Per-cell cell-type labels partitioning cells into two groups (basal, L2)
- PPIN adjacency edges for the biological process of interest (e.g., proliferation_edges, or build_ppin output from genes_bp + full_human_ppin)
- Negative-regulator edge list/criterion used to construct the negative-regulator-excluded network variant

## Outputs

- Per-cell activity vectors (unnormalized sums of PPIN edge contributions) for each network variant (negative regulators included vs excluded)
- Two-sided Mann-Whitney U (stats::wilcox.test) p-values for basal vs L2 per variant
- proliferation_activity_mannwhitney.csv containing per-cell activity values for both variants plus the basal-vs-L2 p-values

## How to apply

Compute per-cell activity with ORIGINS2 as activity(expression_matrix, proliferation_edges, FALSE): the FALSE flag makes the output the unnormalized sum of all PPIN edge contributions per cell, and the network is either the packaged proliferation_edges adjacency or one built with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin). Run activity() twice — once on the complete network retaining negative-regulator edges and once on a filtered copy with negative-regulator edges removed — to obtain two per-cell activity vectors. Partition each vector into basal and L2 groups using the sample's cell-type labels (dplyr/magrittr support the grouping), then run stats::wilcox.test(basal, l2, alternative = 'two.sided') per variant. Judge significance at the 0.01 level: in the healthy breast sample the basal-vs-L2 proliferation-activity difference is significant only when negative regulators are included, which the source card calls an 'overestimated calculation'. Write the per-cell activity values for both variants plus the two basal-vs-L2 p-values to proliferation_activity_mannwhitney.csv, and confirm the two p-values differ to demonstrate the significance call depends on negative-regulator handling.

## Related tools

- **ORIGINS2** (Provides activity() to compute per-cell PPIN activity (normalization=FALSE returns the unnormalized sum of all PPIN edges) and build_ppin() to construct the proliferation PPIN from genes_bp and full_human_ppin) — https://github.com/danielasenraoka/ORIGINS2
- **R (stats::wilcox.test)** (Runs the two-sided Mann-Whitney U test on basal vs L2 activity distributions for each network variant)
- **dplyr** (Data manipulation for grouping per-cell activity values by cell-type labels (basal vs L2) and handling the negative-regulator-filtered edge copy)
- **magrittr** (Pipe operator used to chain the R grouping/testing workflow steps (ORIGINS2 Imports))

## Examples

```
library(ORIGINS2); act <- activity(expression_matrix, proliferation_edges, FALSE); wilcox.test(act[ctype=='basal'], act[ctype=='L2'], alternative='two.sided')
```

## Evaluation signals

- The Mann-Whitney p-value obtained with negative regulators included differs from the p-value obtained with negative regulators excluded (the workflow's stated validation criterion).
- In the healthy breast sample, the basal-vs-L2 proliferation-activity comparison is significant at the 0.01 level only for the negative-regulator-included ('overestimated') variant.
- Each activity vector has one value per cell (one per column of the expression matrix), since normalization=FALSE returns the unnormalized sum of PPIN edge contributions per cell.
- proliferation_activity_mannwhitney.csv exists and contains per-cell activity values for both network variants plus the two basal-vs-L2 p-values.
- wilcox.test is called two-sided on correctly label-partitioned basal and L2 subsets, not on shuffled or unassigned cells.

## Limitations

- With normalization=FALSE the activity is the raw sum of PPIN edge contributions, so values are not directly comparable across PPINs of different sizes (normalization=TRUE divides by the number of edges of the PPIN).
- The significance conclusion is specific to the healthy breast sample and depends on negative-regulator handling; including negative regulators is described as an 'overestimated calculation'.
- Requires pre-annotated cell-type labels (basal, L2) to partition cells; the test is non-parametric and two-sample only.

## Evidence

- [other] Run two-sided Mann-Whitney U tests (stats::wilcox.test in R) on basal vs L2 activity distributions for each network variant.: [evidence span withheld — anchor status: not_found]
- [other] significantly different when negative regulators are included in the PPIN (overestimated calculation), but not when negative regulators are excluded, at the 0.01 significance level.: [evidence span withheld — anchor status: not_found]
