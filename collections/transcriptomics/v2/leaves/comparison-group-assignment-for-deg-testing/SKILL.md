---
name: comparison-group-assignment-for-deg-testing
description: Use when raw bulk RNA-seq count matrices plus sample metadata from drug-treated
  myeloma cell lines (17-AAG single-agent or 17-AAG + Ixazomib combination) must be
  partitioned into sensitive-vs-resistant groups before DESeq2/edgeR differential-expression
  testing.
when_to_use_negative:
- When only single-cell RNA-seq data are available — this skill concerns bulk RNA-seq
  contrasts, not the scRNAseq target-enrichment screening step
- When the goal is drug-combination prediction itself — use the greedy set cover secDrug
  pipeline rather than DE group assignment
- When count matrices are already normalized and contrast group labels are externally
  supplied — AssignGroup partitioning would be redundant
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: secDrug
  role: R package providing the AssignGroup(x, l) helper and the CombinationDrugMyeloma
    dataset used to define sensitive-vs-resistant comparison groups
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: R
  role: Environment in which grouping, DESeq2/edgeR model fitting, and DEG summary
    export are executed
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Packaged dataset object `d` supplying the bulk RNA-seq count matrices and
    sample metadata to be partitioned
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: DESeq2
  role: Negative-binomial differential-expression model fitted on counts after comparison
    groups are assigned
- name: edgeR
  role: Alternative negative-binomial DE model used for the same per-group contrasts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:041a8ec47b60188d9ac80d12e519e7c713ff825e2d6db8fc08630cda9ed58c1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - DESeq2
  - R
  - edgeR
  - secDrug
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# comparison-group-assignment-for-deg-testing

## Summary

A skill for defining sensitive-vs-resistant comparison groups in the HMCL bulk RNA-seq experiment (untreated, 0.5 µM 17-AAG, 17-AAG + 15 nM Ixazomib at 24 h) using the secDrug AssignGroup(x, l) helper, so that DESeq2/edgeR negative-binomial differential-expression contrasts are tested against the correct sample partitions.

## When to use

Use when raw bulk RNA-seq count matrices plus sample metadata from drug-treated myeloma cell lines (17-AAG single-agent or 17-AAG + Ixazomib combination) must be partitioned into sensitive-vs-resistant groups before DESeq2/edgeR differential-expression testing.

## When NOT to use

- When only single-cell RNA-seq data are available — this skill concerns bulk RNA-seq contrasts, not the scRNAseq target-enrichment screening step
- When the goal is drug-combination prediction itself — use the greedy set cover secDrug pipeline rather than DE group assignment
- When count matrices are already normalized and contrast group labels are externally supplied — AssignGroup partitioning would be redundant

## Inputs

- Raw bulk RNA-seq count matrices for HMCLs (untreated baseline, 0.5 µM 17-AAG, 17-AAG + 15 nM Ixazomib at 24 h)
- Sample metadata / grouping labels loaded via the `d` (CombinationDrugMyeloma) dataset object
- Grouping logic supplied to the AssignGroup(x, l) helper (sensitive vs resistant labels)

## Outputs

- Per-contrast DEG counts (deg_summary_table.csv) with p < 0.05 significance calls
- Ranked gene lists (ranked_gene_lists.csv) feeding GSA and limma regulator analysis
- Subset of significant genes with |fold-change| > 2 and FDR < 0.05

## How to apply

Load the deposited bulk RNA-seq count matrices and sample metadata from the secDrug repository via the packaged dataset object `d` (CombinationDrugMyeloma), covering untreated baseline, 0.5 µM 17-AAG single-agent, and 17-AAG + 15 nM Ixazomib combination at 24 h. Apply the AssignGroup(x, l) helper to assign each HMCL to its comparison group (17-AAG-sensitive vs 17-AAG-resistant) before any modeling, since DE contrasts are defined per group pairing. Normalize raw counts and fit negative-binomial differential-expression models in R with DESeq2 and edgeR for each contrast: 17-AAG-sensitive vs -resistant lines, 17-AAG response, and combination response. Report genes significant at p < 0.05, and flag the biologically actionable subset with |fold-change| > 2 and FDR < 0.05. Write per-contrast DEG counts to deg_summary_table.csv and ranked gene lists to ranked_gene_lists.csv for downstream GSA/limma regulator ranking.

## Related tools

- **secDrug** (R package providing the AssignGroup(x, l) helper and the CombinationDrugMyeloma dataset used to define sensitive-vs-resistant comparison groups) — github.com/Ujjal-Mukherjee/secDrug
- **R** (Environment in which grouping, DESeq2/edgeR model fitting, and DEG summary export are executed) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Packaged dataset object `d` supplying the bulk RNA-seq count matrices and sample metadata to be partitioned) — github.com/Ujjal-Mukherjee/secDrug
- **DESeq2** (Negative-binomial differential-expression model fitted on counts after comparison groups are assigned)
- **edgeR** (Alternative negative-binomial DE model used for the same per-group contrasts)

## Examples

```
groups <- AssignGroup(x, l)  # assign HMCLs to sensitive/resistant groups from the CombinationDrugMyeloma dataset object `d`
```

## Evaluation signals

- DEG summary reproduces the reported counts: 421 DE genes for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, and 3974 for the combination
- Within the 3974 combination DE genes (p < 0.05), 853 genes show |fold-change| > 2 with FDR < 0.05
- Top regulators CEBPB, ERBB2, CCND1, and let-7 appear in the ranked outputs derived from the group-defined contrasts
- deg_summary_table.csv contains one row per contrast (sensitive vs resistant, 17-AAG response, combination response) with non-empty DEG counts
- Every sample in the count matrix is assigned to exactly one group by AssignGroup with no unlabeled samples entering the DE models

## Limitations

- The grouping and downstream contrasts are limited to two-drug combinations; the article notes model systems for testing more than two secDrugs are not yet developed
- DEG counts depend on the exact AssignGroup partitioning of the HMCL panel; alternative grouping definitions will change the 421/1449/3974 reported counts
- Supplementary Methods, Tables, and Figures referenced for full parameter details are not included/verifiable in the provided text

## Evidence

- [other] using the packaged dataset object `d` (CombinationDrugMyeloma) and the AssignGroup(x, l) helper to define sensitive-vs-resistant comparison groups: [evidence span withheld — anchor status: not_found]
- [other] A total of 3974 genes changed significantly between untreated and 17-AAG + Ixazomib combination-treated samples (p < 0.05), of which 853 showed |fold-change| > 2 with FDR < 0.05.: [evidence span withheld — anchor status: not_found]
- [other] Normalize raw counts and fit negative-binomial differential-expression models in R with DESeq2 and edgeR: [evidence span withheld — anchor status: not_found]
- [intro] we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance: "we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance to show that the top predicted secondary secDrugs"
