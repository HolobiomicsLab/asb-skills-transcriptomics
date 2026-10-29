---
name: circular-linear-rna-expression-comparison
description: Use when quantifying circRNA expression from ribo− paired-end RNA-seq
  FASTQs and comparing it against cognate linear RNA expression at the same gene loci
  (CIRCscore = FPBcirc/FPBlinear).
when_to_use_negative:
- Input RNA-seq is poly(A)−/ribo− or RNase R-treated, because polyadenylated linear
  RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification
  and subsequent CIRCscore evaluation
- Data are poly(A)+ RNA-seq, where circRNAs are depleted due to their lack of 3′-end
  poly(A) tails
- The analysis only needs circRNA detection without a cognate linear RNA expression
  background (plain CIRCexplorer2 suffices)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0099
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline (clear_quant and circ_quant) that quantifies FPBcirc, FPBlinear,
    and CIRCscore from ribo− RNA-seq
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Provides circRNA annotation from aligned BAM (requires >=2.3.6); its output
    is the -c input to circ_quant
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligns ribo− paired-end reads to the hg38 genome producing the mapped BAM
    (requires >=2.0.5; unmapped reads discarded)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: StringTie
  role: Installation requirement of the CLEAR pipeline (>1.3.6)
  repo: https://ccb.jhu.edu/software/stringtie
- name: pysam
  role: Python package dependency (>=0.8.4) used by CLEAR for BAM handling
  repo: http://pysam.readthedocs.org/en/latest/
- name: pybedtools
  role: Python package dependency used by CLEAR for interval operations
  repo: http://daler.github.io/pybedtools/
- name: TopHat-Fusion
  role: Fusion junction realignment of HISAT2-unmapped reads to detect back-splicing
    junction fragments, using the Bowtie1 index
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:e100cfd51c00e8aaee20fe88b3f0e4874938fa81a3d8477c992e493287b6e946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - CLEAR
  - DCC/circtools
  - HISAT2
  - StringTie
  - TopHat-Fusion
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# circular-linear-rna-expression-comparison

## Summary

A CIRCexplorer3-CLEAR pipeline method that quantifies circular and linear RNA expression from ribosomal-RNA depleted RNA-seq using the FPB (fragments per billion mapped bases) metric computed separately at BSJ and SJ sites, and directly compares the two by dividing FPBcirc by FPBlinear to yield a CIRCscore.

## When to use

Use when quantifying circRNA expression from ribo− paired-end RNA-seq FASTQs and comparing it against cognate linear RNA expression at the same gene loci (CIRCscore = FPBcirc/FPBlinear).

## When NOT to use

- Input RNA-seq is poly(A)−/ribo− or RNase R-treated, because polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation
- Data are poly(A)+ RNA-seq, where circRNAs are depleted due to their lack of 3′-end poly(A) tails
- The analysis only needs circRNA detection without a cognate linear RNA expression background (plain CIRCexplorer2 suffices)

## Inputs

- Ribosomal-RNA depleted paired-end RNA-seq FASTQ files (mate_1 and mate_2)
- hg38 genome FASTA file
- Prebuilt hg38 HISAT2 index
- Prebuilt hg38 Bowtie1 (TopHat-Fusion) index
- Gene annotation GTF file
- CIRCexplorer2 circRNA output file (for circ_quant mode)
- HISAT2-aligned BAM free of unmapped reads (for circ_quant mode)
- refFlat-format gene annotation file (for circ_quant mode)

## Outputs

- output_dir/quant/quant.txt quantification table with chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore fields
- Per-locus CIRCscore values (FPBcirc/FPBlinear) for direct circular-vs-linear comparison

## How to apply

Run clear_quant on ribo− paired-end FASTQ files with the hg38 genome FASTA, a prebuilt HISAT2 (>=2.0.5) index, a Bowtie1 (TopHat-Fusion) index, and an annotation GTF, using the default 5 threads; HISAT2-aligned BAMs must have unmapped reads removed before TopHat-Fusion fusion-junction realignment and CIRCexplorer2 (>=2.3.6) circRNA annotation. Then run circ_quant with the CIRCexplorer2 output (-c), the HISAT2-aligned BAM (-b), and a refFlat gene annotation (-r), keeping the default FPB threshold of 1 for filtering linear SJ and the default circ-to-linear ratio of 1 (optionally -l to consider all reads' lengths). The pipeline normalizes fragments mapped to circRNA-specific BSJ sites and linear RNA-specific SJ sites individually into FPBcirc and FPBlinear, and computes CIRCscore = FPBcirc/FPBlinear so relative circRNA expression uses linear RNA expression as the background. This fixes the incomparability of FPM (circRNA) vs FPKM (linear) used by earlier approaches, and CLEAR is reported as more reliable than DCC/circtools (e.g., ~126 vs ~36 BSJ fragments for circCAMSAP1-1), with economic memory usage.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline (clear_quant and circ_quant) that quantifies FPBcirc, FPBlinear, and CIRCscore from ribo− RNA-seq) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Provides circRNA annotation from aligned BAM (requires >=2.3.6); its output is the -c input to circ_quant) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligns ribo− paired-end reads to the hg38 genome producing the mapped BAM (requires >=2.0.5; unmapped reads discarded)) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **StringTie** (Installation requirement of the CLEAR pipeline (>1.3.6)) — https://ccb.jhu.edu/software/stringtie
- **pysam** (Python package dependency (>=0.8.4) used by CLEAR for BAM handling) — http://pysam.readthedocs.org/en/latest/
- **pybedtools** (Python package dependency used by CLEAR for interval operations) — http://daler.github.io/pybedtools/
- **TopHat-Fusion** (Fusion junction realignment of HISAT2-unmapped reads to detect back-splicing junction fragments, using the Bowtie1 index)

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt exists with the schema fields chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore
- CIRCscore equals FPBcirc/FPBlinear for every locus in quant.txt
- HISAT2-aligned BAM used as circ_quant -b input contains no unmapped reads
- Reproduction of Figure 3A values for circCAMSAP1-1: CIRCscore ~3.65 (vs RT-qPCR ~3.82) with ~126 BSJ fragments, versus circtools CLR ~0.80 with ~36 fragments
- In 12 ENCODE tissues, the fraction of circRNAs with FPBcirc ≥ 1 that have CIRCscore ≥ 1 falls in the ~10%–20% range reported in Figure 5A

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification unsuitable
- circtools DCC calls fewer BSJ-mapped fragments than CLEAR-embedded CIRCexplorer2 (36 vs 126 for circCAMSAP1-1), so cross-tool CLR values are not equivalent to CIRCscore
- hisat_aligned.bam must not contain unmapped reads, and default filtering (FPB threshold 1, circ-to-linear ratio 1) constrains which loci are compared

## Evidence

- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using"
- [methods] clear_quant usage: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [methods] FPB threshold default: "Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]"
- [readme] BAM requirement: "*hisat_aligned.bam should not contain unmapped reads.*"
- [discussion] CIRCscore limitation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
