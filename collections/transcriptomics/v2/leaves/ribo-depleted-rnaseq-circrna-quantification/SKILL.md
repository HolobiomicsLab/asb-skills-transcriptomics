---
name: ribo-depleted-rnaseq-circrna-quantification
description: Use when ribosomal-RNA depleted paired-end RNA-seq FASTQ must be converted
  into per-locus circRNA quantifications (readNumber, FPBcirc, FPBlinear, CIRCscore)
  that are directly comparable to cognate linear RNA expression.
when_to_use_negative:
- Input is poly(A)−/ribo− or RNase R-treated RNA-seq, where polyadenylated linear
  RNAs are largely depleted, making accurate linear quantification and CIRCscore evaluation
  unsuitable
- Input is poly(A)+ RNA-seq data, in which circRNAs are depleted because they lack
  3′ poly(A) tails
- Only linear RNA expression is needed with no circular/linear comparison goal — a
  standard transcript quantifier suffices
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline that quantifies circular and linear RNA expression via FPB and computes
    CIRCscore from ribo− RNA-seq (clear_quant / circ_quant commands)
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Upstream circRNA caller (>=2.3.6) whose circRNA output feeds circ_quant; performs
    fusion junction realignment of TopHat-Fusion candidates
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligns ribo− paired-end reads (>=2.0.5) to produce the mapped BAM (unmapped
    reads removed) used for linear SJ quantification
- name: StringTie
  role: Installation requirement (>1.3.6) for the CLEAR pipeline
  repo: https://ccb.jhu.edu/software/stringtie
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:9089611fc4254d1fc84be005170324e8fa1437136a3128dbd88ee42bec45e809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - StringTie
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ribo-depleted-rnaseq-circrna-quantification

## Summary

Quantify circRNA and cognate linear RNA expression from ribo− RNA-seq with CIRCexplorer3-CLEAR using FPB (fragments per billion mapped bases) values normalized separately over BSJ and SJ sites, and compute CIRCscore = FPBcirc/FPBlinear for direct circular-to-linear comparison.

## When to use

Use when ribosomal-RNA depleted paired-end RNA-seq FASTQ must be converted into per-locus circRNA quantifications (readNumber, FPBcirc, FPBlinear, CIRCscore) that are directly comparable to cognate linear RNA expression.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq, where polyadenylated linear RNAs are largely depleted, making accurate linear quantification and CIRCscore evaluation unsuitable
- Input is poly(A)+ RNA-seq data, in which circRNAs are depleted because they lack 3′ poly(A) tails
- Only linear RNA expression is needed with no circular/linear comparison goal — a standard transcript quantifier suffices

## Inputs

- Ribosomal-RNA depleted (ribo−) paired-end RNA-seq FASTQ files (mate_1.fastq, mate_2.fastq)
- Reference genome FASTA (e.g. hg38.fa)
- HISAT2 genome index
- Bowtie1 (TopHat-Fusion) index
- Gene annotation GTF file
- For circ_quant mode: CIRCexplorer2 circRNA output file, HISAT2-aligned BAM without unmapped reads, refFlat-format gene annotation

## Outputs

- Per-circRNA quantification table (quant.txt) with chrom, start, end, strand, exonCount, readNumber, circType, geneName, FPBcirc, FPBlinear, and CIRCscore columns
- Per-tissue summary tables of circRNA counts, median FPBcirc, CIRCscore distributions, and fraction of circRNAs with CIRCscore ≥ 1 (e.g. encode_tissue_circ_quant.tsv)

## How to apply

Run clear_quant on each ribo− paired-end FASTQ with the reference genome FASTA (e.g. hg38.fa), HISAT2 index, Bowtie1 (TopHat-Fusion) index, and annotation GTF (default 5 threads) to align reads and call circRNAs via CIRCexplorer2; alternatively start from existing CIRCexplorer2 output with circ_quant (-c CIRC file, -b HISAT2-aligned BAM with unmapped reads removed, -r refFlat annotation). circ_quant keeps the default FPB threshold of 1 for choosing circRNAs to filter linear SJ and the default circ-to-linear ratio of 1. Parse the resulting quant.txt/quant RNA-style output for readNumber, FPBcirc ('Expression of circRNA'), FPBlinear ('Expression of cognate linear RNA'), and CIRCscore ('Relative expression of circRNA'). Because circular and linear expression are both normalized as FPB over BSJ and SJ sites respectively, dividing FPBcirc by FPBlinear yields a directly comparable CIRCscore, unlike incomparable FPM vs FPKM quantifications.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline that quantifies circular and linear RNA expression via FPB and computes CIRCscore from ribo− RNA-seq (clear_quant / circ_quant commands)) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Upstream circRNA caller (>=2.3.6) whose circRNA output feeds circ_quant; performs fusion junction realignment of TopHat-Fusion candidates) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligns ribo− paired-end reads (>=2.0.5) to produce the mapped BAM (unmapped reads removed) used for linear SJ quantification)
- **StringTie** (Installation requirement (>1.3.6) for the CLEAR pipeline) — https://ccb.jhu.edu/software/stringtie

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt contains the FPBcirc column ('Expression of circRNA') and CIRCscore column ('Relative expression of circRNA') for every called circRNA
- In an ENCODE 12-tissue analysis, the fraction of circRNAs with FPBcirc ≥ 1 that also have CIRCscore ≥ 1 falls in the ~10%–20% range reported in Figure 5A
- hisat_aligned.bam contains no unmapped reads, as required by circ_quant
- FPB filtering used the default threshold of 1 and the default circ:linear ratio of 1 unless deliberately overridden
- Brain tissues (cortex, occipital, diencephalon) show higher FPBcirc/CIRCscore abundance than non-brain tissues, reproducing the published enrichment pattern

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because depleted polyadenylated linear RNAs preclude accurate linear quantification
- circ_quant requires the HISAT2-aligned BAM to have unmapped reads removed
- By default the -l/--length option is False, i.e. not all reads' lengths are considered; changing it alters FPB normalization

## Evidence

- [abstract] we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR): "we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR)"
- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [readme] usage: circ_quant [-h] -c CIRC -b BAM -r REF [--threshold THRESHOLD] [--ratio RATIO] [-l] [-t] [-o OUTPUT]: "usage: circ_quant [-h] -c CIRC -b BAM -r REF [--threshold THRESHOLD]
                     [--ratio RATIO] [-l] [-t] [-o OUTPUT]"
- [methods] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [discussion] polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
