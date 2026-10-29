---
name: cna-based-aml-cell-classification
description: Use when a scRNA-seq sample has known eWGS-discovered CNA events (e.g.
  1p/7q losses in AML) and you need per-cell tumor/normal labels from copy number,
  or want to validate SNV-based cell calls against a CNA-based ground truth.
when_to_use_negative:
- The tumor sample has no detectable copy-number alterations — the article notes 'CNAs
  are rare in some tumor types, such as AML', so CNA-based classification cannot serve
  as a benchmark in CNA-silent cases like normal-karyotype AML.
- No eWGS-discovered CNA event regions are available to restrict the expression matrix,
  so clustering would be driven by expression rather than tumor-specific copy number.
- The goal is de novo CNA calling from bulk sequencing rather than per-cell classification
  — use copyCat on paired tumor/normal BAMs instead.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: python3
  role: Loads B.tsv and _counts_UB.tsv, runs CONICSmat clustering, and builds the
    CNA-vs-SNV contingency table
- name: pysam
  role: Reads BAM alignment data underlying barcode/mutation calls used in the parallel
    SNV/expression classification
- name: sridnona/cb_sniffer
  role: Calls mutant and ref Chromium molecular barcodes (UB_tag counts in _counts_UB.tsv)
    supporting the SNV/expression-based labels being benchmarked
  repo: https://github.com/sridnona/cb_sniffer
- name: 10XGenomics/vartrix
  role: Extracts single-cell variant information from 10x data for the somatic SNVs
    used in the comparison classification
  repo: https://github.com/10XGenomics/vartrix
- name: chrisamiller/copyCat
  role: Bulk somatic CNA detection from paired tumor/normal coverage used to identify
    the CNA events (1p/7q) that define the AML cells
  repo: https://github.com/chrisamiller/copyCat
- name: Docker (sridnona/python3:180925.v1)
  role: Containerized python3 environment in which the CONICSmat clustering and concordance
    computation are run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:d2dd29e810b05d55e1c1b985a507486e18060651b66681591e9ae3af73eef68a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - 10XGenomics/vartrix
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - chrisamiller/copyCat
  - pysam
  - python3
  - sridnona/cb_sniffer
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cna-based-aml-cell-classification

## Summary

Classify individual AML cells as tumor vs. normal using single-cell copy-number alteration (CNA) profiles inferred from scRNA-seq expression data with CONICSmat, then benchmark those calls against an independent SNV/expression-based classification of the same cells.

## When to use

Use when a scRNA-seq sample has known eWGS-discovered CNA events (e.g. 1p/7q losses in AML) and you need per-cell tumor/normal labels from copy number, or want to validate SNV-based cell calls against a CNA-based ground truth.

## When NOT to use

- The tumor sample has no detectable copy-number alterations — the article notes 'CNAs are rare in some tumor types, such as AML', so CNA-based classification cannot serve as a benchmark in CNA-silent cases like normal-karyotype AML.
- No eWGS-discovered CNA event regions are available to restrict the expression matrix, so clustering would be driven by expression rather than tumor-specific copy number.
- The goal is de novo CNA calling from bulk sequencing rather than per-cell classification — use copyCat on paired tumor/normal BAMs instead.

## Inputs

- Cell-by-gene expression matrix (B.tsv) for the sample (e.g. 809653)
- Molecular-barcode read-count table (_counts_UB.tsv, UB_tag counts from cb_sniffer)
- eWGS-discovered somatic CNA event coordinates (chromosomal regions such as 1p and 7q)
- Per-cell SNV/expression-signature AML labels (cells expressing tumor-specific mutations)

## Outputs

- Per-cell CNA-cluster assignments from CONICSmat (k=4)
- Per-cell CNA-based AML/non-AML labels
- Contingency table cross-tabulating CNA-based vs. SNV/expression-based cell calls (cna_snv_concordance.tsv)
- Directional concordance values (CNA→SNV and SNV→CNA percentages)

## How to apply

Load the cell-by-gene expression matrix (B.tsv) and the Chromium molecular-barcode read-count table (_counts_UB.tsv) with python3. Restrict the matrix to the eWGS-discovered CNA event regions (here chromosomes 1p and 7q) so clustering is driven by the tumor-specific copy-number events rather than baseline expression. Run CONICSmat clustering with k=4 to partition cells by CNA profile, and label cells carrying the tumor-specific 1p/7q events as AML. Independently assign per-cell labels from the SNV/expression-signature approach (cells expressing tumor-specific mutations via cb_sniffer/VarTrix barcodes), then cross-tabulate CNA-based vs. SNV/expression-based calls in a python3 contingency table. Directional concordance is the validation metric: in sample 809653, 95.5% of CNA-AML cells were also SNV-AML and 94.9% of SNV-AML cells were also CNA-AML, so agreement in the mid-90s range indicates the two orthogonal classifications are consistent.

## Related tools

- **python3** (Loads B.tsv and _counts_UB.tsv, runs CONICSmat clustering, and builds the CNA-vs-SNV contingency table)
- **pysam** (Reads BAM alignment data underlying barcode/mutation calls used in the parallel SNV/expression classification)
- **sridnona/cb_sniffer** (Calls mutant and ref Chromium molecular barcodes (UB_tag counts in _counts_UB.tsv) supporting the SNV/expression-based labels being benchmarked) — https://github.com/sridnona/cb_sniffer
- **10XGenomics/vartrix** (Extracts single-cell variant information from 10x data for the somatic SNVs used in the comparison classification) — https://github.com/10XGenomics/vartrix
- **chrisamiller/copyCat** (Bulk somatic CNA detection from paired tumor/normal coverage used to identify the CNA events (1p/7q) that define the AML cells) — https://github.com/chrisamiller/copyCat
- **Docker (sridnona/python3:180925.v1)** (Containerized python3 environment in which the CONICSmat clustering and concordance computation are run)

## Examples

```
python3 -c "import pandas as pd; B=pd.read_csv('B.tsv',sep='\t',index_col=0).loc[cna_regions]; ct=pd.crosstab(cna_labels,snv_labels); ct.to_csv('cna_snv_concordance.tsv',sep='\t')"
```

## Evaluation signals

- The cross-tabulation output file cna_snv_concordance.tsv exists and contains a 2x2 contingency table of CNA-based vs. SNV/expression-based cell calls.
- Directional concordance CNA→SNV reaches 95.5% (fraction of CNA-AML cells also called AML by SNV/expression).
- Directional concordance SNV→CNA reaches 94.9% (fraction of SNV-AML cells also called AML by copy number).
- CONICSmat clustering uses k=4 on the expression matrix restricted to the eWGS CNA event regions (chromosomes 1p and 7q), not the full transcriptome.
- Cells labeled AML by CNA are those carrying the tumor-specific 1p/7q copy-number events, and AML cells are a minority/substantial fraction consistent with the paper's identification of 'hundreds to thousands of cells containing tumor-specific mutations' per sample.

## Limitations

- CNAs are rare in some tumor types such as AML, so a CNA-based classification is not available or informative for many AML samples (e.g. normal-karyotype cases).
- CNAs rarely capture the complete subclonal complexity of any tumor, so CNA-based labels may miss subclonal structure that SNV-based labels resolve.
- Copy-number-inferred clustering reflects only the loci used (1p/7q here); concordance values are directional and sample-specific (95.5% and 94.9% in sample 809653).

## Evidence

- [results] In sample 809653, CNA-based (CONICSmat) and SNV/expression-based AML cell classifications were highly concordant: [evidence span withheld — anchor status: not_found]
- [results] 95.5% of cells classified as AML by copy number were also classified as AML by SNV and expression signature: "95.5% of cells classified as AML by copy number were also classified as AML by SNV and expression signature"
- [intro] CNAs are rare in some tumor types, such as AML22,23. Moreover, CNAs rarely capture the complete subclonal complexity of any tumor: "CNAs are rare in some tumor types, such as AML22,23. Moreover, CNAs rarely capture the complete subclonal complexity of any tumor"
- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [readme] Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
