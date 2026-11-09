---
name: barcode-counts-quality-filtering
description: Use when raw FASTQ or a raw barcode counts table from a population-level
  cellular barcoding experiment must be QC-filtered before downstream clonal analysis,
  and low-quality samples or low-abundance barcodes must be removed.
when_to_use_negative:
- Input is already a QC-filtered counts table matching the study's reported dimensions
  (1,680 barcodes x 41 samples) — re-filtering would remove valid barcodes
- The analysis targets single-cell or spatial barcode extraction with UMI-based filtering
  (umi_count_filter / umi_fraction_filter) rather than bulk read-count filtering
- The barcode design has no reference library and reference-free mode is required
  instead of bowtie1-based quantification
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3361
tools:
- name: BARtab
  role: Nextflow pipeline performing MERGE_READS, FILTER_READS, CUTADAPT_READS, BUILD_BOWTIE_INDEX
    and barcode quantification
  repo: https://github.com/DaneVass/BARtab/
- name: fastp
  role: Quality filtering of merged reads via minqual, pctqual, and complexity_threshold
    parameters
  repo: https://github.com/OpenGene/fastp
- name: cutadapt
  role: Trims flanking constant regions and filters reads for length and N bases before
    alignment
- name: bowtie1
  role: Aligns trimmed reads to the barcode reference with -a --best --strata -m1
    --norc, discarding ambiguous multi-mappers
- name: bartools
  role: R package providing sample-level (5th-percentile) and per-barcode (<10 reads
    in <4 samples) filtering of the counts table
  repo: https://github.com/DaneVass/bartools/
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:5f96ea622d46870f312aaa0319369bf43de14b0c04396db405081ef2b72c12aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FLASh
  - MultiQC
  - R
  - bartools
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# barcode-counts-quality-filtering

## Summary

Quality-control procedure for synthetic cellular barcoding datasets that combines read-level filtering in the BARtab Nextflow pipeline (fastp quality filtering, cutadapt constant-region trimming, bowtie alignment with multi-mapper removal) with sample-level (5th-percentile read-count outlier) and per-barcode (>=10 reads in >=4 samples) filters in bartools/R to yield a high-confidence barcode-by-sample counts matrix.

## When to use

Use when raw FASTQ or a raw barcode counts table from a population-level cellular barcoding experiment must be QC-filtered before downstream clonal analysis, and low-quality samples or low-abundance barcodes must be removed.

## When NOT to use

- Input is already a QC-filtered counts table matching the study's reported dimensions (1,680 barcodes x 41 samples) — re-filtering would remove valid barcodes
- The analysis targets single-cell or spatial barcode extraction with UMI-based filtering (umi_count_filter / umi_fraction_filter) rather than bulk read-count filtering
- The barcode design has no reference library and reference-free mode is required instead of bowtie1-based quantification

## Inputs

- Raw paired-end or single-end FASTQ files from a population-level barcoding run
- Barcode reference library FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)
- Raw barcode-by-sample counts table (e.g. from GEO accession GSE246611)

## Outputs

- Filtered barcode-by-sample counts matrix (exemplar: 1,680 barcodes x 41 samples) written as TSV, e.g. dose_escalation_filtered_counts.tsv
- Per-sample rawcounts files from samtools indexstats and a combined counts table (COMBINE_BARCODE_COUNTS)
- MultiQC report summarizing fastqc, flash, fastp, cutadapt, and bowtie QC metrics

## How to apply

First run read-level QC in BARtab: merge paired-end reads with FLASh (mergeoverlap = 10), quality-filter with fastp using minqual = 20 and pctqual = 80 (set complexity_threshold, e.g. 66 for WSN-style barcodes, to drop poly-A empty-construct reads), trim flanking constant regions with cutadapt (constantmismatches = 0.1, min_readlength = 20, barcode_length = 60), and quantify with bowtie1 (alnmismatches = 2, flags -a --best --strata -m1 --norc) to discard ambiguous multi-mappers. Then, on the barcode-by-sample counts table in R/tidyverse, compute per-sample total read counts, flag samples below the 5th percentile as low-quality outliers and remove them. Next apply the low-stringency per-barcode threshold, retaining only barcodes with at least 10 reads in at least 4 samples; on the GSE246611 dose-escalation dataset this removed 6 samples and 110 low-abundance barcodes. Validate that the resulting matrix has the expected dimensions (1,680 barcodes x 41 samples for the exemplar dataset) and that per-sample QC metrics (fastqc, flash, fastp, cutadapt, bowtie) appear in the MultiQC report.

## Related tools

- **BARtab** (Nextflow pipeline performing MERGE_READS, FILTER_READS, CUTADAPT_READS, BUILD_BOWTIE_INDEX and barcode quantification) — https://github.com/DaneVass/BARtab/
- **fastp** (Quality filtering of merged reads via minqual, pctqual, and complexity_threshold parameters) — https://github.com/OpenGene/fastp
- **cutadapt** (Trims flanking constant regions and filters reads for length and N bases before alignment)
- **bowtie1** (Aligns trimmed reads to the barcode reference with -a --best --strata -m1 --norc, discarding ambiguous multi-mappers)
- **bartools** (R package providing sample-level (5th-percentile) and per-barcode (<10 reads in <4 samples) filtering of the counts table) — https://github.com/DaneVass/bartools/

## Examples

```
nextflow run DaneVass/bartab --mode paired-bulk -profile test_PE,docker
```

## Evaluation signals

- Output TSV exists, parses as valid TSV, and its dimensions equal 1,680 barcodes x 41 samples for the GSE246611 dose-escalation exemplar
- Exactly 6 samples are removed by the 5th-percentile per-sample read-count outlier filter and 110 barcodes by the low-stringency per-barcode filter, as reported in the study
- Combined barcode-by-sample counts table matches the deposited exemplar test_PE run output after COMBINE_BARCODE_COUNTS outer join
- MultiQC report contains summary metrics for every processing stage (fastqc, flash, fastp, cutadapt, bowtie)
- No reads with <10 reads in fewer than 4 samples remain in the filtered matrix (invariant of the per-barcode threshold)

## Limitations

- No gold-standard dataset with known barcode abundance exists, so filter-stringency choices cannot be benchmarked against ground truth
- The default complexity_threshold of 0 retains poly-A empty-construct reads unless raised (e.g. 66 for WSN-style barcodes)
- A Levenshtein clustering distance of 3 is used by default and is conservative, so genuinely distinct barcodes are not collapsed but near-identical sequencing-error variants may inflate counts
- Benchmarking against similar tools is hampered by differing parameter flexibility (e.g. xcalibr only accepts exact matches to the reference)

## Evidence

- [methods] Reads are quality filtered using fastp, and the minimum quality score is set with minqual: "Reads are quality filtered using [fastp](https://github.com/OpenGene/fastp)."
- [methods] complexity_threshold can remove empty-construct poly-A reads: "`complexity_threshold` (default 0) can be set to remove reads with e.g. poly-A tails due to an empty barcode contstruct."
- [methods] Minimum read length default for cutadapt trimming: "The minimum read length can be specified with `min_readlength` (default 20)."
- [readme] BARtab purpose as a barcode counts tabulation pipeline: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
- [discussion] Rationale for filtering stringency decisions: "benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance"
- [intro] bartools as the downstream R analysis package: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
