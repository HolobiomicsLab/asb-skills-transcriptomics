---
name: cross-tool-circrna-quantification-comparison
description: Use when comparing alternative circRNA quantification tools on the same
  ribo− RNA-seq data to decide whether a CIRCscore-style normalized ratio is more
  accurate than CLR-style back-splice counting, e.g. for loci with RT-qPCR validation
  values.
when_to_use_negative:
- 'Input is poly(A)−/ribo− or RNase R-treated RNA-seq: linear RNAs are largely depleted,
  so FPBlinear/CIRCscore evaluation is unsuitable per the article''s limitation.'
- Goal is de novo circRNA discovery/annotation rather than comparing quantification
  accuracy between CLEAR and DCC/circtools.
- Only one quantification tool's output is available; a cross-tool comparison requires
  both CIRCexplorer2/CLEAR and DCC results on the same mapped BAM.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Quantifies circular and linear RNA expression via FPB and computes CIRCscore
    (FPBcirc/FPBlinear) used as the reference metric
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Embedded quantification engine in CLEAR (>=2.3.6) calling BSJ fragments (~126
    for circCAMSAP1-1)
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Maps reads to produce the BAM consumed by circ_quant (>=2.0.5)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: DCC/circtools
  role: Alternative multi-step method run on the same mapped BAM to call BSJ fragments
    (~36) and compute CLR for benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:a09ae2f387f912431fa9318d3f72848cb0911d7dfe46c74fc6d14ea0329aeb80
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-tool-circrna-quantification-comparison

## Summary

Benchmark circRNA quantification accuracy across CLEAR (CIRCscore from FPBcirc/FPBlinear) and circtools/DCC (CLR from DCC-called BSJ fragment counts) on ribo− RNA-seq, validating CIRCscore against RT-qPCR for loci such as circCAMSAP1-1.

## When to use

Use when comparing alternative circRNA quantification tools on the same ribo− RNA-seq data to decide whether a CIRCscore-style normalized ratio is more accurate than CLR-style back-splice counting, e.g. for loci with RT-qPCR validation values.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq: linear RNAs are largely depleted, so FPBlinear/CIRCscore evaluation is unsuitable per the article's limitation.
- Goal is de novo circRNA discovery/annotation rather than comparing quantification accuracy between CLEAR and DCC/circtools.
- Only one quantification tool's output is available; a cross-tool comparison requires both CIRCexplorer2/CLEAR and DCC results on the same mapped BAM.

## Inputs

- PA1 ribo− paired-end FASTQ files
- hg38 genome FASTA (hg38.fa)
- HISAT2 index (hg38.hisat_index)
- Bowtie1/TopHat-Fusion index (hg38.bowtie_index)
- GTF annotation (annotation.gtf)
- refFlat gene annotation (annotation.refFlat)
- CIRCexplorer2 output file
- HISAT2-aligned BAM without unmapped reads

## Outputs

- output_dir/quant/quant.txt with FPBcirc, FPBlinear, and CIRCscore per circRNA
- DCC/circtools BSJ fragment counts per circRNA locus
- Comparison table of CIRCscore, CLR, and BSJ fragment counts (e.g. circCAMSAP1-1_comparison_table.tsv)

## How to apply

Run clear_quant on PA1 ribo− paired-end FASTQ files with the hg38 genome FASTA, HISAT2 and Bowtie1 indices, and GTF annotation to obtain CIRCexplorer2-based quantification (output_dir/quant/quant.txt). Then run circ_quant with the CIRCexplorer2 output (-c), the HISAT2-aligned BAM free of unmapped reads (-b), and refFlat annotation (-r), keeping default --threshold 1 and --ratio 1, to extract FPBcirc, FPBlinear, and CIRCscore (≈3.65 for circCAMSAP1-1, matching RT-qPCR ≈3.82). In parallel, run DCC from circtools on the same mapped BAM to count BSJ-spanning fragments (≈36 for circCAMSAP1-1 vs ≈126 from CLEAR's embedded CIRCexplorer2) and compute CLR (≈0.80). Assemble CIRCscore, CLR, and BSJ fragment counts into a comparison table; DCC's lower fragment-calling efficiency indicates CIRCscore is the more accurate metric.

## Related tools

- **CIRCexplorer3-CLEAR** (Quantifies circular and linear RNA expression via FPB and computes CIRCscore (FPBcirc/FPBlinear) used as the reference metric) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Embedded quantification engine in CLEAR (>=2.3.6) calling BSJ fragments (~126 for circCAMSAP1-1)) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Maps reads to produce the BAM consumed by circ_quant (>=2.0.5)) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **DCC/circtools** (Alternative multi-step method run on the same mapped BAM to call BSJ fragments (~36) and compute CLR for benchmarking)

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- CIRCscore for circCAMSAP1-1 ≈3.65, consistent with RT-qPCR validation of ≈3.82 (Figure 3A).
- CLR for circCAMSAP1-1 ≈0.80 from DCC/circtools.
- BSJ fragment counts for circCAMSAP1-1: ~126 from CIRCexplorer2/CLEAR vs ~36 from DCC; DCC's deficit confirms less efficient BSJ-mapped fragment calling.
- quant.txt contains FPBcirc, FPBlinear, and CIRCscore columns with sensible FPBcirc > 0 for the target locus.
- Assembled comparison table matches Figure 3A values (CIRCscore ~3.65, CLR ~0.80, 126 vs 36 fragments).

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification inaccurate.
- DCC calls fewer BSJ fragments (36 vs 126) at the circCAMSAP1-1 locus, underestimating circRNA expression relative to CIRCexplorer2.
- hisat_aligned.bam must not contain unmapped reads for circ_quant to run correctly.
- No software version numbers stated for CLEAR itself; only CIRCexplorer2 (>=2.3.6) and HISAT2 (>=2.0.5) version requirements are given.

## Evidence

- [abstract] CIRCscore, computed as FPBcirc divided by FPBlinear, indicates relative circRNA expression level using linear RNA expression as background: [evidence span withheld — anchor status: not_found]
- [discussion] comparing DCC to CLEAR: "Compared to other multi-step methods for circular and linear RNA comparison, such as DCC/circtools"
- [readme] circ_quant CLI from README: "circ_quant -c CIRCexplorer2_output.txt -b hisat_aligned.bam -t -r annotation.refFlat -o quant.txt"
- [readme] BAM requirement: "*hisat_aligned.bam should not contain unmapped reads.*"
- [discussion] CIRCscore limitation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
