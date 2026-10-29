---
name: snv-expression-signature-cell-calling
description: Use when single-cell RNA-seq data from a tumor sample needs per-cell
  tumor/normal labels and copy-number-based calling is unavailable or unreliable (e.g.,
  normal-karyotype AML lacking CNAs), or when validating CNA-based calls against SNV/expression
  evidence.
when_to_use_negative:
- When the goal is to determine whether a cell is truly wild-type for a mutation —
  dropout makes true wild-type status indeterminable from scRNA-seq.
- When the tumor has rich copy-number aberrations and a CNA-based classifier alone
  is sufficient; CNAs are rare in AML but this skill's value is greatest where CNAs
  are absent or incomplete.
- When the variant is germline or not somatic/within a CNV event — VarTrix assignment
  is only useful for somatic variants or variants within CNV events.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: python3
  role: Runs cb_sniffer barcode calling, count tables, and the CNA-vs-SNV cross-tabulation
- name: pysam
  role: Parses the Cell Ranger BAM to extract CB/UB barcode-tagged reads for mutation
    calling
- name: cb_sniffer
  role: Calls mutant and reference molecular barcodes per variant from 10x single-cell
    BAMs
  repo: https://github.com/sridnona/cb_sniffer
- name: 10XGenomics/vartrix
  role: Alternative tool for assigning pre-called variants to single cells, producing
    a sparse genome matrix
  repo: https://github.com/10XGenomics/vartrix
- name: Docker (sridnona/python3:180925.v1)
  role: Execution environment for python3/pysam analysis steps
- name: chrisamiller/copyCat
  role: Detects somatic copy-number aberrations used for the CNA-based comparator
    classification
  repo: https://github.com/chrisamiller/copyCat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:cb5b4ab77257ec15a7bfa036d4d15ad04b263ee50ab081dd8ad0e6fb0d8092d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - 10XGenomics/vartrix
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - cb_sniffer
  - chrisamiller/copyCat
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# snv-expression-signature-cell-calling

## Summary

Classifies individual cells as tumor (AML) or normal by combining detection of tumor-specific expressed SNVs in scRNA-seq data with an expression-signature-based label, benchmarked against CNA-based (CONICSmat) classification. Grounded in the Petti et al. 2019 AML single-cell genotyping approach.

## When to use

Use when single-cell RNA-seq data from a tumor sample needs per-cell tumor/normal labels and copy-number-based calling is unavailable or unreliable (e.g., normal-karyotype AML lacking CNAs), or when validating CNA-based calls against SNV/expression evidence.

## When NOT to use

- When the goal is to determine whether a cell is truly wild-type for a mutation — dropout makes true wild-type status indeterminable from scRNA-seq.
- When the tumor has rich copy-number aberrations and a CNA-based classifier alone is sufficient; CNAs are rare in AML but this skill's value is greatest where CNAs are absent or incomplete.
- When the variant is germline or not somatic/within a CNV event — VarTrix assignment is only useful for somatic variants or variants within CNV events.

## Inputs

- Cell Ranger-aligned 10x Chromium Single Cell 5' BAM with CB/UB tags
- eWGS-discovered somatic variant list (chrm, start, stop, ref, var, gene_name, trv_type)
- Cell barcodes file from Cell Ranger (barcodes.tsv / list of good barcodes)
- Cell-by-gene expression matrix (B.tsv)
- Molecular-barcode read-count table (_counts_UB.tsv)
- Reference genome FASTA for VarTrix

## Outputs

- Per-cell AML/non-AML labels from SNV/expression evidence
- Molecular barcode genotype calls (_counts_UB.tsv; mutant/ref barcodes per variant)
- Contingency table of CNA-based vs. SNV/expression-based cell calls (cna_snv_concordance.tsv)
- Directional concordance values (e.g., 95.5% CNA→SNV, 94.9% SNV→CNA in sample 809653)

## How to apply

First, discover somatic mutations using enhanced whole-genome sequencing (eWGS) of paired tumor/normal samples, then restrict detection to those variants in the scRNA-seq data. Call mutant vs. reference molecular barcodes (CB/UB tags) from the Cell Ranger-aligned BAM using cb_sniffer (python3, with -f read, -mq MAPQ, -bq BASEQ filters) or VarTrix, and count reads per Chromium molecular barcode to produce _counts_UB.tsv. Assign each cell a tumor label when it expresses tumor-specific mutations, and independently compute an expression-signature-based AML/non-AML label over the full cell-by-gene matrix (e.g., B.tsv). To benchmark against the copy-number route, partition cells with CONICSmat (k=4) restricted to eWGS-discovered CNA event regions (e.g., 1p and 7q), cross-tabulate the CNA-based and SNV/expression-based labels, and compute directional concordance; in case 809653 this yielded 95.5% (CNA→SNV) and 94.9% (SNV→CNA). Analyze genotyped cells in the context of the entire sample to leverage expression information from non-genotyped cells.

## Related tools

- **python3** (Runs cb_sniffer barcode calling, count tables, and the CNA-vs-SNV cross-tabulation)
- **pysam** (Parses the Cell Ranger BAM to extract CB/UB barcode-tagged reads for mutation calling)
- **cb_sniffer** (Calls mutant and reference molecular barcodes per variant from 10x single-cell BAMs) — https://github.com/sridnona/cb_sniffer
- **10XGenomics/vartrix** (Alternative tool for assigning pre-called variants to single cells, producing a sparse genome matrix) — https://github.com/10XGenomics/vartrix
- **Docker (sridnona/python3:180925.v1)** (Execution environment for python3/pysam analysis steps)
- **chrisamiller/copyCat** (Detects somatic copy-number aberrations used for the CNA-based comparator classification) — https://github.com/chrisamiller/copyCat

## Examples

```
python3 cb_sniffer.py bam_file variants.tsv barcodes.tsv 809653 -f 0 -mq 0 -bq 1
```

## Evaluation signals

- Cross-tabulation table exists as cna_snv_concordance.tsv with CNA-based and SNV/expression-based labels per cell.
- Directional concordance matches 95.5% (CNA→SNV) and 94.9% (SNV→CNA) in benchmark sample 809653.
- cb_sniffer output contains mutant and ref barcodes for each eWGS-discovered variant, filtered by read count (-f), MAPQ (-mq), and base quality (-bq).
- _counts_UB.tsv is produced with per-barcode UB_tag read counts.
- CONICSmat clustering with k=4 on 1p/7q event-restricted matrix separates cells carrying the tumor-specific copy-number events from those without.

## Limitations

- Dropout (transcript and allelic) occurs with most scRNA-seq platforms and reduces the sensitivity of mutation detection by a factor of two.
- It is impossible to determine whether a cell is truly wild-type for a given mutation from scRNA-seq data.
- Partial transcript coverage on end-biased platforms such as Chromium limits variant-detection sensitivity, and coverage drops non-linearly across transcript length so some variants are more detectable than others.
- The utility depends on the mutational composition of the sample and will likely perform better for tumor types with higher mutation burdens than AML.

## Evidence

- [intro] eWGS then scRNA-seq detection: "we first use “enhanced” whole-genome sequencing (eWGS) of paired tumor/normal samples to discover somatic mutations, and then focus on detecting those mutations in the scRNA-seq data"
- [methods] UB barcode counts output: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [readme] cb_sniffer role: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [readme] VarTrix scope: [evidence span withheld — anchor status: external_doc]
- [intro] CNA rarity in AML: "CNAs are rare in some tumor types, such as AML22,23. Moreover, CNAs rarely capture the complete subclonal complexity of any tumor"
- [discussion] Dropout limitation: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
- [discussion] Use non-genotyped cells: "we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells"
