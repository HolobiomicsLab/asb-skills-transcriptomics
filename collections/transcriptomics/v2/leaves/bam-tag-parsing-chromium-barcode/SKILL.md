---
name: bam-tag-parsing-chromium-barcode
description: Use when analyzing 10x Chromium scRNA-seq alignments (BAMs with CB/UB
  tags) or _counts_UB.tsv tables and you need per-cell, per-UMI molecule counts collapsed
  by genomic position — e.g., to profile coverage decay distal to the capture site.
when_to_use_negative:
- Your goal is de novo variant calling from scRNA-seq reads rather than quantifying
  molecule/coverage distributions — noise from reverse transcription makes this a
  different problem
- Transcripts come from multi-isoform genes where molecule-to-TSS/TTS assignment is
  ambiguous — the article restricted the analysis to 20,090 single-isoform genes
- The BAM lacks CB/UB cell and molecular barcode tags from a Chromium pipeline; tag-based
  collapsing cannot proceed
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: python3
  role: Analysis language for parsing barcode tags and collapsing cell-barcode/UMI
    pairs per position
- name: pysam
  role: Python library used to read the Cell Ranger BAM and access cell/molecular
    barcode tags per read
- name: sridnona/python3:180925.v1
  role: Docker image providing the python3 + pysam environment for the counting workflow
- name: cb_sniffer
  role: Complementary tool that parses CB barcodes from Cell Ranger-aligned 10x BAMs
    for mutation genotyping
  repo: https://github.com/sridnona/cb_sniffer
- name: 10x Genomics Chromium Single Cell 5' (v1) Gene Expression
  role: Source library whose alignments and _counts_UB.tsv outputs supply the barcode-tagged
    reads to count
- name: 10x Genomics Chromium Single Cell 3' (v2) Gene Expression
  role: Comparison library processed identically to contrast coverage distal to the
    3' capture site
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:2a412800d749b4f6985ca526555219b02447fb87ef086e36e12d4755381d6542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - 10x Genomics Chromium Single Cell 3' (v2) Gene Expression
  - 10x Genomics Chromium Single Cell 5' (v1) Gene Expression
  - Docker
  - cb_sniffer
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bam-tag-parsing-chromium-barcode

## Summary

Parse BAM files and barcode-annotated count tables from 10x Genomics Chromium single-cell libraries to count reads per Chromium molecular barcode (UB_tag), collapsing to unique cell-barcode/UMI pairs per genomic position. This is the molecule-counting foundation used to quantify transcript coverage as a function of distance from transcript ends in end-biased 5' (v1) and 3' (v2) kits.

## When to use

Use when analyzing 10x Chromium scRNA-seq alignments (BAMs with CB/UB tags) or _counts_UB.tsv tables and you need per-cell, per-UMI molecule counts collapsed by genomic position — e.g., to profile coverage decay distal to the capture site.

## When NOT to use

- Your goal is de novo variant calling from scRNA-seq reads rather than quantifying molecule/coverage distributions — noise from reverse transcription makes this a different problem
- Transcripts come from multi-isoform genes where molecule-to-TSS/TTS assignment is ambiguous — the article restricted the analysis to 20,090 single-isoform genes
- The BAM lacks CB/UB cell and molecular barcode tags from a Chromium pipeline; tag-based collapsing cannot proceed

## Inputs

- BAM files aligned with the 10x Cell Ranger pipeline, carrying cell barcode (CB) and molecular barcode (UB) tags
- _counts_UB.tsv tables of per-cell read counts for the Chromium molecular barcode (UB_tag)
- Gene model annotation restricted to 20,090 single-isoform genes with strand and TSS/TTS coordinates
- Cell barcodes list (e.g., Cell Ranger filtered barcodes.tsv)

## Outputs

- Unique cell-barcode/UMI molecule counts per genomic position per gene
- Distance-binned molecule counts relative to TSS/TTS, normalized as fraction of total molecules per base, per kit (5' v1 vs 3' v2)
- Coverage-vs-distance curves (Fig. 1b style) out to at least 10 kbp from the transcript end

## How to apply

Load the per-cell UMI (_counts_UB.tsv) tables generated from the 5' (v1) and 3' (v2) Chromium libraries, parsing the Chromium molecular barcode (UB_tag) read counts produced by the Cell Ranger pipeline. Using pysam in a python3 session inside the Docker image sridnona/python3:180925.v1, collapse reads to unique cell-barcode/UMI pairs per genomic position — in the article this was done within the 20,090 single-isoform genes so each molecule maps unambiguously to one gene model. Annotate each counted molecule with its signed distance from the gene's transcription start site (TSS) or transcription termination site (TTS), orienting by gene strand, then aggregate counts in genomic-position bins relative to the transcript end and normalize to the fraction of total molecules per base for each kit. The rationale is that end-biased 10x platforms capture only partial transcript coverage, so molecule-level counting relative to TSS/TTS is required to assess how far from the capture site variant detection remains feasible.

## Related tools

- **python3** (Analysis language for parsing barcode tags and collapsing cell-barcode/UMI pairs per position)
- **pysam** (Python library used to read the Cell Ranger BAM and access cell/molecular barcode tags per read)
- **sridnona/python3:180925.v1** (Docker image providing the python3 + pysam environment for the counting workflow)
- **cb_sniffer** (Complementary tool that parses CB barcodes from Cell Ranger-aligned 10x BAMs for mutation genotyping) — https://github.com/sridnona/cb_sniffer
- **10x Genomics Chromium Single Cell 5' (v1) Gene Expression** (Source library whose alignments and _counts_UB.tsv outputs supply the barcode-tagged reads to count)
- **10x Genomics Chromium Single Cell 3' (v2) Gene Expression** (Comparison library processed identically to contrast coverage distal to the 3' capture site)

## Examples

```
python3 cb_sniffer.py aln.bam variants.tsv good_barcodes.tsv UPN508084 -mq 0 -bq 1
```

## Evaluation signals

- After collapsing, every counted molecule corresponds to a unique cell-barcode/UMI pair at a single genomic position; no duplicate UB/cell-barcode combinations remain per position
- The input _counts_UB.tsv contains read counts keyed by Chromium molecular barcode (UB_tag), and the molecules map within 20,090 single-isoform gene bodies
- Signed distances from TSS/TTS are correctly strand-oriented (5' kit distances computed from the 5' TSS, 3' kit from the TTS side)
- Both 5' (v1) and 3' (v2) kits show consistent coverage extending at least 10 kbp from the transcript end
- For the average gene with the 5' kit, at least 2.5% of unique sequenced transcripts map to any given base up to 10 kbp from the 5' transcription start site

## Limitations

- Partial transcript coverage is specific to end-biased platforms such as the Chromium platform and limits the sensitivity of variant detection; coverage drops non-linearly across transcript length, so some variants are much more easily detectable than others
- Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms, reducing detection sensitivity
- Counting was restricted to single-isoform genes (20,090 genes) to avoid ambiguous transcript-end assignment; multi-isoform genes were excluded

## Evidence

- [methods] Barcode counting produces the per-cell UMI table: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [methods] Computing environment for the counting workflow: "Docker image
----------
* sridnona/python3:180925.v1"
- [readme] cb_sniffer README describes parsing cell barcodes from aligned 10x BAMs: "Parse CB barcodes from Single cell rna seq data

Bams aligned using 10x cellranger pipeline"
- [results] Coverage finding from molecule counting relative to the 5' end: [evidence span withheld — anchor status: not_found]
