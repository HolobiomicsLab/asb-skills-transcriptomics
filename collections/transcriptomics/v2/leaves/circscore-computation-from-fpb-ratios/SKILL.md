---
name: circscore-computation-from-fpb-ratios
description: Use when ribo− RNA-seq data has been mapped (HISAT2 + TopHat-Fusion),
  circRNAs annotated via CIRCexplorer2, and FPBcirc/FPBlinear values are available
  per gene locus and a relative circRNA expression level is needed.
when_to_use_negative:
- 'Input is poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs
  are largely depleted, which is unsuitable for accurate linear RNA quantification
  and subsequent CIRCscore evaluation'
- Input is poly(A)+ RNA-seq data, where circRNAs are depleted and BSJ fragments are
  not represented
- The goal is circRNA detection/annotation itself (use CIRCexplorer2); this skill
  starts from mapped reads/CIRCexplorer2 output and computes relative quantification
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Provides clear_quant and circ_quant commands that normalize fragments to FPBcirc
    and FPBlinear and compute CIRCscore
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Annotates circRNAs from the aligned BAM and supplies the circular RNA input
    file consumed by circ_quant (>=2.3.6)
  repo: https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- name: HISAT2
  role: Maps ribo− paired-end reads to the hg38 genome; mapped BAM (unmapped reads
    discarded) feeds FPB quantification
- name: pysam
  role: Python package dependency used by the CLEAR pipeline to process the BAM alignments
- name: pybedtools
  role: Python package dependency used by the CLEAR pipeline for interval operations
    during quantification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:105cccb71f02b9bc76636c3bcef3f1cca3256259331667c5f8303ba0b14d1b33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# circscore-computation-from-fpb-ratios

## Summary

Computation of CIRCscore, the relative circular RNA expression metric defined as FPBcirc divided by FPBlinear, within the CIRCexplorer3-CLEAR pipeline. Because both circular and linear RNA expression from ribosomal-RNA depleted RNA-seq are normalized as fragments per billion mapped bases (FPB), their ratio directly compares circRNA abundance against its cognate linear RNA background.

## When to use

Use when ribo− RNA-seq data has been mapped (HISAT2 + TopHat-Fusion), circRNAs annotated via CIRCexplorer2, and FPBcirc/FPBlinear values are available per gene locus and a relative circRNA expression level is needed.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation
- Input is poly(A)+ RNA-seq data, where circRNAs are depleted and BSJ fragments are not represented
- The goal is circRNA detection/annotation itself (use CIRCexplorer2); this skill starts from mapped reads/CIRCexplorer2 output and computes relative quantification

## Inputs

- Paired-end ribosomal-RNA depleted RNA-seq FASTQ files (mate_1 and mate_2, e.g. from GEO GSE75733/GSE73325)
- Genome FASTA (e.g. hg38.fa)
- Prebuilt HISAT2 index for the genome
- Bowtie1 index for TopHat-Fusion
- Annotation GTF file
- Alternative inputs for circ_quant: CIRCexplorer2 circRNA output file, HISAT2-aligned BAM without unmapped reads, refFlat gene annotation

## Outputs

- output_dir/quant/quant.txt quantification table with chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore columns
- Per-locus CIRCscore values (FPBcirc/FPBlinear) ranking circRNAs by relative expression over their cognate linear RNA background

## How to apply

Run clear_quant on paired-end ribo− FASTQ files with the hg38 genome FASTA, HISAT2 index, Bowtie1 index, and annotation GTF to quantify fragments mapped to circRNA-specific BSJ sites and linear RNA-specific SJ sites as FPB values (default 5 threads). Alternatively run circ_quant starting from a CIRCexplorer2 output file plus the HISAT2-aligned BAM (which must not contain unmapped reads) and a refFlat gene annotation. CircRNAs are chosen to filter linear SJ sites using the default FPB threshold of 1, and the circ-to-linear comparison is adjusted with --ratio (default 1); optionally pass -l to consider all reads' length. For each locus, compute CIRCscore = FPBcirc / FPBlinear, so that highly-expressed circRNAs with low cognate linear RNA expression background stand out with high scores. Write results to output_dir/quant/quant.txt with fields including chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore.

## Related tools

- **CIRCexplorer3-CLEAR** (Provides clear_quant and circ_quant commands that normalize fragments to FPBcirc and FPBlinear and compute CIRCscore) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Annotates circRNAs from the aligned BAM and supplies the circular RNA input file consumed by circ_quant (>=2.3.6)) — https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- **HISAT2** (Maps ribo− paired-end reads to the hg38 genome; mapped BAM (unmapped reads discarded) feeds FPB quantification)
- **pysam** (Python package dependency used by the CLEAR pipeline to process the BAM alignments)
- **pybedtools** (Python package dependency used by the CLEAR pipeline for interval operations during quantification)

## Examples

```
circ_quant -c CIRCexplorer2_output.txt -b hisat_aligned.bam -t -r annotation.refFlat -o quant.txt
```

## Evaluation signals

- output_dir/quant/quant.txt exists and contains the schema fields chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore
- For every locus, CIRCscore numerically equals FPBcirc divided by FPBlinear
- FPBcirc and FPBlinear are normalized fragments-per-billion values computed independently from BSJ-mapped and SJ-mapped fragments respectively
- hisat_aligned.bam contains no unmapped reads (a stated precondition for circ_quant)
- Default filtering behavior: only circRNAs meeting the FPB threshold of 1 are used to filter linear SJ, and the circ-to-linear ratio defaults to 1 unless overridden

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because depleted linear RNA makes FPBlinear inaccurate
- The default FPB threshold (1) and ratio (1) for choosing circRNAs to filter linear SJ may need adjustment via --threshold/--ratio for other comparisons
- When starting from fastq, TopHat-Fusion with the Bowtie1 index is required for back-splice detection; alignment choices (hg38 HISAT2/Bowtie1 indices) must match the reference used for annotation

## Evidence

- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using"
- [abstract] A new quantitation parameter, fragments per billion mapped bases (FPB): "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites or to"
- [methods] usage: circ_quant [-h] -c CIRC -b BAM -r REF [--threshold THRESHOLD] [--ratio RATIO] [-l] [-t] [-o OUTPUT]: "usage: circ_quant [-h] -c CIRC -b BAM -r REF [--threshold THRESHOLD]
                     [--ratio RATIO] [-l] [-t] [-o OUTPUT]"
- [methods] Threshold of FPB for choose circRNAs to filter linear SJ.[default: 1]: "Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]"
- [readme] hisat_aligned.bam should not contain unmapped reads.: "*hisat_aligned.bam should not contain unmapped reads.*"
- [readme] circ_quant -c CIRCexplorer2_output.txt -b hisat_aligned.bam -t -r annotation.refFlat -o quant.txt: "circ_quant -c CIRCexplorer2_output.txt -b hisat_aligned.bam -t -r annotation.refFlat -o quant.txt"
