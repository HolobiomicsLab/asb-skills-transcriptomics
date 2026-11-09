---
name: single-cell-barcode-extraction
description: Use when raw paired-end single-cell amplicon FASTQ from an expressed
  synthetic barcoding experiment must be converted into per-cell lineage barcode annotation
  and clone-size tables, e.g. to cross-validate an existing pipeline like FateMap.
when_to_use_negative:
- Input is already a processed barcode counts/annotation table — this skill starts
  from raw single-cell amplicon FASTQ
- Data are population-level (bulk) barcode sequencing with no cell barcodes — use
  the BARtab bulk mode workflow instead
- Study uses evolving barcode methods (e.g. Cas9-based recorders), which BARtab and
  bartools do not officially support
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: BARtab
  role: Nextflow pipeline orchestrating single-cell barcode extraction, alignment,
    UMI counting, and cell annotation
  repo: github.com/DaneVass/BARtab
- name: FastQC
  role: Quality profiling of raw FASTQ reads
- name: fastp
  role: Read quality filtering (minqual, pctqual parameters)
  repo: https://github.com/OpenGene/fastp
- name: umi-tools
  role: Cell barcode/UMI extraction (whitelist/extract) and UMI error correction and
    counting (count_tab)
  repo: https://github.com/CGATOxford/UMI-tools
- name: cutadapt
  role: Trimming of constant regions and length/N filtering of reads
- name: bowtie1
  role: Indexing the SPLINTR barcode reference library and aligning trimmed barcode
    reads with -a --best --strata -m1 --norc
- name: starcode
  role: starcode-umi used for UMI-based clustering in barcode processing
  repo: https://github.com/gui11aume/starcode
- name: R / bartools
  role: Downstream per-cell annotation comparison and clone-size correlation analysis
    (e.g. vs FateMap)
  repo: https://github.com/DaneVass/bartools/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:faa726bcc5fc62b78668ccb57d6f6e9a03b3ec95cabd6c36677f9ce9588a0abd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BARtab
  - FastQC
  - R
  - R / bartools
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  - starcode
  - umi-tools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-barcode-extraction

## Summary

Extract, error-correct, and tabulate expressed synthetic lineage barcodes per cell from single-cell amplicon sequencing FASTQ using the BARtab Nextflow pipeline (with fastp, umi-tools, cutadapt, bowtie1), producing a per-cell barcode annotation table comparable to FateMap-style outputs.

## When to use

Use when raw paired-end single-cell amplicon FASTQ from an expressed synthetic barcoding experiment must be converted into per-cell lineage barcode annotation and clone-size tables, e.g. to cross-validate an existing pipeline like FateMap.

## When NOT to use

- Input is already a processed barcode counts/annotation table — this skill starts from raw single-cell amplicon FASTQ
- Data are population-level (bulk) barcode sequencing with no cell barcodes — use the BARtab bulk mode workflow instead
- Study uses evolving barcode methods (e.g. Cas9-based recorders), which BARtab and bartools do not officially support

## Inputs

- Paired-end single-cell amplicon FASTQ files (e.g. FateMap FM0-2 dataset)
- Cell barcode whitelist files (<sample_id>_whitelist.tsv, '-1' suffixes removed, e.g. derived from GEO)
- Reference barcode library FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)

## Outputs

- Per-cell lineage barcode annotation table (cell barcode to barcode counts/assignments)
- Per-barcode-per-cell UMI count matrices (after umi_dist 1 error correction, umi_count_filter 5)
- QC reports (FastQC, fastp HTML/JSON reports)
- Per-cell clone sizes for downstream comparison in R (e.g. against FateMap annotations)

## How to apply

Run `nextflow run DaneVass/bartab --mode single-cell --input_type fastq` on paired-end amplicon reads, supplying GEO-derived cell barcode whitelists as <sample_id>_whitelist.tsv via --whitelist_indir with '-1' suffixes removed. QC with FastQC, then quality-filter with fastp (minqual 20, pctqual 80). Extract 16bp cell barcodes and 10bp UMIs from R1 with umi-tools whitelist/extract using the pattern CCCCCCCCCCCCCCCCNNNNNNNNNNNN, trim constant regions with cutadapt (--constants all for reference-based randomly fragmented reads), build a bowtie1 index of the SPLINTR barcode reference library, and align with bowtie -a --best --strata -m1 --norc (alnmismatches 2), keeping only alignments that start at the first or end at the last reference position (FILTER_ALIGNMENTS). Remove PCR chimerism by keeping only the most-supported cell barcode-UMI-lineage barcode combination, error-correct and count UMIs per barcode per cell with umi-tools count_tab (umi_dist 1, umi_count_filter 5), then aggregate per cell with PARSE_BARCODES_SC (umi_fraction_filter 0.3). Correct application yields a cell barcode annotation table whose overlap with an independent pipeline (e.g. FateMap, UMI threshold 15) should reach ~99.6% and Pearson clone-size correlations between 0.965 and 0.998.

## Related tools

- **BARtab** (Nextflow pipeline orchestrating single-cell barcode extraction, alignment, UMI counting, and cell annotation) — github.com/DaneVass/BARtab
- **FastQC** (Quality profiling of raw FASTQ reads)
- **fastp** (Read quality filtering (minqual, pctqual parameters)) — https://github.com/OpenGene/fastp
- **umi-tools** (Cell barcode/UMI extraction (whitelist/extract) and UMI error correction and counting (count_tab)) — https://github.com/CGATOxford/UMI-tools
- **cutadapt** (Trimming of constant regions and length/N filtering of reads)
- **bowtie1** (Indexing the SPLINTR barcode reference library and aligning trimmed barcode reads with -a --best --strata -m1 --norc)
- **starcode** (starcode-umi used for UMI-based clustering in barcode processing) — https://github.com/gui11aume/starcode
- **R / bartools** (Downstream per-cell annotation comparison and clone-size correlation analysis (e.g. vs FateMap)) — https://github.com/DaneVass/bartools/

## Examples

```
nextflow run DaneVass/bartab --mode single-cell --input_type fastq --input 'FM0-2/*_R{1,2}.fastq.gz' --whitelist_indir whitelists/
```

## Evaluation signals

- Cell annotation overlap against the reference pipeline (FateMap, UMI threshold 15) reproduces 99.6% (12 BARtab-only, 145 FateMap-only, 39,884 shared cells across four conditions)
- Pearson correlation of clone sizes between BARtab and FateMap falls between 0.965 and 0.998
- FILTER_ALIGNMENTS step retains only alignments starting at the first or ending at the last reference position
- Post REMOVE_PCR_CHIMERISM, each cell barcode-UMI-lineage barcode combination is represented by only its most-supported read count
- fastp filtering applied with minqual 20 and pctqual 80, and PARSE_BARCODES_SC applied with umi_fraction_filter 0.3, are present in the run report

## Limitations

- Benchmarking against similar tools is hampered by the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- No accepted gold-standard analysis pipeline exists for single-cell cellular barcoding, so validation relies on reproducing prior published annotations
- Only exact matches to the reference are counted by some alternative tools (e.g. xcalibr); parameter choices such as umi_count_filter and umi_fraction_filter materially affect per-cell annotation results

## Evidence

- [methods] Reads are quality filtered using fastp: "Reads are quality filtered using [fastp](https://github.com/OpenGene/fastp)."
- [methods] FILTER_ALIGNMENTS positional filtering: "Alignments that start at the first position or end at the last are retained."
- [methods] PCR chimerism removal rule: "Reads from PCR chimerism are removed by only retaining the cell barcode-umi-lineage barcode combination supported by the most reads."
- [methods] UMI-based barcode filtering parameters: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`) and by the number of UMIs in comparison to the dominant barcode per cell (`--umi_fraction_filter`)."
- [discussion] Benchmark-by-reproduction finding: "We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity."
- [readme] Pipeline purpose from README: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
- [methods] Cutadapt role in workflow: "Constant regions are trimmed and reads are filtered for length and N bases using [cutadapt](https://cutadapt.readthedocs.io/en/stable/)."
