---
name: cell-barcode-whitelist-matching
description: Use when single-cell amplicon FASTQ data contain a fixed cell barcode+UMI
  pattern in R1 (e.g. 16bp barcode + 10bp UMI) and a known cell barcode whitelist
  is available that extracted barcodes must be matched against.
when_to_use_negative:
- Input data is population-level (bulk) barcoding rather than single-cell, where the
  whitelist-based cell barcode extraction step does not apply
- No trusted cell barcode whitelist exists for the sample, so matching would incorrectly
  discard valid cell barcodes
- The barcode and UMI structure differs from the 16bp+10bp R1 pattern assumed by the
  extract pattern
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BARtab
  role: Nextflow pipeline whose single-cell mode orchestrates whitelist-supplied cell
    barcode extraction, --whitelist_indir parameter
  repo: https://github.com/DaneVass/BARtab
- name: umi-tools
  role: whitelist/extract to match cell barcodes against the whitelist and pull out
    barcode+UMI with pattern CCCCCCCCCCCCCCCCNNNNNNNNNNNN; count_tab for UMI error
    correction
  repo: https://github.com/CGATOxford/UMI-tools
- name: fastp
  role: quality filtering of raw reads before extraction (minqual 20, pctqual 80)
  repo: https://github.com/OpenGene/fastp
- name: FastQC
  role: QC of FASTQ files prior to filtering
- name: cutadapt
  role: trims constant regions after barcode extraction (--constants all for randomly
    fragmented reference-based reads)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:3bc5fa021272ab01f6da12170e650c1b1a1ab18aefc75ccbbeb4f63532ffca27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FastQC
  - R
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  - starcode
  - umi-tools
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-barcode-whitelist-matching

## Summary

Extracts and error-filters single-cell lineage barcode reads by matching cell barcodes and UMIs from R1 of paired-end amplicon FASTQ against a supplied per-sample cell barcode whitelist using umi-tools whitelist/extract, as practiced in the BARtab single-cell workflow.

## When to use

Use when single-cell amplicon FASTQ data contain a fixed cell barcode+UMI pattern in R1 (e.g. 16bp barcode + 10bp UMI) and a known cell barcode whitelist is available that extracted barcodes must be matched against.

## When NOT to use

- Input data is population-level (bulk) barcoding rather than single-cell, where the whitelist-based cell barcode extraction step does not apply
- No trusted cell barcode whitelist exists for the sample, so matching would incorrectly discard valid cell barcodes
- The barcode and UMI structure differs from the 16bp+10bp R1 pattern assumed by the extract pattern

## Inputs

- Paired-end single-cell amplicon sequencing FASTQ files (cell barcode + UMI in R1)
- Per-sample cell barcode whitelist TSV files (<sample_id>_whitelist.tsv, '-1' suffixes removed)
- Barcode structure pattern (CCCCCCCCCCCCCCNNNNNNNNNNNN: 16bp cell barcode, 10bp UMI)

## Outputs

- Whitelist-filtered reads with cell barcode and UMI extracted into read headers
- Cell barcode annotation table produced by PARSE_BARCODES_SC (umi_fraction_filter 0.3)
- UMI counts per lineage barcode per cell after error correction (umi_dist 1, umi_count_filter 5)

## How to apply

Run the BARtab Nextflow pipeline in single-cell mode with --input_type fastq and supply GEO-derived cell barcode whitelists via --whitelist_indir as <sample_id>_whitelist.tsv files with '-1' suffixes removed. Extract the 16bp cell barcode and 10bp UMI from R1 using umi-tools whitelist/extract with the pattern CCCCCCCCCCCCCCCCNNNNNNNNNNNN, after QC with FastQC and quality filtering with fastp (minqual 20, pctqual 80). Whitelist matching discards cell barcodes not present in the reference list, preventing noisy/erroneous barcode sequences from entering downstream lineage barcode counting; the retained barcode-UMI combinations then proceed to cutadapt trimming and bowtie1 alignment against the SPLINTR barcode reference (alnmismatches 2, -a --best --strata -m1 --norc).

## Related tools

- **BARtab** (Nextflow pipeline whose single-cell mode orchestrates whitelist-supplied cell barcode extraction, --whitelist_indir parameter) — https://github.com/DaneVass/BARtab
- **umi-tools** (whitelist/extract to match cell barcodes against the whitelist and pull out barcode+UMI with pattern CCCCCCCCCCCCCCCCNNNNNNNNNNNN; count_tab for UMI error correction) — https://github.com/CGATOxford/UMI-tools
- **fastp** (quality filtering of raw reads before extraction (minqual 20, pctqual 80)) — https://github.com/OpenGene/fastp
- **FastQC** (QC of FASTQ files prior to filtering)
- **cutadapt** (trims constant regions after barcode extraction (--constants all for randomly fragmented reference-based reads))

## Examples

```
nextflow run DaneVass/bartab --mode single-cell --input_type fastq --whitelist_indir <whitelist_dir> --input 'FM0-2_*_R{1,2}.fastq.gz'
```

## Evaluation signals

- Whitelist files are named <sample_id>_whitelist.tsv with '-1' suffixes removed and are accepted via --whitelist_indir without parse errors
- Only cell barcodes present in the supplied whitelist survive extraction; novel barcode sequences are excluded
- After downstream counting, per-cell barcode annotations overlap FateMap annotations at ~99.6% (12 BARtab-only, 145 FateMap-only, 39,884 shared cells)
- Pearson clone-size correlations between pipelines fall between 0.965 and 0.998
- UMI counting uses umi_dist 1 with umi_count_filter 5, so barcodes supported by fewer than 5 UMIs are dropped

## Limitations

- Benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Parameter values (e.g. FateMap UMI threshold of 15 vs BARtab umi_count_filter 5) differ between pipelines, so cross-tool comparisons require explicit threshold harmonization
- Existing tools lack flexibility for different barcode designs and lengths, so the fixed 16bp+10bp pattern must match the actual library design

## Evidence

- [other] Extract cell barcodes and UMIs from R1 with umi-tools whitelist/extract using the pattern CCCCCCCCCCCCCCCCNNNNNNNNNNNN: [evidence span withheld — anchor status: not_found]
- [other] supplied via --whitelist_indir as <sample_id>_whitelist.tsv: [evidence span withheld — anchor status: not_found]
- [readme] fastp quality filter: "Reads are quality filtered using [fastp](https://github.com/OpenGene/fastp)."
- [readme] FastQC QC step: "QC of fastq files is performed using [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)."
- [methods] filterable umi count parameter: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] PCR chimerism removal: "Reads from PCR chimerism are removed by only retaining the cell barcode-umi-lineage barcode combination supported by the most reads."
