---
name: constant-region-adapter-trimming
description: Use when raw barcode amplicon fastq reads still carry flanking constant
  sequences upstream/downstream of the barcode and must be trimmed before alignment
  to a barcode reference library (e.g. SPLINTR_mCHERRY_V2).
when_to_use_negative:
- Input reads have already been trimmed and contain only barcode sequence (proceed
  directly to reference alignment/counting)
- Barcodes are extracted from single-cell BAM/spatial data via a different BARtab
  mode rather than bulk amplicon fastq
- The barcode design has no flanking constant region to anchor trimming (use a reference-free
  or template-free extraction tool such as xcalibr or pycashier)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: cutadapt
  role: Trims constant regions and filters reads for length and N bases in the CUTADAPT_READS
    stage
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: BARtab
  role: Nextflow pipeline whose bulk workflow wraps the trimming stage between fastp
    quality filtering and bowtie1 alignment
  repo: https://github.com/DaneVass/BARtab
- name: fastp
  role: Upstream quality and complexity filtering (minqual, pctqual, complexity_threshold)
    that feeds trimmed reads to cutadapt
  repo: https://github.com/OpenGene/fastp
- name: MultiQC
  role: Summarizes cutadapt (and fastqc/flash/fastp/bowtie) metrics to validate the
    trimming step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:147bf45bce25f7b838e083aacecad84c926cba6143cf7928e4e99f311ced33b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BARtab
  - FLASh
  - MultiQC
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# constant-region-adapter-trimming

## Summary

Trim flanking constant regions from barcode reads and filter for length and N bases using cutadapt, a stage of the BARtab bulk Nextflow workflow that prepares reads for reference-based barcode alignment and counting.

## When to use

Use when raw barcode amplicon fastq reads still carry flanking constant sequences upstream/downstream of the barcode and must be trimmed before alignment to a barcode reference library (e.g. SPLINTR_mCHERRY_V2).

## When NOT to use

- Input reads have already been trimmed and contain only barcode sequence (proceed directly to reference alignment/counting)
- Barcodes are extracted from single-cell BAM/spatial data via a different BARtab mode rather than bulk amplicon fastq
- The barcode design has no flanking constant region to anchor trimming (use a reference-free or template-free extraction tool such as xcalibr or pycashier)

## Inputs

- Quality-filtered fastq reads (merged paired-end or single-end) with flanking constant regions, output of the BARtab FILTER_READS/fastp stage
- Constant region sequences flanking the barcode (e.g. SPLINTR up- and down-constants)

## Outputs

- Constant-region-trimmed, length- and N-filtered fastq reads ready for bowtie1 alignment to the barcode reference library
- cutadapt trimming report/QC metrics for MultiQC summarization

## How to apply

After paired-end reads are merged (FLASh) and quality-filtered (fastp, e.g. minqual 20, pctqual 80), run the CUTADAPT_READS stage: trim the flanking constant regions (e.g. SPLINTR upconstant TGACCATGTACGATTGACTA and downconstant TGCTAATGCGTACTGACTAG, constants = 'up') with a tolerated mismatch fraction set via constantmismatches (default 0.1). Cutadapt simultaneously filters reads by minimum length (min_readlength, default 20) and by N bases. Set up_coverage/down_coverage overlap requirements (e.g. 3) and min_readlength/barcode_length so only full-length candidate barcodes survive; too-strict settings discard valid barcodes, too-lenient ones pass empty poly-A construct reads, which complexity_threshold in the preceding fastp step can remove. Rationale: exact-enough constant-region anchors plus length filtering maximize the fraction of trimmed reads that map unambiguously to the barcode reference in the subsequent bowtie1 alignment.

## Related tools

- **cutadapt** (Trims constant regions and filters reads for length and N bases in the CUTADAPT_READS stage) — https://cutadapt.readthedocs.io/en/stable/
- **BARtab** (Nextflow pipeline whose bulk workflow wraps the trimming stage between fastp quality filtering and bowtie1 alignment) — https://github.com/DaneVass/BARtab
- **fastp** (Upstream quality and complexity filtering (minqual, pctqual, complexity_threshold) that feeds trimmed reads to cutadapt) — https://github.com/OpenGene/fastp
- **MultiQC** (Summarizes cutadapt (and fastqc/flash/fastp/bowtie) metrics to validate the trimming step)

## Examples

```
nextflow run DaneVass/bartab --mode paired-bulk -profile test_PE,docker
```

## Evaluation signals

- cutadapt report shows a high fraction of reads with the constant region found within the allowed constantmismatches fraction (default 0.1)
- Trimmed read lengths are consistent with the configured min_readlength (default 20) and barcode_length (e.g. 60)
- Per-sample QC metrics for the cutadapt step appear in the MultiQC report alongside fastqc, flash, fastp and bowtie summaries
- Downstream bowtie1 alignment with alnmismatches = 2 and -m1 yields expected per-barcode counts, and combined barcode-by-sample counts match the deposited test_PE exemplar output

## Limitations

- Benchmarking BARtab against similar tools was difficult due to lack of a gold-standard dataset with known barcode abundance, so optimal trimming parameter selection was not validated on simulated data
- Low-complexity reads from empty barcode constructs (e.g. poly-A tails) are not removed by trimming itself; set a higher complexity_threshold in the upstream fastp step (e.g. 66 for WSN-style barcodes)
- Parameters are barcode-design specific: constant sequences, constantmismatches, coverage and length settings must be adapted to each barcode construct

## Evidence

- [methods] CUTADAPT_READS: "Constant regions are trimmed and reads are filtered for length and N bases using [cutadapt](https://cutadapt.readthedocs.io/en/stable/)."
- [methods] constantmismatches default: "The fraction of mismatches in the constant region can be specified with `constantmismatches` (default 0.1)."
- [methods] min_readlength default: "The minimum read length can be specified with `min_readlength` (default 20)."
- [readme] BARtab pipeline purpose: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
- [methods] test profiles: "Run any of the test datasets using `nextflow run DaneVass/bartab -profile <test_SE,test_PE,test_SE_ref_free,test_sc,test_sc_bam,test_sc_saw_fastq>,<conda,docker,singularity>,<slurm,lsf>`"
