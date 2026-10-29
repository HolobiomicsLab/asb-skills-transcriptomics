---
name: circrna-linear-expression-correlation-analysis
description: Use when validating circRNA quantification from ribo− RNA-seq against
  RT-qPCR or existing FPKM/FPM measures, or when reproducing published Pearson correlations
  (e.g. r=0.92, 0.78, 0.72, 1.00, 0.93) for FPB and CIRCscore values.
when_to_use_negative:
- 'Input data are poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear
  RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification
  and subsequent CIRCscore evaluation'
- Input is poly(A)+ RNA-seq data, where circRNAs are depleted and BSJ-based quantification
  of circRNA expression is unreliable
- The goal is circRNA discovery/annotation rather than quantification validation —
  use the base CIRCexplorer2 pipeline instead
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: Quantifies circular and linear RNA expression as FPB values from ribo− RNA-seq
    and computes CIRCscore (FPBcirc/FPBlinear)
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Upstream circRNA detection dependency (>=2.3.6); circ_quant mode accepts CIRCexplorer2
    output plus HISAT2 BAM for quantification
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Maps ribo− RNA-seq reads to produce the BAM used for linear SJ-based quantification
    (>=2.0.5)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:f52aa02dd823ea606a7adff356a870763c702d591a292330dceaa2cbb99df34a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# circrna-linear-expression-correlation-analysis

## Summary

Reproduce the CLEAR pipeline's circular vs linear RNA quantification statistics by computing Pearson correlations between FPBcirc/FPBlinear/CIRCscore values and reference measures (FPKM, FPM, RT-qPCR) on ribo− RNA-seq data, validating the pipeline's quantification against published coefficients.

## When to use

Use when validating circRNA quantification from ribo− RNA-seq against RT-qPCR or existing FPKM/FPM measures, or when reproducing published Pearson correlations (e.g. r=0.92, 0.78, 0.72, 1.00, 0.93) for FPB and CIRCscore values.

## When NOT to use

- Input data are poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation
- Input is poly(A)+ RNA-seq data, where circRNAs are depleted and BSJ-based quantification of circRNA expression is unreliable
- The goal is circRNA discovery/annotation rather than quantification validation — use the base CIRCexplorer2 pipeline instead

## Inputs

- PA1 ribo− paired-end RNA-seq FASTQ files (e.g. GEO GSE75733/GSE73325)
- hg38 genome FASTA (hg38.fa)
- HISAT2 index (hg38.hisat_index)
- TopHat-Fusion Bowtie1 index (hg38.bowtie_index)
- Gene annotation GTF (annotation.gtf)
- Reference quantification table with FPKM, FPM, and RT-qPCR measurements for validated RNAs (supplementary Table S3)

## Outputs

- output_dir/quant/quant.txt with per-circRNA FPBcirc, FPBlinear, and CIRCscore values
- pearson_correlations.csv with computed Pearson correlation coefficients for each comparison

## How to apply

Run the CLEAR pipeline (CIRCexplorer3-CLEAR, requiring CIRCexplorer2 >=2.3.6 and HISAT2 >=2.0.5) via clear_quant on paired-end ribo− RNA-seq FASTQ files with a genome FASTA, HISAT2 and TopHat-Fusion (Bowtie1) indices, and a GTF annotation, producing output_dir/quant/quant.txt with FPBcirc, FPBlinear, and CIRCscore per circRNA. CIRCscore is defined as FPBcirc divided by FPBlinear, enabling direct comparison of circular and linear RNA from the same locus, unlike incomparable FPM vs FPKM units. Extract values for the RT-qPCR-validated RNAs (13 in supplementary Table S3) plus reference FPKM, FPM, and RT-qPCR measurements. Compute Pearson correlation coefficients for each published comparison: FPBlinear vs FPKMlinear (Fig. 2A, r=0.92, under FPBlinear>0 and FPKMlinear>0), FPBlinear vs RT-qPCR (Fig. 2B, r=0.78), FPBcirc vs FPMcirc (Fig. 2C, r=1.00), FPBcirc vs RT-qPCR (Fig. 2D, r=0.72), and CIRCscore vs RT-qPCR circ/linear ratio (Fig. 2F, r=0.93). Accept reproduction when each computed r matches the published value within reasonable numerical tolerance.

## Related tools

- **CIRCexplorer3-CLEAR** (Quantifies circular and linear RNA expression as FPB values from ribo− RNA-seq and computes CIRCscore (FPBcirc/FPBlinear)) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Upstream circRNA detection dependency (>=2.3.6); circ_quant mode accepts CIRCexplorer2 output plus HISAT2 BAM for quantification) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Maps ribo− RNA-seq reads to produce the BAM used for linear SJ-based quantification (>=2.0.5)) — https://ccb.jhu.edu/software/hisat2/index.shtml

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- Pearson r for FPBlinear vs FPKMlinear reproduces 0.92 (restricted to FPBlinear>0 and FPKMlinear>0) within numerical tolerance
- Pearson r for FPBcirc vs FPMcirc reproduces 1.00, confirming unit consistency of the two junction-read normalization schemes
- Pearson r values for FPBlinear vs RT-qPCR (0.78), FPBcirc vs RT-qPCR (0.72), and CIRCscore vs RT-qPCR circ/linear ratio (0.93) match published values
- quant.txt contains the expected schema including FPBcirc, FPBlinear, and CIRCscore columns for every detected circRNA
- CIRCscore column equals FPBcirc/FPBlinear for each circRNA row (invariant check)

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate
- circ_quant's hisat_aligned.bam input should not contain unmapped reads
- Default filter threshold for choosing circRNAs to filter linear SJ is FPB=1; --ratio (default 1) adjusts the circ vs linear comparison — non-default settings change reproduced statistics

## Evidence

- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [intro] Since FPM is unscaled to FPKM, the relative expression levels of most circRNAs are not comparable: "Since FPM is unscaled to FPKM, the relative expression levels of most circRNAs are not comparable to those of their cognate linear RNAs when analyzing RNA-seq datasets"
- [methods] clear_quant usage with FASTQ, genome, HISAT and Bowtie1 indices and GTF: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [discussion] CIRCscore limitation for poly(A)−/ribo− or RNase R-treated datasets: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
- [readme] Output table fields FPBcirc, FPBlinear, CIRCscore in quant.txt: "| FPBcirc     | Expression of circRNA                 | | FPBlinear   | Expression of cognate linear RNA      | | CIRCscore   | Relative expression of circRNA        |"
