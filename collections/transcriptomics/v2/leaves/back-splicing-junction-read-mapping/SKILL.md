---
name: back-splicing-junction-read-mapping
description: Use when quantifying circRNA expression from ribosomal-RNA depleted paired-end
  RNA-seq FASTQ files by detecting fragments spanning back-splicing junction sites,
  prior to FPB/CIRCscore quantification.
when_to_use_negative:
- 'Input is poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs
  are largely depleted, making FPBlinear unreliable and precluding valid CIRCscore
  evaluation'
- Input is poly(A)+ RNA-seq, in which circRNAs are depleted because they lack 3′ poly(A)
  tails
- Only linear RNA expression is of interest and no circ/linear comparison is needed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CIRCexplorer2
  role: Annotates back-splicing junction reads into circular RNA calls via fusion
    junction realignment (TopHat-Fusion); CLEAR requires >=2.3.6
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligns RNA-seq reads to the hg38 genome producing the BAM of mapped reads
    consumed by circ_quant; requires >=2.0.5
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: CIRCexplorer3-CLEAR
  role: Wraps mapping and quantification via clear_quant/circ_quant to convert BSJ
    read counts into FPBcirc values
  repo: https://github.com/YangLab/CLEAR
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:7ddeb9d45c886dcea7b964ffb777d91a972f5fd638e38cbe4e2101b15a24fe86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CIRCexplorer2
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - StringTie
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# back-splicing-junction-read-mapping

## Summary

Detect and quantify back-splicing junction (BSJ) reads from ribo− RNA-seq by aligning reads with HISAT2 and realigning unmapped/edge reads with TopHat-Fusion (Bowtie1) via CIRCexplorer2, providing the circRNA calls that feed CLEAR's circ_quant for FPBcirc and CIRCscore computation.

## When to use

Use when quantifying circRNA expression from ribosomal-RNA depleted paired-end RNA-seq FASTQ files by detecting fragments spanning back-splicing junction sites, prior to FPB/CIRCscore quantification.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq: polyadenylated linear RNAs are largely depleted, making FPBlinear unreliable and precluding valid CIRCscore evaluation
- Input is poly(A)+ RNA-seq, in which circRNAs are depleted because they lack 3′ poly(A) tails
- Only linear RNA expression is of interest and no circ/linear comparison is needed

## Inputs

- Ribo− paired-end RNA-seq FASTQ files (mate_1 and mate_2)
- hg38 genome FASTA file
- HISAT2 index files for hg38
- Bowtie1 index files for TopHat-Fusion
- Annotation GTF file
- refFlat format gene annotation file (for circ_quant)

## Outputs

- CIRCexplorer2 circRNA call file with readNumber, exonSizes, exonOffsets, and fusion-junction realignment flags
- HISAT2-aligned BAM with unmapped reads removed
- quant.txt table with readNumber, FPBcirc, FPBlinear, and CIRCscore per circRNA

## How to apply

Start from ribo− paired-end FASTQ files and run clear_quant with the hg38 genome FASTA (-g), a HISAT2 index (>=2.0.5) for genome alignment to BAM, and a Bowtie1 index for TopHat-Fusion fusion junction realignment (-j), plus an annotation GTF (-G) to precisely annotate the back-spliced circular RNAs. The HISAT2-aligned BAM must have unmapped reads removed before downstream use. Feed the CIRCexplorer2 circRNA calls and the cleaned BAM into circ_quant with a refFlat annotation, keeping the default FPB threshold of 1 for filtering linear SJ and default circ-to-linear ratio of 1. Each circRNA record carries readNumber (junction reads) and the FPBcirc normalization of fragments mapped to circRNA-specific BSJ sites, which is later divided by FPBlinear to yield CIRCscore.

## Related tools

- **CIRCexplorer2** (Annotates back-splicing junction reads into circular RNA calls via fusion junction realignment (TopHat-Fusion); CLEAR requires >=2.3.6) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligns RNA-seq reads to the hg38 genome producing the BAM of mapped reads consumed by circ_quant; requires >=2.0.5) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **CIRCexplorer3-CLEAR** (Wraps mapping and quantification via clear_quant/circ_quant to convert BSJ read counts into FPBcirc values) — https://github.com/YangLab/CLEAR

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- Verify the HISAT2-aligned BAM contains no unmapped reads before circ_quant
- Confirm quant.txt output includes the FPBcirc ('Expression of circRNA') and CIRCscore ('Relative expression of circRNA') columns
- Check that readNumber in the circRNA records corresponds to back-splicing junction read counts
- For ENCODE 12-tissue data, the fraction of FPBcirc >= 1 circRNAs with CIRCscore >= 1 should fall in the ~10%–20% range reported in Figure 5A
- score field reflects the fusion junction realignment flag from CIRCexplorer2

## Limitations

- CIRCscore from BSJ-derived FPBcirc cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are depleted there
- The BAM fed to circ_quant must have unmapped reads removed or quantification fails
- No explicit CLEAR software version numbers are stated; only CIRCexplorer2 >=2.3.6 and HISAT2 >=2.0.5 are pinned

## Evidence

- [abstract] examination of global circRNA expression from RNA-seq datasets generally relies on the detection of RNA-seq fragments spanning BSJ sites: "examination of global circRNA expression from RNA-seq datasets generally relies on the detection of RNA-seq fragments spanning BSJ sites"
- [readme] clear_quant usage: "usage: clear_quant [-h] -1 M1 [-2 M2] -g GENOME -i HISAT -j BOWTIE1 -G GTF"
- [readme] circ_quant BAM requirement: "*hisat_aligned.bam should not contain unmapped reads.*"
- [methods] threshold parameter: "Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]"
- [discussion] poly(A)− limitation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
