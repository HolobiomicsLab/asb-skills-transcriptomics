---
name: ribo-minus-rnaseq-processing
description: Use when ribo− RNA-seq FASTQ files must be processed into per-circRNA
  FPBcirc, FPBlinear, and CIRCscore values, e.g. to compare circRNA expression between
  tumor and paired-normal samples such as the 20 HCC/paired-normal libraries from
  GSE77509.
when_to_use_negative:
- 'Input is poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs
  are largely depleted, which is unsuitable for accurate linear RNA quantification
  and subsequent CIRCscore evaluation'
- Input is poly(A)+ RNA-seq data, in which circRNAs are depleted because they lack
  3′ poly(A) tails
- The goal is circRNA detection alone from CIRCexplorer2 output without circular/linear
  expression comparison — then CIRCexplorer2 by itself suffices and circ_quant/CIRCscore
  normalization is unnecessary
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: CIRCexplorer3-CLEAR
  role: 'Core pipeline: quantifies circular and linear RNA expression from ribo− RNA-seq
    and computes FPBcirc, FPBlinear, and CIRCscore'
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Underlying circRNA identification engine (>=2.3.6) whose output can feed circ_quant
    when starting from CIRCexplorer2 output files
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Maps ribo− RNA-seq reads to the genome in BAM format for quantification (>=2.0.5)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:6d7f0842d023020e6947a9352a06a86b7388ac6d67ed9ac094dce1855c44b93c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ribo-minus-rnaseq-processing

## Summary

Process ribosomal-RNA depleted (ribo−) RNA-seq data with the CIRCexplorer3-CLEAR pipeline to quantify circular and linear RNA expression via FPB (fragments per billion mapped bases) and compute a CIRCscore (FPBcirc/FPBlinear) that allows direct comparison of circRNA expression against its cognate linear RNA background.

## When to use

Use when ribo− RNA-seq FASTQ files must be processed into per-circRNA FPBcirc, FPBlinear, and CIRCscore values, e.g. to compare circRNA expression between tumor and paired-normal samples such as the 20 HCC/paired-normal libraries from GSE77509.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation
- Input is poly(A)+ RNA-seq data, in which circRNAs are depleted because they lack 3′ poly(A) tails
- The goal is circRNA detection alone from CIRCexplorer2 output without circular/linear expression comparison — then CIRCexplorer2 by itself suffices and circ_quant/CIRCscore normalization is unnecessary

## Inputs

- Paired-end ribo− RNA-seq FASTQ files (mate 1 and mate 2), e.g. 20 HCC and 20 paired-normal libraries from GEO accession GSE77509
- hg38 genome FASTA file (hg38.fa)
- hg38 HISAT2 index files
- hg38 bowtie1 index files for TopHat-Fusion
- Gene annotation GTF file (and refFlat-format annotation when starting from CIRCexplorer2 output with circ_quant)

## Outputs

- Per-sample output_dir/quant/quant.txt with fields including chrom, start, end, strand, exonSizes, readNumber, FPBcirc, FPBlinear, and CIRCscore per circRNA
- Cumulative distribution curves of FPBcirc and CIRCscore for sample groups (e.g. HCC mixture vs normal mixture)
- Results table (e.g. circscore_ttest_results.csv) containing two-tailed unpaired Student's t-test P-values and cumulative distribution data

## How to apply

Install CLEAR from github.com/YangLab/CLEAR with 'python ./setup.py install', ensuring CIRCexplainer2 (>=2.3.6), HISAT2 (>=2.0.5), pysam, and pybedtools are available. For each library, run clear_quant with paired mates (-1/-2), the hg38 genome FASTA (-g), the hg38 HISAT2 index (-i), the hg38 bowtie1 index used for TopHat-Fusion (-j), and an annotation GTF (-G), writing to a per-sample output directory; CLEAR maps reads with HISAT2 and performs fusion junction realignment. Parse output_dir/quant/quant.txt to extract FPBcirc (fragments mapped to circRNA-specific BSJ sites), FPBlinear (fragments mapped to linear RNA-specific SJ sites), and CIRCscore (FPBcirc/FPBlinear) per circRNA. CIRCscore uses linear RNA expression as the background so relative circRNA expression is comparable across samples, whereas raw FPBcirc alone often is not; optionally adjust the circ-vs-linear comparison with the circ_quant --ratio parameter (default 1) and the FPB threshold (default 1) for choosing circRNAs to filter linear SJ. Downstream, build cumulative distributions per group (e.g. HCC mixture vs normal mixture) and test differences with a two-tailed unpaired Student's t-test (scipy.stats.ttest_ind).

## Related tools

- **CIRCexplorer3-CLEAR** (Core pipeline: quantifies circular and linear RNA expression from ribo− RNA-seq and computes FPBcirc, FPBlinear, and CIRCscore) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Underlying circRNA identification engine (>=2.3.6) whose output can feed circ_quant when starting from CIRCexplorer2 output files) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Maps ribo− RNA-seq reads to the genome in BAM format for quantification (>=2.0.5)) — https://ccb.jhu.edu/software/hisat2/index.shtml

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt output exists at output_dir/quant/quant.txt for each sample and contains populated FPBcirc, FPBlinear, and CIRCscore columns
- CIRCscore values equal FPBcirc divided by FPBlinear for each circRNA (ratio adjustment default 1)
- BAM input to circ_quant contains no unmapped reads ('hisat_aligned.bam should not contain unmapped reads')
- Group-level expectation from the source card: CIRCscore separates HCC from paired-normal samples (P = 3.59e-5), whereas FPBcirc does not (P = 0.99); a run reproducing this contrast supports correct application
-  circRNAs with low FPBcirc (below the --threshold default of 1) are filtered from linear SJ selection as specified

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate
- CIRCscore depends on accurate cognate linear RNA quantification, so it requires ribo− RNA-seq with sufficient linear SJ coverage
- The circ_quant --threshold (default 1) and --ratio (default 1) parameters affect which linear SJs are used and how circ-vs-linear comparison is adjusted; non-default choices must be documented

## Evidence

- [abstract] we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR): "we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR)"
- [abstract] CIRCscore computed as FPBcirc/FPBlinear: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [readme] clear_quant usage from README: "usage: clear_quant [-h] -1 M1 [-2 M2] -g GENOME -i HISAT -j BOWTIE1 -G GTF"
- [readme] clear_quant example command: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
- [methods] FPB threshold filter step: "Threshold of FPB for choose circRNAs to filter linear SJ.[default: 1]"
- [discussion] CIRCscore limitation on RNase R datasets: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
