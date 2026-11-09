---
name: cumulative-distribution-analysis
description: Use when comparing two sample groups' circRNA expression distributions
  quantified per-circRNA (e.g. FPBcirc or CIRCscore from CLEAR quant.txt outputs)
  to decide whether one metric better discriminates the groups.
when_to_use_negative:
- Input data are poly(A)-/ribo- or RNase R-treated RNA-seq datasets, where linear
  RNAs are depleted and CIRCscore cannot be accurately computed
- The metric of interest is not a per-circRNA continuous quantity (e.g. already-aggregated
  per-sample summaries or categorical annotations)
- Only raw junction read counts are available without FPB normalization, since unnormalized
  counts confound library-size differences
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline that quantifies FPBcirc, FPBlinear, and CIRCscore per circRNA from
    ribo- RNA-seq, providing the per-circRNA values used to build cumulative distributions
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Underlying circRNA detection engine (>=2.3.6) that CLEAR's clear_quant/circ_quant
    rely on to identify back-splicing junctions
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligner (>=2.0.5) used inside clear_quant to map reads to the genome, producing
    the BAM used for quantification
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:0ab63312dd3329b1679c3806887da44a132b9e983a4e858b64bb162acecf4f9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cumulative-distribution-analysis

## Summary

Builds cumulative distribution curves of circRNA expression metrics (FPBcirc or CIRCscore) across sample groups and applies two-tailed unpaired Student's t-tests to test whether a normalization metric separates conditions, e.g. HCC vs paired-normal liver samples from ribo- RNA-seq quantified with CIRCexplorer3-CLEAR.

## When to use

Use when comparing two sample groups' circRNA expression distributions quantified per-circRNA (e.g. FPBcirc or CIRCscore from CLEAR quant.txt outputs) to decide whether one metric better discriminates the groups.

## When NOT to use

- Input data are poly(A)-/ribo- or RNase R-treated RNA-seq datasets, where linear RNAs are depleted and CIRCscore cannot be accurately computed
- The metric of interest is not a per-circRNA continuous quantity (e.g. already-aggregated per-sample summaries or categorical annotations)
- Only raw junction read counts are available without FPB normalization, since unnormalized counts confound library-size differences

## Inputs

- Paired-end ribo- RNA-seq FASTQ files per sample (e.g. GSE77509 HCC and paired-normal libraries)
- hg38 genome FASTA, hg38 HISAT2 index, hg38 bowtie1 index, and annotation GTF
- clear_quant output quant.txt per sample with FPBcirc, FPBlinear, and CIRCscore columns

## Outputs

- Cumulative distribution curves of FPBcirc and CIRCscore for HCC mixture vs normal mixture
- Results table (e.g. circscore_ttest_results.csv) with both P-values and cumulative distribution data

## How to apply

Run clear_quant per library on paired-end FASTQ files (e.g. the 20 HCC and 20 paired-normal ribo- samples from GSE77509) and parse output_dir/quant/quant.txt to extract FPBcirc, FPBlinear, and CIRCscore per circRNA. Build cumulative distribution curves of each metric for the HCC mixture versus the normal mixture, following the paper's Figure 6A/6B comparison. For each metric separately, perform a two-tailed unpaired Student's t-test (e.g. scipy.stats.ttest_ind) between the HCC and normal values. The rationale is that FPBcirc alone reflects raw back-splice junction support and may show no group difference, whereas CIRCscore = FPBcirc/FPBlinear normalizes by the cognate linear RNA background and can reveal significant group separation. Record both P-values and the cumulative distribution data in a results table for reporting.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline that quantifies FPBcirc, FPBlinear, and CIRCscore per circRNA from ribo- RNA-seq, providing the per-circRNA values used to build cumulative distributions) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Underlying circRNA detection engine (>=2.3.6) that CLEAR's clear_quant/circ_quant rely on to identify back-splicing junctions) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligner (>=2.0.5) used inside clear_quant to map reads to the genome, producing the BAM used for quantification) — https://ccb.jhu.edu/software/hisat2/index.shtml

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir; scipy.stats.ttest_ind(hcc_circscore, normal_circscore)
```

## Evaluation signals

- Two-tailed unpaired Student's t-test on CIRCscore yields a significant P-value (paper reports P = 3.59e-5 for HCC vs normal), while FPBcirc yields a non-significant P-value (paper reports P = 0.99)
- The cumulative distribution curves for CIRCscore visibly separate HCC from normal samples, showing circRNAs generally lowly expressed in HCC
- quant.txt files exist at output_dir/quant/quant.txt for all 40 libraries and contain populated FPBcirc, FPBlinear, and CIRCscore fields
- Results table includes both P-values and the underlying cumulative distribution data

## Limitations

- CIRCscore cannot be applied to poly(A)-/ribo- or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate
- FPBcirc alone may fail to distinguish conditions (P = 0.99 in the HCC example), so relying on it without CIRCscore normalization can mask real differences
- CIRCscore default behavior assumes --ratio 1 for circ/linear comparison adjustment and a default FPB threshold of 1 for choosing circRNAs to filter linear SJ

## Evidence

- [results] CircRNAs are generally lowly expressed in almost all HCC samples when CIRCscore normalizes circRNA expression with the cognate linear RNA background (Figure 6B, P = 3.59 × 10−5): [evidence span withheld — anchor status: not_found]
- [results] whereas FPBcirc fails to distinguish HCC from paired normal samples (Figure 6A, P = 0.99): [evidence span withheld — anchor status: not_found]
- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using"
- [readme] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
- [discussion] polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
