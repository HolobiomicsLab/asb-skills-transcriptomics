---
name: welch-two-sample-t-testing
description: Use when cells have been assigned intact or mutant gRNAs by GiRAFR and
  you need to test whether mutant gRNA cells fail to downregulate their target genes
  relative to intact gRNA cells.
when_to_use_negative:
- When gRNA-to-cell labels come from tools that do not detect mutant gRNAs (e.g.,
  Cell Ranger labels gRNAs with 1 Hamming distance as intact), since mislabeled cells
  attenuate the intact-vs-mutant expression contrast
- When the question is which gRNA mutation types or positions are tolerable or damaging
  — the article declares this fine-grained appraisal beyond its scope
- When the input is already a summarized table of expression statistics rather than
  per-cell count values
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GiRAFR
  role: Produces the mutation-aware gRNA-to-cell assignment labels (intact vs spacer-mutant
    vs scaffold-mutant) that define the comparison groups
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: Python
  role: Hosts scipy.stats.ttest_ind used to run the Welch two-sample t-tests and pandas/numpy
    for assembling the results table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:68d5de2773aa7bb8e17f8ca4900de1c454eed7202abda1ab2ffcbbfc3763a56b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - GiRAFR
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# welch-two-sample-t-testing

## Summary

Welch two-sample t-testing of single-cell target gene expression between gRNA classes (intact vs spacer-mutant vs scaffold-mutant) to demonstrate that mutant gRNAs are dysfunctional. It reproduces the GiRAFR paper's functional impact analysis (Figure 2b, P = 8.2e-6).

## When to use

Use when cells have been assigned intact or mutant gRNAs by GiRAFR and you need to test whether mutant gRNA cells fail to downregulate their target genes relative to intact gRNA cells.

## When NOT to use

- When gRNA-to-cell labels come from tools that do not detect mutant gRNAs (e.g., Cell Ranger labels gRNAs with 1 Hamming distance as intact), since mislabeled cells attenuate the intact-vs-mutant expression contrast
- When the question is which gRNA mutation types or positions are tolerable or damaging — the article declares this fine-grained appraisal beyond its scope
- When the input is already a summarized table of expression statistics rather than per-cell count values

## Inputs

- GiRAFR gRNA-to-cell assignment files (cells.gRNA.txt, cells.gRNA.single.txt) labeling cells as intact, spacer-mutant, or scaffold-mutant
- Single-cell transcriptome raw or normalized gene expression count matrix
- Filtered cell barcode list from the expression library mapping

## Outputs

- target_gene_expression_welch_ttest.tsv with per-target-gene Welch t-statistics, degrees of freedom, and p-values comparing intact vs spacer-mutant, intact vs scaffold-mutant, and intact vs untransduced cell groups

## How to apply

Load GiRAFR's gRNA-to-cell assignment output (cells.gRNA.txt and cells.gRNA.single.txt, produced by assign_gRNA.assign_gRNA_to_cell with min_umi default = 3 or auto two-component Gaussian mixture thresholds) to label each cell barcode as carrying an intact, spacer-mutant, or scaffold-mutant gRNA. Intersect these labels with the filtered cell barcodes and the single-cell transcriptome count matrix so both matrices share only the filtered barcodes. For each target gene, extract raw or normalized counts for the three gRNA-class groups and run scipy.stats.ttest_ind with equal_var=False (Welch's test) comparing intact-gRNA cells against each mutant class and against untransduced cells. Assemble t-statistics, degrees of freedom, and p-values into a table and confirm the direction of the effect: intact gRNAs significantly reduce target gene expression while spacer- and scaffold-mutant gRNAs do not.

## Related tools

- **GiRAFR** (Produces the mutation-aware gRNA-to-cell assignment labels (intact vs spacer-mutant vs scaffold-mutant) that define the comparison groups) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **Python** (Hosts scipy.stats.ttest_ind used to run the Welch two-sample t-tests and pandas/numpy for assembling the results table)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(intact_cells_expr, spacer_mutant_cells_expr, equal_var=False)
```

## Evaluation signals

- The intact-vs-mutant Welch test on target gene expression yields a significant p-value consistent with the reported Figure 2b statistic (P = 8.2e-6)
- Spacer-mutant and scaffold-mutant groups show NO significant downregulation of their target genes
- The direction of every t-statistic confirms intact gRNAs reduce target gene expression while mutant gRNAs do not
- Only cell barcodes shared between the gRNA assignment output and the filtered expression-library barcode list enter the test
- Output table contains t-statistic, degrees of freedom, and p-value columns for each target gene comparison

## Limitations

- Interpretation depends on gRNA assignment accuracy, which requires adequate read depths; specific defined read depths need to be attained for accurate gRNA assignment and mutation detection
- Head-to-head comparison with other studies is unfeasible because studies often use ad hoc customized pipelines and fixed thresholds
- Failure to detect mutant gRNAs inflates the estimate of untransduced cells and attenuates apparent downregulation of target genes

## Evidence

- [abstract] Mutant gRNAs are dysfunctional: "We show that mutant gRNAs are dysfunctional"
- [methods] min_umi default threshold for gRNA assignment: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [methods] gRNA-to-cell assignment step: "Step4: Assign gRNAs to cells"
- [readme] README: mutant gRNAs dysfunctional and misannotation consequences: "We show that mutant gRNAs are dysfunctional, and failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells"
- [discussion] Cell Ranger mislabels mutant gRNAs as intact: "gRNAs with a single mutation (1 Hamming distance) are labeled as intact by CellRanger but similarly show an attenuated phenotype"
