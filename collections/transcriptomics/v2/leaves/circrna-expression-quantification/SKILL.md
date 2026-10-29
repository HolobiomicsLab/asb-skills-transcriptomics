---
name: circrna-expression-quantification
description: Use when you need to quantify circRNA expression and compare it to cognate
  linear RNA expression from ribo− RNA-seq FASTQ files, e.g. to distinguish tumor
  vs paired-normal circRNA profiles where standard fragment counts fail.
when_to_use_negative:
- 'Input RNA-seq is poly(A)−/ribo− or RNase R-treated: linear RNAs are largely depleted,
  so accurate linear RNA quantification and subsequent CIRCscore evaluation is unsuitable'
- Data is poly(A)+ RNA-seq, in which circRNAs are depleted due to their lack of 3′-end
  poly(A) tails
- You only need circRNA discovery/annotation without circular-vs-linear expression
  comparison (plain CIRCexplorer2 suffices)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline that quantifies circular and linear RNA expression from ribo− RNA-seq
    via clear_quant/circ_quant, emitting FPBcirc, FPBlinear, and CIRCscore
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Upstream circRNA caller (>=2.3.6) whose output (fusion junction realignment,
    circular RNA file) circ_quant consumes; supports TopHat-Fusion, STAR, MapSplice,
    BWA, segemehl
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Read mapper (>=2.0.5) producing the BAM of mapped reads used for linear SJ
    quantification in circ_quant
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:0f46ee5f7bf7d566dbfca79b4a437bada3c02c3e668c776664d741821476f460
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# circrna-expression-quantification

## Summary

Quantify circular and linear RNA expression from ribosomal-RNA depleted (ribo−) RNA-seq using the CIRCexplorer3-CLEAR pipeline, which reports FPBcirc, FPBlinear, and a CIRCscore (FPBcirc/FPBlinear) per circRNA so circular and linear expression at the same gene loci can be directly compared.

## When to use

Use when you need to quantify circRNA expression and compare it to cognate linear RNA expression from ribo− RNA-seq FASTQ files, e.g. to distinguish tumor vs paired-normal circRNA profiles where standard fragment counts fail.

## When NOT to use

- Input RNA-seq is poly(A)−/ribo− or RNase R-treated: linear RNAs are largely depleted, so accurate linear RNA quantification and subsequent CIRCscore evaluation is unsuitable
- Data is poly(A)+ RNA-seq, in which circRNAs are depleted due to their lack of 3′-end poly(A) tails
- You only need circRNA discovery/annotation without circular-vs-linear expression comparison (plain CIRCexplorer2 suffices)

## Inputs

- Ribo− (ribosomal-RNA depleted) paired-end RNA-seq FASTQ files (e.g. 20 HCC and paired normal libraries from GEO accession GSE77509)
- Reference genome FASTA (e.g. hg38.fa)
- HISAT2 genome index (e.g. hg38.hisat_index)
- Bowtie1 index for TopHat-Fusion (e.g. hg38.bowtie_index)
- Annotation GTF file (e.g. annotation.gtf); refFlat annotation for circ_quant mode

## Outputs

- output_dir/quant/quant.txt per sample with circRNA coordinates, geneName, readNumber, FPBcirc, FPBlinear, and CIRCscore
- Cumulative distribution curves of FPBcirc and CIRCscore for condition mixtures (e.g. HCC vs normal)
- Two-tailed unpaired Student's t-test P-values comparing conditions for FPBcirc and CIRCscore (e.g. circscore_ttest_results.csv)

## How to apply

Install CIRCexplorer3-CLEAR from github.com/YangLab/CLEAR with 'python ./setup.py install' (requires CIRCexplorer2 >=2.3.6, HISAT2 >=2.0.5, pysam, pybedtools). For each library run clear_quant with paired mates (-1/-2), the genome FASTA such as hg38 (-g), the HISAT2 index (-i), a bowtie1 index for TopHat-Fusion (-j), and an annotation GTF (-G). Parse output_dir/quant/quant.txt to extract FPBcirc (fragments per billion mapped bases at BSJ sites), FPBlinear (fragments at SJ sites), and CIRCscore = FPBcirc/FPBlinear per circRNA. To compare conditions (e.g. 20 HCC vs paired normal samples from GSE77509), build cumulative distribution curves of FPBcirc and CIRCscore and run two-tailed unpaired Student's t-tests separately; CIRCscore normalizes circRNA expression by the cognate linear RNA background and can separate HCC from normal (P = 3.59e-5) where FPBcirc alone cannot (P = 0.99).

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline that quantifies circular and linear RNA expression from ribo− RNA-seq via clear_quant/circ_quant, emitting FPBcirc, FPBlinear, and CIRCscore) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Upstream circRNA caller (>=2.3.6) whose output (fusion junction realignment, circular RNA file) circ_quant consumes; supports TopHat-Fusion, STAR, MapSplice, BWA, segemehl) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Read mapper (>=2.0.5) producing the BAM of mapped reads used for linear SJ quantification in circ_quant) — https://ccb.jhu.edu/software/hisat2/index.shtml

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt contains all three expression fields populated: FPBcirc, FPBlinear, and CIRCscore for each circRNA row
- CIRCscore values equal FPBcirc divided by FPBlinear (with optional --ratio adjustment, default 1) for each circRNA
- hisat_aligned.bam used by circ_quant should not contain unmapped reads
- Reproducing the HCC vs paired-normal comparison should yield CIRCscore P ≈ 3.59e-5 and FPBcirc P ≈ 0.99 (two-tailed unpaired t-test)
- circRNAs below the FPB threshold (--threshold, default 1) are filtered from linear SJ selection

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification
- No software version numbers stated for CLEAR itself; requires CIRCexplorer2 >=2.3.6 and HISAT2 >=2.0.5
- No data accessions deposited for the RNA-seq comparisons referenced in the article discussion

## Evidence

- [abstract] A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites or to: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites"
- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [methods] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [readme] hisat_aligned.bam should not contain unmapped reads.: "*hisat_aligned.bam should not contain unmapped reads.*"
- [discussion] polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
