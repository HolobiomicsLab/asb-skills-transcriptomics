---
name: back-splicing-junction-circrna-detection
description: Use when ribo− RNA-seq FASTQ files must be screened for circRNAs via
  BSJ-spanning fragments, and quantification of circRNA expression relative to cognate
  linear RNA (CIRCscore = FPBcirc/FPBlinear) is required.
when_to_use_negative:
- Input is poly(A)+ RNA-seq or poly(A)−/ribo− or RNase R-treated data — polyadenylated
  linear RNAs are depleted in the latter and CIRCscore evaluation is unsuitable there
- Goal is only linear gene expression quantification without circRNA/BSJ analysis
- No bowtie1 (TopHat-Fusion) or HISAT2 index for the target genome is available and
  none can be built
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline orchestrating HISAT2 mapping and CIRCexplorer2 realignment to call
    BSJ fragments and compute FPBcirc, FPBlinear, and CIRCscore
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Performs fusion junction realignment to identify back-splicing junction reads
    (requires ≥2.3.6)
  repo: https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- name: HISAT2
  role: Maps RNA-seq reads to the genome; the aligned BAM (without unmapped reads)
    is input to circ_quant (requires ≥2.0.5)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: StringTie
  role: Installation requirement for CLEAR (>1.3.6)
  repo: https://ccb.jhu.edu/software/stringtie
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:99ab850dcfadde242dcc6e2a8b705f6be434a48d5115d5ae28496d04c199134d
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
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# back-splicing-junction-circRNA-detection

## Summary

Detect and quantify circular RNAs from ribosomal-RNA depleted (ribo−) RNA-seq by identifying RNA-seq fragments spanning back-splicing junction (BSJ) sites, using the CLEAR pipeline's clear_quant (HISAT2 alignment + CIRCexplorer2 ≥2.3.6 fusion junction realignment) to produce per-locus readNumber, FPBcirc, FPBlinear, and CIRCscore values.

## When to use

Use when ribo− RNA-seq FASTQ files must be screened for circRNAs via BSJ-spanning fragments, and quantification of circRNA expression relative to cognate linear RNA (CIRCscore = FPBcirc/FPBlinear) is required.

## When NOT to use

- Input is poly(A)+ RNA-seq or poly(A)−/ribo− or RNase R-treated data — polyadenylated linear RNAs are depleted in the latter and CIRCscore evaluation is unsuitable there
- Goal is only linear gene expression quantification without circRNA/BSJ analysis
- No bowtie1 (TopHat-Fusion) or HISAT2 index for the target genome is available and none can be built

## Inputs

- Paired-end ribo− RNA-seq FASTQ files (mate_1.fastq, mate_2.fastq)
- Reference genome FASTA (e.g., hg38.fa)
- HISAT2 genome index
- Bowtie1 index for TopHat-Fusion
- Gene annotation GTF file
- refFlat-format gene annotation (for circ_quant)
- HISAT2-aligned BAM free of unmapped reads (for circ_quant)
- CIRCexplorer2 circRNA output file (for circ_quant)

## Outputs

- output_dir/quant/quant.txt table with chrom, start, end, strand, exonCount, readNumber (BSJ junction reads), FPBcirc, FPBlinear, and CIRCscore per circRNA
- Filtered circRNA call set restricted to loci with ≥3 BSJ-supporting fragments
- Genomic-coordinate overlap counts (chrom, start, end, strand) against independent call sets (e.g., MapSplice, DCC)

## How to apply

Install CLEAR from github.com/YangLab/CLEAR with 'python ./setup.py install' (requires CIRCexplorer2 ≥2.3.6, HISAT2 ≥2.0.5, StringTie >1.3.6, pysam ≥0.8.4, pybedtools). Run clear_quant on paired-end FASTQ mates with the hg38 genome FASTA (-g), HISAT2 index (-i), bowtie1/TopHat-Fusion index (-j), and annotation GTF (-G); clear_quant maps reads with HISAT2, performs fusion junction realignment with CIRCexplorer2 (TopHat-Fusion bowtie1 index), and emits output_dir/quant/quant.txt. If a CIRCexplorer2 output file already exists, run circ_quant with -c CIRCexplorer2_output.txt, -b a HISAT2-aligned BAM that must not contain unmapped reads, and -r a refFlat annotation, using the default FPB threshold of 1 and circ:linear ratio of 1. Filter detected circRNAs by a minimum of ≥3 BSJ-supporting fragments (readNumber) for downstream overlap comparisons. Validate outputs by checking that quant.txt contains the BSJ readNumber, FPBcirc, FPBlinear, and CIRCscore columns and that reported loci (e.g., circCAMSAP1-1 with CIRCscore ≈3.65 from ~126 BSJ fragments) reproduce published values.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline orchestrating HISAT2 mapping and CIRCexplorer2 realignment to call BSJ fragments and compute FPBcirc, FPBlinear, and CIRCscore) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Performs fusion junction realignment to identify back-splicing junction reads (requires ≥2.3.6)) — https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- **HISAT2** (Maps RNA-seq reads to the genome; the aligned BAM (without unmapped reads) is input to circ_quant (requires ≥2.0.5)) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **StringTie** (Installation requirement for CLEAR (>1.3.6)) — https://ccb.jhu.edu/software/stringtie

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt contains the documented columns including readNumber, FPBcirc, FPBlinear, and CIRCscore for each circRNA locus
- Reproducing Figure 3B-C: overlap of ≥3-fragment-filtered call sets yields 356 circRNAs detected by CIRCexplorer2 and MapSplice but missed by DCC, and 787 in the three-way overlap
- For circCAMSAP1-1, CLEAR reports ~126 BSJ fragments and a CIRCscore of ~3.65, consistent with RT-qPCR validation of ~3.82
- CIRCscore values equal FPBcirc/FPBlinear for each locus
- Input BAM to circ_quant contains no unmapped reads (a stated requirement)

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification and CIRCscore evaluation inaccurate
- DCC (circtools) is less efficient at BSJ-mapped fragment calling: among 787 overlapping circRNAs in PA1 ribo− data, DCC yielded fewer BSJ fragments (e.g., 36 vs CLEAR's 126 for circCAMSAP1-1, CLR ~0.80)
- hisat_aligned.bam must not contain unmapped reads
- No software version numbers are stated for CLEAR itself in the repository context

## Evidence

- [abstract] circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR): "we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR)"
- [abstract] FPB and CIRCscore definition: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually"
- [methods] install command: "git clone https://github.com/YangLab/CLEAR
cd CLEAR
python ./setup.py install"
- [methods] clear_quant example with hg38: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [readme] BAM must exclude unmapped reads: "hisat_aligned.bam should not contain unmapped reads."
- [discussion] CIRCscore limitation for other library types: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
- [readme] output readNumber field: "readNumber  | Number of junction reads              |"
