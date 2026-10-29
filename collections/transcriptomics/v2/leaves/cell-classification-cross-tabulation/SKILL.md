---
name: cell-classification-cross-tabulation
description: Use when two cell-level AML/tumor classification labels exist for the
  same barcodes (e.g., CNA-based CONICSmat clusters vs. SNV/expression-based genotype
  calls) and you must quantify how well one predicts the other.
when_to_use_negative:
- The tumor sample has no somatic copy-number alterations (e.g., normal-karyotype
  AML) and cannot be partitioned by CNA-based clustering, so only the SNV/expression
  route applies.
- You need per-cell calls in cells lacking matched tumor/normal eWGS to pre-discover
  the SNVs or CNA regions used to define the two labels.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Runs the CONICSmat clustering, the SNV/expression labeling, and the contingency-table
    cross-tabulation and concordance computation
- name: pysam
  role: Python BAM access used within the python3 workflow for barcode/read-level
    operations
- name: Docker (sridnona/python3:180925.v1)
  role: Pinned execution environment reproducing the article's python3/pysam analysis
- name: cb_sniffer
  role: Calls mutant and reference molecular barcodes from 10x single-cell data, providing
    the SNV-based per-cell genotype evidence used as one classification arm
  repo: https://github.com/sridnona/cb_sniffer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:61f29cd6c88618dd82a69f46df1cf72d70b80e22d67d3d33c2dc32efbab496bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - cb_sniffer
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cell-classification-cross-tabulation

## Summary

Cross-tabulates two independent single-cell tumor-classification schemes — a CNA-based clustering (CONICSmat, k=4 on eWGS-discovered 1p/7q CNA regions) and an SNV/expression-signature-based assignment — and computes directional concordance between them, as benchmarked in AML sample 809653.

## When to use

Use when two cell-level AML/tumor classification labels exist for the same barcodes (e.g., CNA-based CONICSmat clusters vs. SNV/expression-based genotype calls) and you must quantify how well one predicts the other.

## When NOT to use

- The tumor sample has no somatic copy-number alterations (e.g., normal-karyotype AML) and cannot be partitioned by CNA-based clustering, so only the SNV/expression route applies.
- You need per-cell calls in cells lacking matched tumor/normal eWGS to pre-discover the SNVs or CNA regions used to define the two labels.

## Inputs

- Per-cell expression matrix for sample 809653 (B.tsv)
- Chromium molecular-barcode read-count table (_counts_UB.tsv)
- eWGS-discovered somatic CNA event coordinates on chromosomes 1p and 7q
- Tumor-specific SNV list with per-cell variant expression calls (e.g., from cb_sniffer)

## Outputs

- Per-cell AML/non-AML labels from CNA-based CONICSmat clustering (k=4)
- Per-cell AML/non-AML labels from the SNV/expression-signature approach
- 2x2 contingency table (cna_snv_concordance.tsv)
- Directional concordance percentages (CNA→SNV and SNV→CNA)

## How to apply

Load the per-cell expression matrix (B.tsv) and the Chromium molecular-barcode read-count table (_counts_UB.tsv) with python3 in the sridnona/python3:180925.v1 Docker image (using pysam as needed for BAM-level work). Restrict the matrix to the eWGS-discovered CNA event regions on chromosomes 1p and 7q, then run CONICSmat with k=4 to partition cells by copy-number profile; cells carrying the tumor-specific 1p/7q events are labeled AML. Independently label each cell AML/non-AML via the SNV/expression-signature approach (cells expressing tumor-specific mutations). Build a python3 contingency table cross-tabulating CNA-based vs. SNV/expression-based labels, then compute directional concordance in both directions. For sample 809653 the validated result is 95.5% (CNA→SNV) and 94.9% (SNV→CNA), so agreement between the two orthogonal strategies is the expected outcome when both calls are correct.

## Related tools

- **python3** (Runs the CONICSmat clustering, the SNV/expression labeling, and the contingency-table cross-tabulation and concordance computation)
- **pysam** (Python BAM access used within the python3 workflow for barcode/read-level operations)
- **Docker (sridnona/python3:180925.v1)** (Pinned execution environment reproducing the article's python3/pysam analysis)
- **cb_sniffer** (Calls mutant and reference molecular barcodes from 10x single-cell data, providing the SNV-based per-cell genotype evidence used as one classification arm) — https://github.com/sridnona/cb_sniffer

## Examples

```
python3 -c "import pandas as pd; ct=pd.crosstab(df['cna_label'],df['snv_label']); ct.to_csv('cna_snv_concordance.tsv',sep='\t')"
```

## Evaluation signals

- The cross-tabulation output file cna_snv_concordance.tsv exists and is a 2x2 contingency table of CNA-based vs. SNV/expression-based labels.
- Directional concordance CNA→SNV equals 95.5% for sample 809653.
- Directional concordance SNV→CNA equals 94.9% for sample 809653.
- CONICSmat clustering used k=4 on the 1p/7q CNA-restricted expression matrix, not the full genome-wide matrix.
- Every cell barcode in the contingency table carries both a CNA-based and an SNV/expression-based label (no unmatched barcodes).

## Limitations

- CNAs are rare in tumor types such as AML and rarely capture the complete subclonal complexity of any tumor, so the CNA-based arm may be unavailable or incomplete for many samples.
- Transcript and allelic dropout occur with most scRNA-seq platforms, reduce the sensitivity of mutation detection by a factor of two, and make it impossible to determine whether a cell is truly wild-type for a given mutation.
- Partial transcript coverage on end-biased platforms such as Chromium drops non-linearly across transcript length, so some variants are much more easily detectable than others, biasing the SNV-based arm.

## Evidence

- [abstract] abstract: "use the results to distinguish AML cells (including normal-karyotype AML cells) from normal cells"
- [intro] intro: "CNAs are rare in some tumor types, such as AML22,23. Moreover, CNAs rarely capture the complete subclonal complexity of any tumor"
- [methods] methods: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [readme] readme: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [discussion] discussion: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
