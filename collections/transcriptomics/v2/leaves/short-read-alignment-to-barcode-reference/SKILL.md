---
name: short-read-alignment-to-barcode-reference
description: Use when barcode-bearing reads have been extracted and trimmed and must
  be assigned to a known synthetic barcode reference library (e.g. SPLINTR barcodes
  in FASTA) to produce per-barcode alignment counts.
when_to_use_negative:
- No barcode reference library exists for the barcode design — use the reference-free
  mode instead of bowtie1 alignment.
- 'Input reads are raw, untrimmed FASTQ: constant regions must be removed with cutadapt
  first or alignments will not span the full barcode and FILTER_ALIGNMENTS will reject
  them.'
- Downstream analysis needs clone/cell annotation rather than bulk barcode counts
  — continue to UMI error correction and PARSE_BARCODES_SC instead of stopping at
  the counts table.
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: bowtie1
  role: Indexes the barcode reference FASTA and aligns trimmed reads with -a --best
    --strata -m1 --norc and alnmismatches = 2
  repo: http://bowtie-bio.sourceforge.net/index.shtml
- name: cutadapt
  role: Trims flanking constant regions (e.g. upconstant TGACCATGTACGATTGACTA, downconstant
    TGCTAATGCGTACTGACTAG) so reads span the barcode for alignment
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: samtools
  role: Sorts the alignment SAM and counts alignments per barcode (indexstats / COUNT_BARCODES_SAM)
    into counts tables
- name: BARtab
  role: Nextflow pipeline orchestrating index building, alignment (BUILD_BOWTIE_INDEX,
    FILTER_ALIGNMENTS) and barcode counting
  repo: https://github.com/DaneVass/BARtab/
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:c530033d45d00ac4c6c536ae8ddfe85630e5f7bcec73cbdd2be070c50e9823b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BARtab
  - FLASh
  - FastQC
  - MultiQC
  - R
  - bartools
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  - starcode
  - umi-tools
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# short-read-alignment-to-barcode-reference

## Summary

Aligns quality-filtered, constant-region-trimmed sequencing reads against a known synthetic barcode reference library (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta) using bowtie1 within the BARtab pipeline, then filters alignments so each read is assigned to a single lineage barcode for downstream quantification.

## When to use

Use when barcode-bearing reads have been extracted and trimmed and must be assigned to a known synthetic barcode reference library (e.g. SPLINTR barcodes in FASTA) to produce per-barcode alignment counts.

## When NOT to use

- No barcode reference library exists for the barcode design — use the reference-free mode instead of bowtie1 alignment.
- Input reads are raw, untrimmed FASTQ: constant regions must be removed with cutadapt first or alignments will not span the full barcode and FILTER_ALIGNMENTS will reject them.
- Downstream analysis needs clone/cell annotation rather than bulk barcode counts — continue to UMI error correction and PARSE_BARCODES_SC instead of stopping at the counts table.

## Inputs

- Trimmed FASTQ reads (constant regions removed by cutadapt)
- Barcode reference library FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)
- bowtie1 index of the barcode reference

## Outputs

- Sorted SAM file of barcode alignments
- Per-sample rawcounts files (bulk mode, via samtools indexstats)
- counts/<sample_id>.counts.tsv (single-cell/spatial mode, via COUNT_BARCODES_SAM)
- bowtie/cutadapt/fastp QC metrics in the MultiQC report

## How to apply

Build a bowtie1 index of the barcode reference FASTA (BUILD_BOWTIE_INDEX) and align trimmed reads with the flags -a --best --strata -m1 --norc and alnmismatches = 2: -a --best --strata reports all best alignments, -m1 discards reads with multiple valid alignments (ambiguous multi-mappers), and --norc prevents reverse-complement hits. Constant regions should first be trimmed with cutadapt (e.g. upconstant TGACCATGTACGATTGACTA, downconstant TGCTAATGCGTACTGACTAG, constantmismatches 0.1, min_readlength 20; use --constants all for randomly fragmented single-cell/SAW reads). Apply FILTER_ALIGNMENTS, retaining only alignments that start at the first reference position or end at the last, so only full-length barcode-spanning hits are kept. Count alignments per barcode from the sorted SAM file with samtools indexstats (bulk) or COUNT_BARCODES_SAM (single-cell/spatial) to yield rawcounts or counts/<sample_id>.counts.tsv. Validate by checking that retained alignments span the full barcode and that MultiQC summaries from fastp/cutadapt/bowtie show expected read recovery.

## Related tools

- **bowtie1** (Indexes the barcode reference FASTA and aligns trimmed reads with -a --best --strata -m1 --norc and alnmismatches = 2) — http://bowtie-bio.sourceforge.net/index.shtml
- **cutadapt** (Trims flanking constant regions (e.g. upconstant TGACCATGTACGATTGACTA, downconstant TGCTAATGCGTACTGACTAG) so reads span the barcode for alignment) — https://cutadapt.readthedocs.io/en/stable/
- **samtools** (Sorts the alignment SAM and counts alignments per barcode (indexstats / COUNT_BARCODES_SAM) into counts tables)
- **BARtab** (Nextflow pipeline orchestrating index building, alignment (BUILD_BOWTIE_INDEX, FILTER_ALIGNMENTS) and barcode counting) — https://github.com/DaneVass/BARtab/

## Examples

```
nextflow run DaneVass/bartab -profile test_SE,docker
```

## Evaluation signals

- Retained alignments start at the first or end at the last reference position (FILTER_ALIGNMENTS) — full barcode-spanning hits only.
- Ambiguous multi-mappers are discarded: -m1 means each read yields at most one alignment, so counts tables contain no ambiguous assignments.
- Per-sample barcode-by-sample counts match the deposited exemplar test_PE run output when run on the test fastq files.
- bowtie, fastp and cutadapt QC metrics appear in the MultiQC report with expected read recovery at each step.
- In the single-cell FateMap comparison arm, the full BARtab workflow downstream of alignment reproduces 99.6% cell annotation overlap (12 BARtab-only, 145 FateMap-only, 39,884 both).

## Limitations

- Alignment-based quantification requires a reference library; evolving barcode methods are not officially supported by BARtab/bartools.
- Constant-region trimming parameters must match the barcode design: complexity_threshold may need to be raised (e.g. 66 for WSN-style barcodes) to remove poly-A empty-construct reads, and --constants all is recommended only for randomly fragmented reads.
- Reads with multiple equally-best alignments (-m1) are discarded, which trades sensitivity for specificity; alnmismatches = 2 bounds tolerated sequencing error.

## Evidence

- [methods] If a reference is provided, it is indexed using [bowtie1](http://bowtie-bio.sourceforge.net/index.shtml).: "If a reference is provided, it is indexed using [bowtie1](http://bowtie-bio.sourceforge.net/index.shtml)."
- [methods] Alignments that start at the first position or end at the last are retained.: "Alignments that start at the first position or end at the last are retained."
- [methods] Constant regions are trimmed and reads are filtered for length and N bases using [cutadapt](https://cutadapt.readthedocs.io/en/stable/).: "Constant regions are trimmed and reads are filtered for length and N bases using [cutadapt](https://cutadapt.readthedocs.io/en/stable/)."
- [readme] A Nextflow pipeline to tabulate synthetic barcode counts from NGS data: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
