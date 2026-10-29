---
name: fpb-expression-normalization
description: Use when quantifying circRNA and cognate linear RNA expression from ribosomal-RNA
  depleted RNA-seq so both can be directly compared on a common scale, or when reproducing
  FPB/CIRCscore-based correlation or tissue-enrichment analyses.
when_to_use_negative:
- Input is poly(A)-/ribo- or RNase R-treated RNA-seq, where polyadenylated linear
  RNAs are largely depleted, making accurate linear RNA quantification and CIRCscore
  evaluation unsuitable
- Input RNA-seq is poly(A)+ enriched, where circRNAs are depleted due to their lack
  of 3' poly(A) tails
- Goal is de novo circRNA discovery/annotation rather than expression quantification
  and circ-vs-linear comparison
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline that normalizes fragments mapped to BSJ and SJ sites into FPB values
    and computes CIRCscore via clear_quant/circ_quant
  repo: https://github.com/YangLab/CLEAR
- name: HISAT2
  role: Initial read alignment to the hg38 genome producing the mapped BAM used for
    FPB quantification
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: CIRCexplorer2
  role: Annotates circRNAs from TopHat-Fusion fusion-junction realignment; provides
    circRNA calls consumed by circ_quant (>=2.3.6 required)
  repo: https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- name: StringTie
  role: Installation requirement of the CLEAR pipeline (>1.3.6)
  repo: https://ccb.jhu.edu/software/stringtie
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:ec7ece74feb84a391c1c7cf8b31fdcf7a48610ef6574b7877b851ae4e3a5b261
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# fpb-expression-normalization

## Summary

Normalize RNA-seq fragments mapped individually to circRNA-specific back-splicing junction (BSJ) sites and linear RNA-specific splicing junction (SJ) sites into FPB (fragments per billion mapped bases) values, then divide FPBcirc by FPBlinear to compute a CIRCscore for direct circular-vs-linear expression comparison using CIRCexplorer3-CLEAR.

## When to use

Use when quantifying circRNA and cognate linear RNA expression from ribosomal-RNA depleted RNA-seq so both can be directly compared on a common scale, or when reproducing FPB/CIRCscore-based correlation or tissue-enrichment analyses.

## When NOT to use

- Input is poly(A)-/ribo- or RNase R-treated RNA-seq, where polyadenylated linear RNAs are largely depleted, making accurate linear RNA quantification and CIRCscore evaluation unsuitable
- Input RNA-seq is poly(A)+ enriched, where circRNAs are depleted due to their lack of 3' poly(A) tails
- Goal is de novo circRNA discovery/annotation rather than expression quantification and circ-vs-linear comparison

## Inputs

- Ribosomal-RNA depleted paired-end RNA-seq FASTQ files (mate_1 and mate_2)
- hg38 genome FASTA (hg38.fa)
- Prebuilt hg38 HISAT2 index
- hg38 Bowtie1 index for TopHat-Fusion
- Annotation GTF file
- CIRCexplorer2 circRNA output file (for circ_quant entry point)
- HISAT2-aligned BAM without unmapped reads (for circ_quant entry point)
- refFlat-format gene annotation (for circ_quant entry point)

## Outputs

- quant.txt quantification table (output_dir/quant/quant.txt) with chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore per circRNA
- circRNA_quant.txt from circ_quant (default output filename)
- Per-circRNA CIRCscore values (FPBcirc/FPBlinear) enabling circ-vs-linear comparison

## How to apply

Run clear_quant on ribo- paired-end FASTQ files with the hg38 genome FASTA, HISAT2 index, TopHat-Fusion (Bowtie1) index, and annotation GTF; CLEAR aligns reads with HISAT2 (>=2.0.5), realigns unmapped reads with TopHat-Fusion via CIRCexplorer2 (>=2.3.6), and quantifies fragments mapped to BSJ and SJ sites individually, normalizing each to FPB. Alternatively, start from CIRCexplorer2 circRNA calls with circ_quant using the HISAT2 BAM (without unmapped reads), a refFlat annotation, and defaults --threshold 1 (FPB cutoff for filtering linear SJ) and --ratio 1 (circ-vs-linear comparison adjustment); optionally pass -l to consider all reads' lengths. Parse output_dir/quant/quant.txt for readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore per locus, where CIRCscore equals FPBcirc/FPBlinear. Filter circRNAs with FPBcirc >= 1 and classify those with CIRCscore >= 1 as expressed at comparable or higher level than their cognate linear RNA.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline that normalizes fragments mapped to BSJ and SJ sites into FPB values and computes CIRCscore via clear_quant/circ_quant) — https://github.com/YangLab/CLEAR
- **HISAT2** (Initial read alignment to the hg38 genome producing the mapped BAM used for FPB quantification) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **CIRCexplorer2** (Annotates circRNAs from TopHat-Fusion fusion-junction realignment; provides circRNA calls consumed by circ_quant (>=2.3.6 required)) — https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- **StringTie** (Installation requirement of the CLEAR pipeline (>1.3.6)) — https://ccb.jhu.edu/software/stringtie

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt exists with schema fields chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore
- For each locus, CIRCscore equals FPBcirc divided by FPBlinear
- FPBlinear correlates with FPKMlinear with Pearson r ≈ 0.92 in PA1 cells (both > 0), and FPBcirc correlates with FPMcirc with r ≈ 1.00
- Median FPB is largely unchanged across virtual sequencing lengths (2x101, 1x101, 1x50 bp) while median FPM deviates with read length
- Among circRNAs with FPBcirc >= 1, the fraction with CIRCscore >= 1 falls in the ~10%–20% range reported across the 12 ENCODE tissues

## Limitations

- CIRCscore cannot be applied to poly(A)-/ribo- or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, unsuitable for accurate linear RNA quantification
- hisat_aligned.bam input to circ_quant must not contain unmapped reads
- FPM used for circRNAs is unscaled to FPKM used for linear RNAs, so pre-CIRCscore circ-vs-linear comparisons are incomparable

## Evidence

- [abstract] A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites or to: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites or to"
- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [methods] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [readme] Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]: "Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]"
- [discussion] polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
