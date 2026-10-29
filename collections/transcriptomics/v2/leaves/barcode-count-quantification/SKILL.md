---
name: barcode-count-quantification
description: Use when raw bulk cellular barcoding FASTQ (single- or paired-end) must
  be converted into a barcode-by-sample counts table, with documented QC metrics at
  each stage.
when_to_use_negative:
- Input is already a barcode counts table or processed single-cell/spatial data —
  use bartools downstream analysis or the BARtab single-cell workflow instead
- No barcode reference library exists and a reference-free mode (e.g. test_SE_ref_free)
  is required rather than bowtie-based quantification
- Barcode is sequenced inline with cell barcodes/UMIs such that PCR chimerism removal
  and UMI-based filtering (REMOVE_PCR_CHIMERISM, umi_count_filter) are the priority
  rather than bulk read counting
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BARtab
  role: Nextflow pipeline orchestrating merge, filter, trim, align, and count stages
    to generate barcode counts tables
  repo: https://github.com/DaneVass/BARtab
- name: FLASh
  role: Merges forward and reverse reads in the paired-bulk mode
  repo: http://ccb.jhu.edu/software/FLASH/
- name: fastp
  role: Quality-filters merged reads (minqual, pctqual, complexity_threshold)
  repo: https://github.com/OpenGene/fastp
- name: cutadapt
  role: Trims flanking constant regions and filters reads for length and N bases
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: bowtie1
  role: Indexes the barcode reference library and aligns trimmed reads with mismatch
    tolerances
  repo: http://bowtie-bio.sourceforge.net/index.shtml
- name: samtools
  role: Counts reads per barcode from the sorted SAM file (indexstats)
- name: MultiQC
  role: Aggregates per-stage QC metrics into one report to validate processing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:73a316398c2bfe25a1dee0ab55ba58e90fa7c570a39703846cfdb6b69fce53d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FLASh
  - MultiQC
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# barcode-count-quantification

## Summary

End-to-end BARtab Nextflow pre-processing and quantification of population-level cellular barcoding FASTQ data: paired-end merging with FLASh, fastp quality filtering, cutadapt constant-region trimming, bowtie1 reference alignment, and per-sample barcode read counting into a barcode-by-sample counts table.

## When to use

Use when raw bulk cellular barcoding FASTQ (single- or paired-end) must be converted into a barcode-by-sample counts table, with documented QC metrics at each stage.

## When NOT to use

- Input is already a barcode counts table or processed single-cell/spatial data — use bartools downstream analysis or the BARtab single-cell workflow instead
- No barcode reference library exists and a reference-free mode (e.g. test_SE_ref_free) is required rather than bowtie-based quantification
- Barcode is sequenced inline with cell barcodes/UMIs such that PCR chimerism removal and UMI-based filtering (REMOVE_PCR_CHIMERISM, umi_count_filter) are the priority rather than bulk read counting

## Inputs

- Single- or paired-end FASTQ files from bulk population-level barcoding experiments (e.g. the test_SE / test_PE datasets)
- Barcode reference library FASTQ/FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)

## Outputs

- Per-sample raw barcode counts files (samtools indexstats from sorted SAM)
- Combined barcode-by-sample counts table (outer-joined via COMBINE_BARCODE_COUNTS)
- MultiQC report summarizing fastqc, flash, fastp, cutadapt, and bowtie QC metrics

## How to apply

Run the BARtab paired-bulk workflow ('nextflow run DaneVass/bartab --mode paired-bulk'): merge forward/reverse reads with FLASh (mergeoverlap = 10), then quality-filter merged reads with fastp using minqual = 20 and pctqual = 80 (complexity_threshold defaults to 0, but raise it, e.g. 66 for WSN-style barcodes, to remove poly-A empty-construct reads). Trim flanking constant regions with cutadapt (e.g. SPLINTR upconstant = TGACCATGTACGATTGACTA and downconstant = TGCTAATGCGTACTGACTAG, constantmismatches = 0.1, min_readlength = 20, barcode_length = 60). Build a bowtie1 index of the barcode reference library (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta) and align trimmed reads with alnmismatches = 2 and flags -a --best --strata -m1 --norc so ambiguous multi-mappers are discarded. Count reads per barcode from the sorted SAM with samtools indexstats, combine per-sample rawcounts with an outer join (COMBINE_BARCODE_COUNTS), and validate every stage via the MultiQC report (fastqc, flash, fastp, cutadapt, bowtie metrics).

## Related tools

- **BARtab** (Nextflow pipeline orchestrating merge, filter, trim, align, and count stages to generate barcode counts tables) — https://github.com/DaneVass/BARtab
- **FLASh** (Merges forward and reverse reads in the paired-bulk mode) — http://ccb.jhu.edu/software/FLASH/
- **fastp** (Quality-filters merged reads (minqual, pctqual, complexity_threshold)) — https://github.com/OpenGene/fastp
- **cutadapt** (Trims flanking constant regions and filters reads for length and N bases) — https://cutadapt.readthedocs.io/en/stable/
- **bowtie1** (Indexes the barcode reference library and aligns trimmed reads with mismatch tolerances) — http://bowtie-bio.sourceforge.net/index.shtml
- **samtools** (Counts reads per barcode from the sorted SAM file (indexstats))
- **MultiQC** (Aggregates per-stage QC metrics into one report to validate processing)

## Examples

```
nextflow run DaneVass/bartab --mode paired-bulk
```

## Evaluation signals

- Combined barcode-by-sample counts table matches the deposited exemplar test_PE run output exactly
- fastp filtering reports reads meeting minqual = 20 across ≥ pctqual = 80% of bases; complexity filtering removes poly-A empty-construct reads when threshold is raised (e.g. 66 for WSN barcodes)
- cutadapt reports trimmed reads at or above min_readlength = 20 and barcode_length = 60 with constant-region mismatch fraction within constantmismatches = 0.1
- bowtie alignment with -m1 --norc flags discards multi-mappers, so counts derive only from unambiguous alignments (≤ alnmismatches = 2)
- MultiQC report shows plausible, non-zero metrics for every stage (fastqc, flash, fastp, cutadapt, bowtie), confirming each processing step ran

## Limitations

- complexity_threshold default of 0 retains poly-A reads from empty barcode constructs; the threshold must be tuned per barcode design (e.g. 66 for WSN-style barcodes)
- Benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Benchmarking on simulated data (for accuracy and optimal parameter selection) was not performed, so accuracy and optimal settings are not formally established

## Evidence

- [methods] If running in mode paired-bulk, forward and reverse reads are merged using FLASh.: "If running in mode paired-bulk, forward and reverse reads are merged using [FLASh](http://ccb.jhu.edu/software/FLASH/)."
- [methods] Reads are quality filtered using fastp.: "Reads are quality filtered using [fastp](https://github.com/OpenGene/fastp)."
- [methods] Constant regions are trimmed and reads are filtered for length and N bases using cutadapt.: "Constant regions are trimmed and reads are filtered for length and N bases using [cutadapt](https://cutadapt.readthedocs.io/en/stable/)."
- [methods] The complexity_threshold default of 0 can be set to remove poly-A tails from empty barcode constructs.: "`complexity_threshold` (default 0) can be set to remove reads with e.g. poly-A tails due to an empty barcode contstruct."
- [methods] Alignments that start at the first position or end at the last are retained.: "Alignments that start at the first position or end at the last are retained."
- [readme] A Nextflow pipeline to tabulate synthetic barcode counts from NGS data: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
- [abstract] we present BARtab, a portable and scalable Nextflow pipeline: "we present BARtab, a portable and scalable Nextflow pipeline"
