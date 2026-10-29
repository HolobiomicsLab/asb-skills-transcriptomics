---
name: differential-gene-expression-testing
description: Use when pseudo-bulked gene-level count matrices from 3'-tag scRNA-seq
  (scUTRquant output) are available and you need to identify genes whose mRNA abundance
  differs between two cell types or conditions.
when_to_use_negative:
- The goal is testing differential 3'UTR isoform usage/length rather than mRNA abundance
  — use scUTRboot WUI tests (|dWUI| > 0.10) instead
- Data are full-length or non-3'-tag scRNA-seq that cannot be processed by the scUTRquant/kallisto-bustools
  pipeline
- Only a single aggregated count vector is available without replicate cells per condition,
  so no t-test can be performed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scUTRquant
  role: Produces the gene-level and 3'UTR isoform count SingleCellExperiment objects
    that serve as input for the differential expression tests
  repo: https://github.com/Mayrlab/scUTRquant
- name: scUTRboot
  role: Statistical testing package used in parallel to call differential UTR usage,
    whose output gene sets are compared against the DGE results
  repo: https://github.com/mfansler/scutrboot
- name: kallisto-bustools
  role: Underlying quantification workflow (pseudoalignment with kallisto bus, barcode
    correction with bustools) that generates the raw count matrices upstream
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:42488f8a5733c645b8b195561a02549414d6613789f866917d8a4fef40051c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# differential-gene-expression-testing

## Summary

Calls differentially expressed genes between cell types by Welch t-testing pseudo-aggregated gene-level counts from scUTRquant SingleCellExperiment objects, retaining hits with |fold change| > 1.5 and q < 0.05, so that DGE gene sets can be compared against 3'UTR-length (DUL) changes.

## When to use

Use when pseudo-bulked gene-level count matrices from 3'-tag scRNA-seq (scUTRquant output) are available and you need to identify genes whose mRNA abundance differs between two cell types or conditions.

## When NOT to use

- The goal is testing differential 3'UTR isoform usage/length rather than mRNA abundance — use scUTRboot WUI tests (|dWUI| > 0.10) instead
- Data are full-length or non-3'-tag scRNA-seq that cannot be processed by the scUTRquant/kallisto-bustools pipeline
- Only a single aggregated count vector is available without replicate cells per condition, so no t-test can be performed

## Inputs

- scUTRquant SingleCellExperiment objects with sparse gene-level count matrices
- Cell metadata (colData) with cell-type annotations
- Cell-type pair definitions specifying the two groups to compare

## Outputs

- Table of differentially expressed genes per cell-type pair with fold change, p-value, and multiple-testing-corrected q-value
- DGE gene sets (|FC| > 1.5, q < 0.05) formatted for downstream overlap analysis with DUL sets
- Overlap fractions and chi-square independence test results (e.g. dge_dul_overlap_fractions.tsv)

## How to apply

Load the scUTRquant-generated SingleCellExperiment objects and extract the sparse gene-level count matrix plus cell metadata from colData. Aggregate counts per cell type, restrict to genes co-expressed in both members of the comparison (for the DGE-vs-DUL analysis, multi-UTR genes shared across the pair), and run per-gene Welch t-tests on gene-level counts. Retain differentially expressed genes with |fold change| > 1.5 and q < 0.05 after multiple-testing correction. The rationale is to obtain a gene-level expression-change set that is directly comparable, at matched significance thresholds, to the differentially-used-UTR (DUL) gene sets produced by scUTRboot WUI tests (|dWUI| > 0.10, q < 0.05); overlaps between the two sets are then assessed with chi-square tests of independence.

## Related tools

- **scUTRquant** (Produces the gene-level and 3'UTR isoform count SingleCellExperiment objects that serve as input for the differential expression tests) — https://github.com/Mayrlab/scUTRquant
- **scUTRboot** (Statistical testing package used in parallel to call differential UTR usage, whose output gene sets are compared against the DGE results) — https://github.com/mfansler/scutrboot
- **kallisto-bustools** (Underlying quantification workflow (pseudoalignment with kallisto bus, barcode correction with bustools) that generates the raw count matrices upstream)

## Examples

```
t.test(counts_a, counts_b, var.equal = FALSE)  # Welch t-test per gene between aggregated cell-type count vectors
```

## Evaluation signals

- Each reported overlap fraction between DGE and DUL gene sets falls within the expected 0-7.4% range across the 17 cell-type pairs
- Chi-square tests of independence indicate statistical independence of DGE and DUL sets in 14/17 sample pairs
- Called DGE hits all satisfy |fold change| > 1.5 and q < 0.05 after multiple-testing correction
- Count matrices are restricted to genes co-expressed (multi-UTR genes) in both members of each cell-type pair before testing

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so 3'UTR isoform resolution depends on the curated CS annotations rather than de novo mapping
- DGE and DUL changes affect mostly different genes, so a DGE call list alone cannot substitute for 3'UTR-length analysis or vice versa
- Fewer than 20% of 3'UTR changes regulate mRNA or protein abundance, so DGE hits are an incomplete proxy for functional output differences

## Evidence

- [abstract] Extensive 3'UTR length changes across cell types are as widespread and coordinately regulated as gene expression changes but affect mostly different genes: "we discover extensive 3'UTR length changes across cell types that are as widespread and coordinately regulated as gene expression changes but affect mostly different genes"
- [intro] Expression and UTR-length changes occur in largely non-overlapping gene groups: "observed that changes in gene expression and in 3′UTR length occurred in different groups of genes, indicating that they largely represent independent regulatory events"
- [discussion] scUTRboot identifies significant 3'UTR differences for comparison with expression: "Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis"
- [other] Workflow defines the concrete thresholds used for the DGE calls: [evidence span withheld — anchor status: not_found]
