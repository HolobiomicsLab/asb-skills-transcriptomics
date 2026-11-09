---
name: directional-concordance-analysis
description: Use when two orthogonal per-cell label assignments (e.g., copy-number-based
  vs mutation/expression-based) exist for the same cell barcodes and you need to quantify
  how well one predicts the other, directionally.
when_to_use_negative:
- The sample lacks copy-number alterations — CNAs are rare in most AML samples, so
  the CNA-based reference classification may not be definable
- The two label sets do not share a common cell-barcode identifier space, making a
  per-cell contingency table impossible without reconciliation
- You need variant detection sensitivity, not agreement between existing per-cell
  classifications — concordance analysis does not address dropout-related sensitivity
  limits
tools:
- name: python3
  role: Loads B.tsv and _counts_UB.tsv, builds the CNA-vs-SNV contingency table, and
    computes directional concordance percentages
- name: cb_sniffer
  role: Calls mutant and reference molecular barcodes from 10x single-cell BAMs to
    derive the SNV/expression-based per-cell AML labels
  repo: https://github.com/sridnona/cb_sniffer
- name: VarTrix
  role: Alternative tool to assign previously called variants to individual cell barcodes
    as input for the SNV/expression-based label vector
  repo: https://github.com/10XGenomics/vartrix
- name: Docker (sridnona/python3:180925.v1)
  role: Containerized environment in which the python3/pysam concordance workflow
    is run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:f34bedf0a70a1bb513f4cb6c08bf3b64fe725e6b06f94b33b0867ed7510c4476
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - VarTrix
  - cb_sniffer
  - python3
  - sridnona/python3:180925.v1
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# directional-concordance-analysis

## Summary

Cross-tabulates two independent single-cell classification schemes — here CNA-based (CONICSmat k=4 clustering on 1p/7q eWGS CNA regions) and SNV/expression-signature-based AML cell calls — and computes directional concordance to benchmark one method against the other in sample 809653.

## When to use

Use when two orthogonal per-cell label assignments (e.g., copy-number-based vs mutation/expression-based) exist for the same cell barcodes and you need to quantify how well one predicts the other, directionally.

## When NOT to use

- The sample lacks copy-number alterations — CNAs are rare in most AML samples, so the CNA-based reference classification may not be definable
- The two label sets do not share a common cell-barcode identifier space, making a per-cell contingency table impossible without reconciliation
- You need variant detection sensitivity, not agreement between existing per-cell classifications — concordance analysis does not address dropout-related sensitivity limits

## Inputs

- Cell-by-gene expression matrix for sample 809653 (B.tsv)
- Chromium molecular-barcode read-count table (_counts_UB.tsv, UB_tag counts)
- eWGS-discovered CNA event regions on chromosomes 1p and 7q
- Per-cell SNV/expression-signature AML calls from cb_sniffer/VarTrix variant-to-cell assignment

## Outputs

- Cross-tabulation table of CNA-based vs SNV/expression-based AML/non-AML cell calls (cna_snv_concordance.tsv)
- Directional concordance values: CNA→SNV (95.5%) and SNV→CNA (94.9%)

## How to apply

Load the sample-809653 expression matrix (B.tsv) and the molecular-barcode read-count table (_counts_UB.tsv) in python3, then restrict to the eWGS-discovered CNA regions on chromosomes 1p and 7q and run CONICSmat with k=4 to partition cells by CNA profile; cells carrying the tumor-specific 1p/7q events are labeled AML. Independently label each cell AML/non-AML via the SNV/expression-signature approach (cells expressing tumor-specific mutations, called with tools such as cb_sniffer, are AML). Cross-tabulate the two label vectors in a python3 contingency table, then compute directional concordance twice: fraction of CNA-AML cells also called AML by SNV/expression, and fraction of SNV-AML cells also called AML by CNA. Rationale: CNA-based classification is better established, so it serves as the reference against which the SNV/expression approach is benchmarked; directional values differ because the denominators (each method's positive set) differ.

## Related tools

- **python3** (Loads B.tsv and _counts_UB.tsv, builds the CNA-vs-SNV contingency table, and computes directional concordance percentages)
- **cb_sniffer** (Calls mutant and reference molecular barcodes from 10x single-cell BAMs to derive the SNV/expression-based per-cell AML labels) — https://github.com/sridnona/cb_sniffer
- **VarTrix** (Alternative tool to assign previously called variants to individual cell barcodes as input for the SNV/expression-based label vector) — https://github.com/10XGenomics/vartrix
- **Docker (sridnona/python3:180925.v1)** (Containerized environment in which the python3/pysam concordance workflow is run)

## Examples

```
python3 -c "import pandas as pd; df=pd.read_csv('labels.tsv',sep='\t'); ct=pd.crosstab(df.cna_call,df.snv_call); ct.to_csv('cna_snv_concordance.tsv',sep='\t')"
```

## Evaluation signals

- cna_snv_concordance.tsv exists and is a valid contingency table whose row/column totals match the number of cells classified by each method
- CNA→SNV directional concordance equals 95.5% (cells classified AML by copy number also classified AML by SNV/expression)
- SNV→CNA directional concordance equals 94.9% (cells classified AML by SNV/expression also classified AML by copy number)
- The CONICSmat clustering used k=4 restricted to 1p/7q CNA regions, and every cell appears in exactly one cell of the contingency table
- The two directional percentages differ (95.5% vs 94.9%), confirming the two denominators (positive sets) were computed separately

## Limitations

- CNAs rarely reflect the complete clonal architecture of a tumor and are rare in most AML samples, so the CNA-based reference labels may not capture full subclonal complexity
- Dropout (transcript and allelic dropout) on scRNA-seq platforms reduces SNV-based classification sensitivity — dropout reduces mutation-detection sensitivity by a factor of two — so discordant cells may reflect sensitivity limits rather than true disagreement
- It is impossible to determine whether a cell is truly wild-type for a given mutation, limiting interpretation of non-AML calls in the SNV/expression arm

## Evidence

- [results] Directional concordance between CNA- and SNV-based AML cell calls: [evidence span withheld — anchor status: not_found]
- [readme] Mutation barcodes from 10x data feed the SNV-based labels: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [methods] UB_tag counts are an input to the workflow: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [discussion] CNA-based classification is not always available in AML: "Because CNAs rarely reflect the complete clonal architecture of a tumor (and are rare in most AML samples)"
