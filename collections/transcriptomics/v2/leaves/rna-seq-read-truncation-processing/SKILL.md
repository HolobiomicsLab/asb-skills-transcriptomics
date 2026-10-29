---
name: rna-seq-read-truncation-processing
description: Use when you need to generate derivative sequencing-length variants (single-end
  truncations of paired-end FASTQ reads) to test robustness of an expression quantification
  metric across sequencing lengths and strategies.
when_to_use_negative:
- The input is already single-end or the question does not involve comparing quantification
  metrics across read lengths/strategies — truncation adds no information
- The dataset is poly(A)−/ribo− or RNase R-treated RNA-seq, where CIRCscore cannot
  be applied because polyadenylated linear RNAs are largely depleted and unsuitable
  for linear quantification
- Reads are shorter than 50 bp, so the 1×50 bp truncation cannot be meaningfully derived
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: clear_quant/circ_quant quantify FPBcirc, FPBlinear, and CIRCscore from original
    and truncated FASTQ/BAM inputs
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Produces the circRNA output file consumed by circ_quant (version >=2.3.6)
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Maps original and truncated reads to hg38, producing BAM input for circ_quant
    (>=2.0.5)
- name: pysam
  role: Python package used for the read-processing script that extracts read1 and
    truncates to 50 bp (>=0.8.4)
- name: pybedtools
  role: Python package supporting the read-processing/genomic interval handling in
    the workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:a86afd1c0350bfca26fbc7780ec283ea6649cc72dd5d94ae83d7e2e5b7ef216a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - StringTie
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# rna-seq-read-truncation-processing

## Summary

Construct virtual RNA-seq datasets by truncating reads from an existing paired-end ribo− dataset (e.g., keep read1 for 1×101 bp, keep the first 50 bp of read1 for 1×50 bp) to test whether a quantification parameter such as FPB is resistant to differences in sequencing lengths and strategies, in contrast to FPM.

## When to use

Use when you need to generate derivative sequencing-length variants (single-end truncations of paired-end FASTQ reads) to test robustness of an expression quantification metric across sequencing lengths and strategies.

## When NOT to use

- The input is already single-end or the question does not involve comparing quantification metrics across read lengths/strategies — truncation adds no information
- The dataset is poly(A)−/ribo− or RNase R-treated RNA-seq, where CIRCscore cannot be applied because polyadenylated linear RNAs are largely depleted and unsuitable for linear quantification
- Reads are shorter than 50 bp, so the 1×50 bp truncation cannot be meaningfully derived

## Inputs

- Paired-end 2×101 bp ribo− RNA-seq FASTQ files (e.g., GSM2072380 cortex from GEO)
- hg38 genome FASTA file
- hg38 HISAT2 index and Bowtie1 (TopHat-Fusion) index
- Annotation GTF file and refFlat-format gene annotation
- Read-processing script (pysam/pybedtools) for read1 extraction and 50 bp truncation

## Outputs

- Derivative FASTQ datasets: 1×101 bp single-end and 1×50 bp single-end
- HISAT2 BAM alignments for original and truncated datasets
- quant.txt files with readNumber, FPBcirc, FPBlinear, and CIRCscore per circRNA
- Median FPM, FPB, and FPKM comparison table/plot (Figure 2E-style) across the three datasets

## How to apply

Download the source paired-end FASTQ files (e.g., 2×101 bp ribo− cortex RNA-seq for GSM2072380 from GEO) and use a read-processing script (pysam/pybedtools ecosystem) to create two derivative datasets: one keeping only read1 (1×101 bp, single-end) and one keeping the first 50 bp of read1 (1×50 bp, single-end). Quantify each derivative with clear_quant in single-end mode (omit -2) and the original in paired-end mode, mapping with HISAT2 (>=2.0.5) against hg38 FASTA with the hg38 HISAT2 and Bowtie1 indices and annotation GTF. Then run circ_quant from CIRCexplorer2 output with the HISAT2 BAM (which must not contain unmapped reads), refFlat annotation, and defaults (--threshold 1, --ratio 1) to obtain readNumber, FPBcirc, FPBlinear, and CIRCscore per circRNA in quant.txt. Compare median FPB, FPM, and FPKM across the three datasets (as in Figure 2E) to decide whether the metric is length-strategy invariant: FPB should stay largely unchanged while FPM varies with read length.

## Related tools

- **CIRCexplorer3-CLEAR** (clear_quant/circ_quant quantify FPBcirc, FPBlinear, and CIRCscore from original and truncated FASTQ/BAM inputs) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Produces the circRNA output file consumed by circ_quant (version >=2.3.6)) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Maps original and truncated reads to hg38, producing BAM input for circ_quant (>=2.0.5))
- **pysam** (Python package used for the read-processing script that extracts read1 and truncates to 50 bp (>=0.8.4))
- **pybedtools** (Python package supporting the read-processing/genomic interval handling in the workflow)

## Examples

```
clear_quant -1 read1_trunc50.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir_1x50
```

## Evaluation signals

- Median FPB of the 1×101 and 1×50 datasets remains close to the median FPB of the full 2×101 dataset
- Median FPM deviates with read length across the three datasets, providing the expected contrast with FPB
- All three quant.txt outputs exist with populated FPBcirc, FPBlinear, and CIRCscore fields
- The hisat_aligned.bam input to circ_quant contains no unmapped reads
- Each circRNA entry carries a valid readNumber and a fusion-junction realignment score flag in the output schema

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate
- Derivative datasets are single-end, so clear_quant must be run without -2; paired-end-specific quantification behavior is not preserved
- CIRCexplorer2 version >=2.3.6 is required for compatibility with CLEAR quantification inputs/outputs

## Evidence

- [other] When virtual RNA-seq datasets (1 × 101 bp and 1 × 50 bp) were constructed from the 2 × 101 bp cortex ribo− data, FPB largely remained unchanged: [evidence span withheld — anchor status: not_found]
- [abstract] A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually"
- [readme] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
- [readme] hisat_aligned.bam should not contain unmapped reads.: "hisat_aligned.bam should not contain unmapped reads."
- [readme] -2 is only used when running with pair-end read. This should contain #2 mates.: "-2 is only used when running with pair-end read. This should contain #2 mates."
