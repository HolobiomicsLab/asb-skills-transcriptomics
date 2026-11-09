---
name: ribo-depleted-rnaseq-alignment
description: Use when input is paired-end ribo− RNA-seq FASTQ data and the goal is
  circRNA/linear RNA expression analysis with CLEAR (FPBquantification, CIRCscore)
  or downstream circRNA detection (CIRCexplorer2, DCC).
when_to_use_negative:
- Input is poly(A)+ selected RNA-seq, which depletes circular RNAs lacking poly(A)
  tails
- Input is RNase R-treated RNA-seq when an accurate CIRCscore is needed, since linear
  RNAs are largely degraded and FPBlinear becomes unreliable
- The BAM supplied to circ_quant still contains unmapped reads
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3523
tools:
- name: HISAT2
  role: Primary spliced aligner mapping ribo− RNA-seq reads to the hg38 genome; version
    >=2.0.5 required
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: CIRCexplorer3-CLEAR
  role: Pipeline wrapping the alignment (clear_quant) that chains HISAT2, TopHat-Fusion,
    CIRCexplorer2 and FPB/CIRCscore quantification
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Annotates circRNAs from the aligned BAM via fusion-junction realignment; version
    >=2.3.6 required
  repo: https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:522fe9f227301da8cc394a061c619978ece4f3ec920739b108f14bbe742ec338
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - DCC/circtools
  - HISAT2
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# ribo-depleted-rnaseq-alignment

## Summary

Aligns ribosomal-RNA depleted (ribo−) RNA-seq reads with HISAT2 and performs TopHat-Fusion fusion-junction realignment using a Bowtie1 index, producing the mapped BAMs that CIRCexplorer3-CLEAR uses to quantify fragments at circRNA-specific BSJ sites and linear RNA-specific SJ sites.

## When to use

Use when input is paired-end ribo− RNA-seq FASTQ data and the goal is circRNA/linear RNA expression analysis with CLEAR (FPBquantification, CIRCscore) or downstream circRNA detection (CIRCexplorer2, DCC).

## When NOT to use

- Input is poly(A)+ selected RNA-seq, which depletes circular RNAs lacking poly(A) tails
- Input is RNase R-treated RNA-seq when an accurate CIRCscore is needed, since linear RNAs are largely degraded and FPBlinear becomes unreliable
- The BAM supplied to circ_quant still contains unmapped reads

## Inputs

- Paired-end ribo− RNA-seq FASTQ files (mate_1 and mate_2)
- Reference genome FASTA (e.g. hg38.fa)
- Prebuilt HISAT2 index (>=2.0.5)
- Bowtie1 index for TopHat-Fusion
- Annotation GTF file

## Outputs

- hisat_aligned.bam containing only mapped reads
- TopHat-Fusion/CIRCexplorer2 fusion-junction realignment output for BSJ detection
- output_dir/quant/quant.txt quantification table with chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, CIRCscore

## How to apply

Retrieve the paired-end FASTQ files (e.g. PA1 ribo− RNA-seq from GEO GSE75733/GSE73325) together with the hg38 genome FASTA, a prebuilt hg38 HISAT2 index, a hg38 Bowtie1 index (for TopHat-Fusion), and an annotation GTF. Align reads with HISAT2 (>=2.0.5), ensuring the resulting hisat_aligned.bam contains only mapped reads, because CLEAR's circ_quant requires the BAM to exclude unmapped reads. Perform fusion-junction realignment with TopHat-Fusion on the Bowtie1 index to detect back-splicing junction fragments, relying on CIRCexplorer2 (>=2.3.6) for circRNA annotation. Practically, run CLEAR's clear_quant directly on the FASTQ files (default 5 threads), which chains HISAT2 alignment, TopHat-Fusion realignment, and CIRCexplorer2-based circRNA calling before FPB/CIRCscore quantification.

## Related tools

- **HISAT2** (Primary spliced aligner mapping ribo− RNA-seq reads to the hg38 genome; version >=2.0.5 required) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **CIRCexplorer3-CLEAR** (Pipeline wrapping the alignment (clear_quant) that chains HISAT2, TopHat-Fusion, CIRCexplorer2 and FPB/CIRCscore quantification) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Annotates circRNAs from the aligned BAM via fusion-junction realignment; version >=2.3.6 required) — https://github.com/YangLab/CIRCexplorer2/tree/master/circ2

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- hisat_aligned.bam exists and contains no unmapped reads, as required by circ_quant
- quant.txt schema contains chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, CIRCscore
- For each locus, CIRCscore equals FPBcirc divided by FPBlinear
- clear_quant runs with the documented argument set (-1 mate_1 -2 mate_2 -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf) and default 5 threads
- Downstream quantification reproduces expected correlation values (e.g. FPBlinear vs FPKMlinear Pearson r = 0.92 in PA1 cells)

## Limitations

- BAM input must not contain unmapped reads, or circ_quant fails
- HISAT2 >=2.0.5 and CIRCexplorer2 >=2.3.6 are minimum version requirements; StringTie >1.3.6 and pysam >=0.8.4 are also required
- CIRCscore cannot be accurately applied to poly(A)−/ribo− or RNase R-treated datasets because linear RNA quantification is unreliable

## Evidence

- [intro] circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR, or CLEAR for simplicity: "for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR, or CLEAR for simplicity, Figure 1B)"
- [methods] clear_quant starts from FASTQ with HISAT2, TopHat-Fusion (Bowtie1) and GTF inputs: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [methods] circ_quant requires a mapped-read BAM, which is why the BAM must exclude unmapped reads: "-b BAM, --bam BAM     Input mapped reads from HISAT2 in BAM format."
- [readme] README warning that hisat_aligned.bam must not contain unmapped reads: "*hisat_aligned.bam should not contain unmapped reads.*"
- [discussion] Quantification is based on fragments mapped to BSJ and SJ sites individually: "we have developed CLEAR by applying normalized RNA-seq fragments solely mapped to BSJ or canonical SJ sites individually"
