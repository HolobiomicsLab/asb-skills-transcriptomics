---
name: cross-dataset-read-length-stability-comparison
description: Use when evaluating whether an RNA-seq expression quantification parameter
  is robust to sequencing length or strategy differences, by building virtual short-read/single-end
  derivatives from paired-end data and comparing median quantification values.
when_to_use_negative:
- Input is already a quantification table and no FASTQ-derived derivative datasets
  can be constructed
- The dataset is poly(A)−/ribo− or RNase R-treated, where linear RNAs are depleted
  and linear quantification (and thus metric comparison) is unreliable
- The question concerns biological reproducibility across samples rather than technical
  stability of a normalization metric across sequencing configurations
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: clear_quant/circ_quant quantify circRNAs and linear RNAs, producing FPBcirc,
    FPBlinear, and CIRCscore used for the cross-dataset comparison
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Provides the circRNA annotation output (>=2.3.6) that circ_quant consumes
    as input
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Maps reads (>=2.0.5) to the hg38 genome, producing the BAM used for quantification
    in each dataset variant
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: pysam
  role: Python package dependency (>=0.8.4) for BAM/FASTQ manipulation in the read-processing
    and quantification steps
  repo: http://pysam.readthedocs.org/en/latest/
- name: pybedtools
  role: Python package dependency for interval manipulation in the CLEAR pipeline
  repo: http://daler.github.io/pybedtools/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:9f2bf0a0098e943d0d25ffaaa90fd6d5dafde54dcf12c29fcb576b15ce93042e
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-dataset-read-length-stability-comparison

## Summary

Assesses whether a quantification metric (FPB) is resistant to differences in sequencing read lengths and strategies by generating virtual derivative RNA-seq datasets from one sample and comparing median FPB, FPM, and FPKM across them, as done for the cortex ribo− dataset GSM2072380.

## When to use

Use when evaluating whether an RNA-seq expression quantification parameter is robust to sequencing length or strategy differences, by building virtual short-read/single-end derivatives from paired-end data and comparing median quantification values.

## When NOT to use

- Input is already a quantification table and no FASTQ-derived derivative datasets can be constructed
- The dataset is poly(A)−/ribo− or RNase R-treated, where linear RNAs are depleted and linear quantification (and thus metric comparison) is unreliable
- The question concerns biological reproducibility across samples rather than technical stability of a normalization metric across sequencing configurations

## Inputs

- Paired-end 2x101 bp ribo− RNA-seq FASTQ files (e.g. GSM2072380 cortex sample from GEO)
- hg38 genome FASTA, HISAT2 index, Bowtie1 index, and annotation GTF
- refFlat-format gene annotation
- Read-processing script for generating truncated/single-end FASTQ derivatives (pysam/pybedtools environment)

## Outputs

- Three derivative/original FASTQ datasets (2x101 bp, 1x101 bp, 1x50 bp)
- quant.txt per dataset with readNumber, FPBcirc, FPBlinear, and CIRCscore fields
- Comparison table/plot of median FPM, FPB, and FPKM across the three datasets

## How to apply

Download paired 2x101 bp ribo− cortex RNA-seq FASTQ files (e.g. GSM2072380 from GEO) and construct derivative datasets with a read-processing script: keep only read1 to make a 1x101 bp set, and truncate read1 to 50 bp to make a 1x50 bp set. Quantify each of the three datasets with CIRCexplorer3-CLEAR's clear_quant (paired-end for the original, single-end for derivatives), mapping reads with HISAT2 (>=2.0.5) to hg38 using HISAT2 and Bowtie1 indices plus annotation GTF, or run circ_quant on CIRCexplorer2 output with the HISAT2 BAM and refFlat annotation using default parameters (--threshold 1, --ratio 1). Collect the quant.txt outputs and compute median FPM, FPB, and FPKM per dataset to populate the comparison (Figure 2E-style table/plot). The rationale is that FPB, being normalized to mapped bases, should stay largely unchanged as read length or paired/single-end strategy changes, whereas FPM varies and FPKM is less strongly but still measurably altered.

## Related tools

- **CIRCexplorer3-CLEAR** (clear_quant/circ_quant quantify circRNAs and linear RNAs, producing FPBcirc, FPBlinear, and CIRCscore used for the cross-dataset comparison) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Provides the circRNA annotation output (>=2.3.6) that circ_quant consumes as input) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Maps reads (>=2.0.5) to the hg38 genome, producing the BAM used for quantification in each dataset variant) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **pysam** (Python package dependency (>=0.8.4) for BAM/FASTQ manipulation in the read-processing and quantification steps) — http://pysam.readthedocs.org/en/latest/
- **pybedtools** (Python package dependency for interval manipulation in the CLEAR pipeline) — http://daler.github.io/pybedtools/

## Examples

```
clear_quant -1 read1_50bp.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o quant_1x50_dir
```

## Evaluation signals

- Median FPB of the 1x101 bp and 1x50 bp derivative datasets remains close to the full 2x101 bp dataset
- Median FPM deviates with read length, confirming the expected contrast with FPB
- All three quant.txt files exist with FPBcirc, FPBlinear, and CIRCscore fields populated
- The HISAT2 BAM supplied to circ_quant contains no unmapped reads

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear quantification unsuitable
- The stability test is constructed from a single source sample (virtual datasets), so it measures normalization robustness, not biological variability
- FPKM is 'not greatly altered' across lengths, so FPB's advantage over FPKM is less pronounced than over FPM

## Evidence

- [results] When virtual RNA-seq datasets (1 × 101 bp and 1 × 50 bp) were constructed from the 2 × 101 bp cortex ribo− data, FPB largely remained unchanged across sequencing lengths and strategies, unlike FPM: [evidence span withheld — anchor status: not_found]
- [readme] clear_quant usage from README: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
- [readme] BAM constraint from README: "*hisat_aligned.bam should not contain unmapped reads.*"
- [abstract] FPB definition from abstract: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually"
- [abstract] CIRCscore definition from abstract: "dividing FPBcirc by FPBlinear to generate a CIRCscore, which indicates the relative circRNA expression level using"
