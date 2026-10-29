---
name: bam-tag-parsing-cb-ub
description: Use when somatic SNVs from eWGS must be detected per-cell in 10x 5' scRNA-seq
  BAMs by pileup over reads carrying valid CB and UB_tag values, producing per-cell
  mutant counts.
when_to_use_negative:
- The goal is de novo variant calling from scRNA-seq reads — the approach targets
  previously discovered somatic SNVs (e.g. from eWGS) because calling variants in
  RNA-seq data is prone to reverse-transcription noise.
- BAMs lack the CB/UB_tag tag conventions of Cell Ranger output, since the pileup
  depends on parsing these per-read barcode and UMI tags.
- Multi-allelic or indel-centric assignment is required via an off-the-shelf tool
  — note VarTrix instead ignores all multi-allelic sites and its README distinguishes
  its own Smith-Waterman-based workflow.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: python3
  role: Runtime language for the barcode-aware pileup script
- name: pysam
  role: Parses BAM pileups and reads CB and UB_tag per-read tags at variant positions
- name: Docker
  role: Container image sridnona/python3:180925.v1 packages python3 and pysam for
    reproducible execution
- name: sridnona/python3:180925.v1
  role: Specific Docker image providing the python3/pysam environment
- name: cb_sniffer
  role: Mutation barcode caller whose CB-parsing logic consumes the BAM tags to call
    mutant and ref barcodes
  repo: https://github.com/sridnona/cb_sniffer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:7ec8a3d3acc17f0e1fcf745c031055599cc00f36982e2529123a3970c9c65702
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - cb_sniffer
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bam-tag-parsing-cb-ub

## Summary

Parse 10x Genomics Cell Ranger BAM files via pysam by reading the cell barcode (CB) and Chromium molecular barcode/UMI (UB_tag) tags, then collapse reads per cell/UMI for barcode-aware variant pileups used by cb_sniffer-style cell genotyping.

## When to use

Use when somatic SNVs from eWGS must be detected per-cell in 10x 5' scRNA-seq BAMs by pileup over reads carrying valid CB and UB_tag values, producing per-cell mutant counts.

## When NOT to use

- The goal is de novo variant calling from scRNA-seq reads — the approach targets previously discovered somatic SNVs (e.g. from eWGS) because calling variants in RNA-seq data is prone to reverse-transcription noise.
- BAMs lack the CB/UB_tag tag conventions of Cell Ranger output, since the pileup depends on parsing these per-read barcode and UMI tags.
- Multi-allelic or indel-centric assignment is required via an off-the-shelf tool — note VarTrix instead ignores all multi-allelic sites and its README distinguishes its own Smith-Waterman-based workflow.

## Inputs

- 10x 5' scRNA-seq BAM files aligned by the 10x cellranger pipeline (with .bai index)
- Variant file with header: chrm, start, stop, ref, var, gene_name, trv_type (somatic SNVs from eWGS)
- List of good barcodes file (Cell Ranger cell barcodes)

## Outputs

- _counts_UB.tsv table of per-cell, per-variant mutant-UMI read counts for the UB_tag molecular barcode
- Per-barcode mutant/ref genotype calls (cb_sniffer output prefixed by sample/UPN)

## How to apply

Launch the containerized environment sridnona/python3:180925.v1, which provides python3 and pysam, then index each 10x 5' scRNA-seq BAM from the five AML samples and load the eWGS-discovered somatic SNV list. For each variant position, run a pysam pileup restricted to reads whose CB cell barcode and UB_tag Chromium molecular barcode follow the 10x bam_tags conventions, optionally filtering by MAPQ and base quality (cb_sniffer defaults: --mapq 0, --baseq 1, --filter 0 reads per barcode). Collapse reads by UMI within each cell barcode and classify each cell/variant pair as mutant when at least one mutant-supporting read/UMI is present; without mutant evidence a cell is labeled unknown, since dropout makes it impossible to call a cell truly wild-type. Write the per-cell per-variant mutant-UMI read counts to _counts_UB.tsv, which feeds downstream subclone identification and genotype-to-expression integration.

## Related tools

- **python3** (Runtime language for the barcode-aware pileup script)
- **pysam** (Parses BAM pileups and reads CB and UB_tag per-read tags at variant positions)
- **Docker** (Container image sridnona/python3:180925.v1 packages python3 and pysam for reproducible execution)
- **sridnona/python3:180925.v1** (Specific Docker image providing the python3/pysam environment)
- **cb_sniffer** (Mutation barcode caller whose CB-parsing logic consumes the BAM tags to call mutant and ref barcodes) — https://github.com/sridnona/cb_sniffer

## Examples

```
python3 cb_sniffer.py sample.bam variants.tsv good_barcodes.txt UPN1 -f 0 -mq 0 -bq 1
```

## Evaluation signals

- Per-sample counts of mutant-labeled cells fall within the reported 'hundreds to thousands' range and match Table 1 values (669, 604, 8200, 2003, 354) across the five AML cases.
- Average mutant cells across variants is ~49 per variant and ~3732 mutant cells per sample (~22% of total cells), ranging from 396 to 8200.
- Output file _counts_UB.tsv is produced with per-cell per-variant UB_tag read counts as documented in the methods.
- Every read contributing to the pileup carries a valid CB barcode and UB_tag consistent with 10x bam_tags conventions.
- Each cell/variant pair resolves to exactly one class (mutant if ≥1 mutant-supporting UMI, otherwise unknown), reflecting the dropout-driven impossibility of wild-type calls.

## Limitations

- Dropout (transcript and allelic) occurs with most scRNA-seq platforms and reduces sensitivity of mutation detection by a factor of two, so cells without mutant reads cannot be declared wild-type.
- Partial transcript coverage on the end-biased Chromium platform, dropping non-linearly across transcript length, means some variants are much more easily detectable than others.
- cb_sniffer defaults are permissive (filter=0 reads per barcode, mapq=0, baseq=1); without stricter -f/-mq/-bq thresholds, low-quality evidence can contribute to calls.

## Evidence

- [methods] Pysam pileup over CB/UB_tag reads: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [methods] Docker environment with python3 and pysam: "Docker image
----------
* sridnona/python3:180925.v1"
- [readme] cb_sniffer parses CB barcodes: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data   
Bams aligned using 10x cellranger pipeline"
- [readme] cb_sniffer CLI with mapq/baseq/filter options: "usage: cb_sniffer.py [-h] [-f FILTER] [-mq MAPQ] [-bq BASEQ]
                     bam_file variant_file barcodes upn"
- [abstract] Hundreds to thousands of mutant cells per AML sample: "Applying this approach to five cryopreserved AML samples, we identify hundreds to thousands of cells containing tumor-specific mutations in each case"
