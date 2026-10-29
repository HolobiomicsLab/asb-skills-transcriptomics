---
name: hisat2-rna-seq-read-alignment
description: Use when quantifying circular and linear RNA expression from ribo− RNA-seq
  FASTQ files with CIRCexplorer3-CLEAR, which requires HISAT2-aligned BAM input for
  CIRCexplorer2 calling and FPB/CIRCscore computation.
when_to_use_negative:
- Input RNA-seq is poly(A)+ rather than ribo−; CIRCscore evaluation depends on linear
  RNA quantification that poly(A)+ depletion contexts (poly(A)−/ribo−, RNase R-treated)
  make unreliable
- Input is already a HISAT2-aligned BAM plus CIRCexplorer2 output and only FPB/CIRCscore
  extraction is needed — circ_quant should be used directly instead of re-aligning
- The mapped BAM still contains unmapped reads — circ_quant explicitly requires a
  BAM without unmapped reads
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HISAT2
  role: Spliced read aligner producing the BAM of mapped reads consumed by CIRCexplorer2/CLEAR
    quantification (requires >=2.0.5)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: CIRCexplorer3-CLEAR
  role: Pipeline (clear_quant/circ_quant) that embeds HISAT2 alignment and uses its
    BAM output to compute FPBcirc, FPBlinear, and CIRCscore
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Consumes the HISAT2 BAM for fusion junction realignment and circRNA identification;
    requires >=2.3.6
  repo: https://github.com/YangLab/CIRCexplorer2
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:baf19a61567877ad6b472a549f2808934fc9126f676ae2990e51660de35bc4c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - CLEAR
  - DCC/circtools
  - HISAT2
  - StringTie
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hisat2-rna-seq-read-alignment

## Summary

Maps ribo− RNA-seq reads to the genome with HISAT2 (>=2.0.5) to produce the BAM that the CLEAR/CIRCexplorer3 pipeline requires for circRNA (CIRCexplorer2) and linear-RNA quantification. The BAM must be stripped of unmapped reads before downstream circ_quant processing.

## When to use

Use when quantifying circular and linear RNA expression from ribo− RNA-seq FASTQ files with CIRCexplorer3-CLEAR, which requires HISAT2-aligned BAM input for CIRCexplorer2 calling and FPB/CIRCscore computation.

## When NOT to use

- Input RNA-seq is poly(A)+ rather than ribo−; CIRCscore evaluation depends on linear RNA quantification that poly(A)+ depletion contexts (poly(A)−/ribo−, RNase R-treated) make unreliable
- Input is already a HISAT2-aligned BAM plus CIRCexplorer2 output and only FPB/CIRCscore extraction is needed — circ_quant should be used directly instead of re-aligning
- The mapped BAM still contains unmapped reads — circ_quant explicitly requires a BAM without unmapped reads

## Inputs

- Paired-end or single-end ribo− RNA-seq FASTQ files (e.g., 2x101 bp PA1 or cortex datasets)
- hg38 genome FASTA (hg38.fa)
- hg38 HISAT2 index
- hg38 Bowtie1/TopHat-Fusion index for TopHat-Fusion
- Annotation GTF file
- refFlat-format gene annotation (for circ_quant)

## Outputs

- HISAT2-aligned BAM free of unmapped reads
- output_dir/quant/quant.txt with per-circRNA fields: chrom, start, end, name, score, strand, readNumber, FPBcirc, FPBlinear, CIRCscore

## How to apply

Run clear_quant starting from FASTQ files, supplying the hg38 genome FASTA (-g), the HISAT2 index (-i), the TopHat-Fusion/Bowtie1 index (-j), and the annotation GTF (-G); clear_quant internally aligns reads with HISAT2 (>=2.0.5) as part of the CIRCexplorer2-based quantification producing output_dir/quant/quant.txt. Alternatively, align reads separately with HISAT2 and pass the resulting BAM to circ_quant via -b, together with the CIRCexplorer2 circRNA file (-c) and refFlat annotation (-r); the BAM must not contain unmapped reads. Both paired-end (2x101 bp, -1/-2) and single-end runs are supported, so derivative datasets such as 1x101 or 1x50 bp reads can be aligned for FPB-stability comparisons. Correct alignment matters because BSJ- and SJ-mapped fragments feed FPBcirc, FPBlinear, and ultimately CIRCscore (FPBcirc/FPBlinear); under-mapping BSJ-spanning fragments degrades CIRCscore accuracy relative to tools like DCC/circtools CLR.

## Related tools

- **HISAT2** (Spliced read aligner producing the BAM of mapped reads consumed by CIRCexplorer2/CLEAR quantification (requires >=2.0.5)) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **CIRCexplorer3-CLEAR** (Pipeline (clear_quant/circ_quant) that embeds HISAT2 alignment and uses its BAM output to compute FPBcirc, FPBlinear, and CIRCscore) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Consumes the HISAT2 BAM for fusion junction realignment and circRNA identification; requires >=2.3.6) — https://github.com/YangLab/CIRCexplorer2

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt exists at output_dir/quant/ and contains populated readNumber, FPBcirc, FPBlinear, and CIRCscore fields
- The BAM passed to circ_quant contains no unmapped reads (README: 'hisat_aligned.bam should not contain unmapped reads')
- For the circCAMSAP1-1 locus, the CIRCexplorer2/CLEAR path identifies ~126 BSJ fragments and yields CIRCscore ~3.65 (matching RT-qPCR ~3.82), versus only ~36 fragments and CLR ~0.80 from DCC/circtools
- Median FPB of virtual single-end derivatives (1x101, 1x50 bp from 2x101 bp data) remains close to the paired-end original while median FPM deviates
- HISAT2 version is >=2.0.5 per the CLEAR installation requirements

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification unsuitable
- circ_quant requires the HISAT2 BAM to be free of unmapped reads; feeding a raw aligner output with unmapped records will break the quantification
- FPM (and read counts) vary with sequencing length/strategy; only FPB-based comparison is robust across 1x101 vs 1x50 vs 2x101 bp datasets

## Evidence

- [readme] -b BAM, --bam BAM     Input mapped reads from HISAT2 in BAM format.: "circ_quant ... Input mapped reads from HISAT2 in BAM format."
- [readme] *hisat_aligned.bam should not contain unmapped reads.*: "hisat_aligned.bam should not contain unmapped reads."
- [readme] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
- [abstract] we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR): "circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR)"
- [discussion] user-friendly due to the application of reliable CIRCexplorer2: "user-friendly due to the application of reliable CIRCexplorer2"
