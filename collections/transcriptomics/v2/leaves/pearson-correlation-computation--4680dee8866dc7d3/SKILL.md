---
name: pearson-correlation-computation
description: Use when validating that CLEAR-derived FPB and CIRCscore values from
  ribo− paired-end RNA-seq reproduce published Pearson correlations against FPKM/FPM
  or RT-qPCR measurements.
when_to_use_negative:
- 'Input RNA-seq is poly(A)−/ribo− or RNase R-treated: polyadenylated linear RNAs
  are largely depleted, which is unsuitable for accurate linear RNA quantification
  and subsequent CIRCscore evaluation'
- Data comes from poly(A)+ RNA-seq, where circRNAs are depleted and FPB-based circRNA
  quantification from BSJ fragments is not the appropriate frame
- The comparison quantities are not expression measurements (e.g. already-computed
  correlation tables or unrelated omics matrices), since FPB/FPM/FPKM scaling assumptions
  do not apply
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: Produces the FPBcirc, FPBlinear, and CIRCscore values in quant.txt that are
    correlated against FPKM/FPM and RT-qPCR references
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Required dependency (>=2.3.6) for back-splicing junction detection feeding
    CLEAR's circ_quant quantification
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Read alignment (>=2.0.5) producing the mapped BAM from which CLEAR computes
    SJ-based FPBlinear values
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:abf0d53d1c351f44552a97b9b32a6d41be0e5bdd789e02805ffbe714e243072e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pearson-correlation-computation

## Summary

Compute and validate Pearson correlation coefficients between CIRCexplorer3-CLEAR quantification outputs (FPBcirc, FPBlinear, CIRCscore from quant.txt) and reference measures (FPKMlinear, FPMcirc, RT-qPCR) on PA1 ribo− RNA-seq data, reproducing the paper's Fig. 2 correlations (r = 0.92, 0.78, 0.72, 1.00, 0.93).

## When to use

Use when validating that CLEAR-derived FPB and CIRCscore values from ribo− paired-end RNA-seq reproduce published Pearson correlations against FPKM/FPM or RT-qPCR measurements.

## When NOT to use

- Input RNA-seq is poly(A)−/ribo− or RNase R-treated: polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation
- Data comes from poly(A)+ RNA-seq, where circRNAs are depleted and FPB-based circRNA quantification from BSJ fragments is not the appropriate frame
- The comparison quantities are not expression measurements (e.g. already-computed correlation tables or unrelated omics matrices), since FPB/FPM/FPKM scaling assumptions do not apply

## Inputs

- PA1 ribo− paired-end RNA-seq FASTQ files (GEO accessions GSE75733/GSE73325)
- hg38 genome FASTA (hg38.fa)
- HISAT2 index (hg38.hisat_index) and Bowtie1/TopHat-Fusion index (hg38.bowtie_index)
- Annotation GTF file (annotation.gtf)
- Supplementary Table S3 listing the 13 RT-qPCR-validated RNAs with FPKM, FPM, and RT-qPCR measurements

## Outputs

- quant.txt (output_dir/quant/quant.txt) with FPBcirc, FPBlinear, and CIRCscore columns per circRNA
- pearson_correlations.csv with computed Pearson coefficients per comparison
- Validated correlation values expected to match r = 0.92, 0.78, 0.72, 1.00, and 0.93

## How to apply

Download PA1 ribo− paired-end FASTQ files and reference annotation, then run clear_quant (requires CIRCexplorer2 >=2.3.6 and HISAT2 >=2.0.5) to obtain output_dir/quant/quant.txt containing FPBcirc, FPBlinear, and CIRCscore per circRNA. The rationale is that FPB normalizes RNA-seq fragments mapped to circRNA-specific BSJ sites or linear RNA-specific SJ sites individually, putting circular and linear expression on a directly comparable scale — unlike FPM (circRNA) vs FPKM (linear), which are incomparable because FPM is unscaled to FPKM. Extract FPBcirc, FPBlinear, and CIRCscore for the 13 RT-qPCR-validated RNAs (supplementary Table S3) plus the reference FPKM, FPM, and RT-qPCR values. For the FPBlinear-vs-FPKMlinear comparison, restrict to conditions where both FPBlinear and FPKMlinear > 0. Compute Pearson r for each of the five comparisons — FPBlinear vs FPKMlinear (Fig. 2A), FPBlinear vs RT-qPCR (Fig. 2B), FPBcirc vs FPMcirc (Fig. 2C), FPBcirc vs RT-qPCR (Fig. 2D), and CIRCscore vs RT-qPCR circ/linear ratio (Fig. 2F) — and write the coefficients to pearson_correlations.csv.

## Related tools

- **CIRCexplorer3-CLEAR** (Produces the FPBcirc, FPBlinear, and CIRCscore values in quant.txt that are correlated against FPKM/FPM and RT-qPCR references) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Required dependency (>=2.3.6) for back-splicing junction detection feeding CLEAR's circ_quant quantification) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Read alignment (>=2.0.5) producing the mapped BAM from which CLEAR computes SJ-based FPBlinear values) — https://ccb.jhu.edu/software/hisat2/index.shtml

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- Computed FPBlinear-vs-FPKMlinear Pearson r matches the published 0.92 within reasonable numerical tolerance, using only circRNAs where both FPBlinear and FPKMlinear > 0
- All five reproduced coefficients match the published values (r = 0.92, 0.78, 0.72, 1.00, 0.93) within tolerance
- quant.txt contains the expected FPBcirc, FPBlinear, and CIRCscore fields for every called circRNA
- Pearson r for FPBcirc vs FPMcirc equals 1.00, confirming identical fragment-normalization behavior
- pearson_correlations.csv is written with one coefficient per specified comparison

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate
- FPM and FPKM are on unscaled bases, so circRNA FPM values cannot be directly compared to linear FPKM without the FPB/CIRCscore normalization
- Correlations were established on PA1 ribo− cells with 13 RT-qPCR-validated RNAs; transfer to other cell types or datasets is not demonstrated here

## Evidence

- [intro] Since FPM is unscaled to FPKM, the relative expression levels of most circRNAs are not comparable to those of their cognate linear RNAs when analyzing RNA-seq datasets: "Since FPM is unscaled to FPKM, the relative expression levels of most circRNAs are not comparable to those of their cognate linear RNAs when analyzing RNA-seq datasets"
- [abstract] dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using: "dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using"
- [methods] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [discussion] we have developed CLEAR by applying normalized RNA-seq fragments solely mapped to BSJ or canonical SJ sites individually: "we have developed CLEAR by applying normalized RNA-seq fragments solely mapped to BSJ or canonical SJ sites individually"
- [discussion] polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
