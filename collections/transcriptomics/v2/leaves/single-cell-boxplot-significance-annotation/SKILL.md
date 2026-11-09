---
name: single-cell-boxplot-significance-annotation
description: Use when you have cell-level group assignments (e.g., DICE-derived primed
  vs unprimed TACs) plus continuous per-cell scores such as Palantir fate probabilities
  or chromVAR TF activity, and need boxplots with annotated one-sided Wilcoxon significance.
when_to_use_negative:
- Input data lacks cell-level group labels (e.g., only bulk or cluster-aggregated
  summary statistics), so a per-cell rank-sum comparison and per-group boxplot cannot
  be constructed
- The hypothesis is two-sided or multi-directional — the paper's design uses one-sided
  Wilcoxon tests (alternative='greater') tied to a specific priming direction per
  lineage
- Scores were not produced at matched cell resolution (e.g., chromVAR activities from
  a different cell population than the RNA-derived primed assignments), breaking the
  primed/unprimed comparison
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ggpubr
  role: Draws the group-wise boxplots and adds geom_signif significance annotations
    carrying the one-sided wilcox.test P-values
- name: ggplot2
  role: Underlying plotting grammar for the fate-probability and summed-TF-activity
    boxplots saved as tac_priming_atac_validation.png
- name: R
  role: Implementation language in which the Wilcoxon rank-sum tests (wilcox.test,
    alternative='greater') and plotting are run
- name: Palantir
  role: Supplies the per-cell fate probabilities toward HS and IRS lineages that are
    compared by the boxplots
  repo: dpeerlab/Palantir
- name: chromVAR
  role: Supplies the per-cell TF activity scores (Runx2, Zfhx3, Foxo3, Lmo7) from
    scATAC-Seq peaks and motifs that are summed and compared per group
  repo: GreenleafLab/chromVAR
- name: DICE
  role: Provides the primed vs unprimed cell assignments (FindPrimedCells) that define
    the groups compared in the boxplots
  repo: github.com/aet21/DICE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:6fd4121f98fb56f9fc3224603ca6c20ccb1bb81a77d3c9c04d757779bce09c3c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/dpeerlab/Palantir
  tools:
  - DICE
  - Palantir
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggpubr
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-boxplot-significance-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A visualization-and-inference skill for comparing per-cell scores (e.g., Palantir cell-fate probabilities, chromVAR TF activities) between primed and unprimed single-cell groups using ggplot2/ggpubr boxplots annotated with one-sided Wilcoxon rank-sum P-values via geom_signif, as used in the DICE paper to validate hair-follicle primed TAC states in SHARE-Seq scATAC-Seq data.

## When to use

Use when you have cell-level group assignments (e.g., DICE-derived primed vs unprimed TACs) plus continuous per-cell scores such as Palantir fate probabilities or chromVAR TF activity, and need boxplots with annotated one-sided Wilcoxon significance.

## When NOT to use

- Input data lacks cell-level group labels (e.g., only bulk or cluster-aggregated summary statistics), so a per-cell rank-sum comparison and per-group boxplot cannot be constructed
- The hypothesis is two-sided or multi-directional — the paper's design uses one-sided Wilcoxon tests (alternative='greater') tied to a specific priming direction per lineage
- Scores were not produced at matched cell resolution (e.g., chromVAR activities from a different cell population than the RNA-derived primed assignments), breaking the primed/unprimed comparison

## Inputs

- Cell-level group assignments for primed vs unprimed populations (e.g., DICE FindPrimedCells output for HS- and IRS-primed TACs, with overlapping assignments removed)
- Palantir cell-fate probabilities toward each competing lineage (HS and IRS) per cell
- chromVAR TF activity scores for Runx2, Zfhx3, Foxo3 and Lmo7 computed from peak-level scATAC-Seq data and binding motifs
- Optional supporting scores: cisTopic topic scores and gene-level accessibility aggregated over marker-gene-associated peaks (3kb upstream + gene body, log-normalized with Seurat)

## Outputs

- ggplot2/ggpubr boxplot figure comparing score distributions per group with geom_signif one-sided Wilcoxon P-value annotations (saved as tac_priming_atac_validation.png)
- One-sided Wilcoxon rank-sum P-values (alternative='greater') for each primed-vs-unprimed comparison

## How to apply

Assign cells to groups (e.g., DICE-derived HS-primed vs IRS-primed vs unprimed TACs, excluding overlapping primed cells), then for each group extract the continuous score to test — Palantir fate probability toward the HS or IRS lineage, or summed chromVAR TF activity for Runx2/Zfhx3 (HS) and Foxo3/Lmo7 (IRS) computed from peak-level scATAC-Seq and motif matches. Run one-sided Wilcoxon rank-sum tests with alternative='greater' comparing each primed group against unprimed TACs, so the alternative direction matches the biological hypothesis (primed cells higher toward their assigned lineage). Draw ggplot2/ggpubr boxplots per group (e.g., fate probability and summed TF activity), overlay geom_signif significance annotations carrying the wilcox.test P-values, and save the figure (in the paper's SHARE-Seq case, as tac_priming_atac_validation.png). Validation succeeds when primed TACs show significantly higher TF activity/accessibility and fate probability toward their assigned lineage; as a symmetry check, HS-primed cells should not show elevated IRS scores and vice versa.

## Related tools

- **ggpubr** (Draws the group-wise boxplots and adds geom_signif significance annotations carrying the one-sided wilcox.test P-values)
- **ggplot2** (Underlying plotting grammar for the fate-probability and summed-TF-activity boxplots saved as tac_priming_atac_validation.png)
- **R** (Implementation language in which the Wilcoxon rank-sum tests (wilcox.test, alternative='greater') and plotting are run)
- **Palantir** (Supplies the per-cell fate probabilities toward HS and IRS lineages that are compared by the boxplots) — dpeerlab/Palantir
- **chromVAR** (Supplies the per-cell TF activity scores (Runx2, Zfhx3, Foxo3, Lmo7) from scATAC-Seq peaks and motifs that are summed and compared per group) — GreenleafLab/chromVAR
- **DICE** (Provides the primed vs unprimed cell assignments (FindPrimedCells) that define the groups compared in the boxplots) — github.com/aet21/DICE

## Examples

```
ggpubr::ggboxplot(tac, x="group", y="fate_prob", color="group") + ggpubr::stat_compare_means(method="wilcox.test", alternative="greater")
```

## Evaluation signals

- One-sided Wilcoxon P-values (alternative='greater') for primed vs unprimed comparisons reach significance, matching the paper's wilcox.test annotations on the figure
- Direction check: the primed group's median fate probability and summed TF activity toward its OWN assigned lineage (HS-primed -> HS, IRS-primed -> IRS) exceed the unprimed group's; the cross-lineage comparison should not show this elevation
- Figure file is produced with the expected name (tac_priming_atac_validation.png) and contains geom_signif annotations whose P-values equal the independently recomputed wilcox.test outputs
- Overlapping primed cells (assigned to both HS- and IRS-primed sets) were excluded before testing, mirroring the paper's intersect-based exclusion step
- Boxplots contain one box per group with the number of cells in each group consistent with the DICE assignment counts

## Limitations

- Validation of scRNA-derived primed states in scATAC-Seq relied on an invented, approximate method for assigning corresponding primed states in the scATAC data, so group labels feeding the boxplots are not ground truth
- The one-sided Wilcoxon test only tests the pre-specified direction; it cannot detect scores that are significantly LOWER in the primed group or non-monotonic group differences
- In the paper's real datasets, DICE-identified regulatory factors (e.g., Runx2, Zfhx3, Foxo3, Lmo7) were not experimentally perturbed — significance annotations support computational validation only

## Evidence

- [other] Draw ggplot2/ggpubr boxplots of fate probability and summed TF activity per group with geom_signif significance annotations: [evidence span withheld — anchor status: not_found]
- [other] one-sided Wilcoxon on fate probability and TF activity: [evidence span withheld — anchor status: not_found]
- [other] chromVAR TF activity scores for the four regulatory factors: [evidence span withheld — anchor status: not_found]
- [readme] chromVAR role in scoring motif accessibility variability: [evidence span withheld — anchor status: external_doc]
