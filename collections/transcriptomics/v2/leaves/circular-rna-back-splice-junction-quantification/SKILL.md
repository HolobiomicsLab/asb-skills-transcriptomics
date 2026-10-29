---
name: circular-rna-back-splice-junction-quantification
description: Use when ribo− depleted RNA-seq FASTQ (or a CIRCexplorer2 circRNA output
  plus a HISAT2 BAM) needs circRNA quantification per BSJ with readNumber, FPBcirc,
  FPBlinear, and CIRCscore per locus.
when_to_use_negative:
- Input is poly(A)−/ribo− or RNase R-treated RNA-seq, where polyadenylated linear
  RNAs are largely depleted and accurate linear RNA quantification (and thus CIRCscore)
  is unsuitable
- Input is already a CIRCexplorer3-CLEAR quant.txt with FPBcirc/FPBlinear/CIRCscore
  fields populated — re-quantification is redundant
- Only poly(A)+ RNA-seq is available, since circRNAs are depleted in poly(A)+ datasets
  and BSJ fragments will be underrepresented
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline (clear_quant / circ_quant) that normalizes fragments mapped to BSJ
    and SJ sites into FPBcirc, FPBlinear, and CIRCscore
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Upstream circRNA detection providing the circular RNA input file for circ_quant
    (requires >=2.3.6)
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Spliced read aligner producing the BAM of mapped reads used by clear_quant/circ_quant
    (requires >=2.0.5)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: StringTie
  role: Installation requirement of the CLEAR pipeline (linear transcript assembly;
    >1.3.6)
  repo: https://ccb.jhu.edu/software/stringtie
- name: pysam
  role: Python BAM/FASTQ handling dependency of CLEAR (>=0.8.4)
- name: pybedtools
  role: Python genomic-interval manipulation dependency of CLEAR
  repo: http://daler.github.io/pybedtools/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:e3e300db0575812f7e2ab44c67c5275817c71cb205065254807e3249a0ed0c56
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# circular-rna-back-splice-junction-quantification

## Summary

Quantify circular RNA expression from ribo− RNA-seq by counting fragments mapped to back-splicing junction (BSJ) sites with the CIRCexplorer3-CLEAR pipeline, and express circRNA and cognate linear RNA expression in a common FPB (fragments per billion mapped bases) scale so CIRCscore (FPBcirc/FPBlinear) can be computed.

## When to use

Use when ribo− depleted RNA-seq FASTQ (or a CIRCexplorer2 circRNA output plus a HISAT2 BAM) needs circRNA quantification per BSJ with readNumber, FPBcirc, FPBlinear, and CIRCscore per locus.

## When NOT to use

- Input is poly(A)−/ribo− or RNase R-treated RNA-seq, where polyadenylated linear RNAs are largely depleted and accurate linear RNA quantification (and thus CIRCscore) is unsuitable
- Input is already a CIRCexplorer3-CLEAR quant.txt with FPBcirc/FPBlinear/CIRCscore fields populated — re-quantification is redundant
- Only poly(A)+ RNA-seq is available, since circRNAs are depleted in poly(A)+ datasets and BSJ fragments will be underrepresented

## Inputs

- Paired-end or single-end FASTQ files from ribosomal-RNA depleted (ribo−) RNA-seq (e.g. GSM2072380 cortex 2x101 bp)
- hg38 genome FASTA and HISAT2 index
- hg38 Bowtie1 index (for TopHat-Fusion)
- Annotation GTF file
- CIRCexplorer2 circular RNA output file (for circ_quant)
- HISAT2-aligned BAM without unmapped reads (for circ_quant)
- refFlat-format gene annotation file (for circ_quant)

## Outputs

- quant.txt (output_dir/quant/quant.txt) with per-circRNA fields: chrom, start, end, strand, exonSizes/Offsets, readNumber, geneName, isoformName, FPBcirc, FPBlinear, CIRCscore
- Per-locus CIRCscore values (FPBcirc/FPBlinear) flagging highly-expressed circRNAs with low linear RNA background

## How to apply

Install CLEAR (git clone https://github.com/YangLab/CLEAR; python ./setup.py install) with CIRCexplorer2 (>=2.3.6), HISAT2 (>=2.0.5), StringTie (>1.3.6), pysam (>=0.8.4) and pybedtools. For paired-end FASTQ, run clear_quant with -1/-2 mates, the hg38 genome FASTA, hg38 HISAT2 and Bowtie1 (TopHat-Fusion) indices, and the annotation GTF; for single-end derivatives omit -2. Alternatively, start from a CIRCexplorer2 circRNA file and a HISAT2 BAM that must not contain unmapped reads, running circ_quant with the refFlat annotation and default parameters (--threshold 1 for FPB-based filtering of linear SJ, --ratio 1 for circ/linear comparison). The workflow performs fusion junction realignment and normalizes fragments mapped to circRNA-specific BSJ sites or linear RNA-specific SJ sites individually, yielding quant.txt with FPBcirc, FPBlinear, and CIRCscore per circRNA. CIRCscore = FPBcirc/FPBlinear uses linear expression as background, enabling direct comparison of circular and linear RNA from the same locus.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline (clear_quant / circ_quant) that normalizes fragments mapped to BSJ and SJ sites into FPBcirc, FPBlinear, and CIRCscore) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Upstream circRNA detection providing the circular RNA input file for circ_quant (requires >=2.3.6)) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Spliced read aligner producing the BAM of mapped reads used by clear_quant/circ_quant (requires >=2.0.5)) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **StringTie** (Installation requirement of the CLEAR pipeline (linear transcript assembly; >1.3.6)) — https://ccb.jhu.edu/software/stringtie
- **pysam** (Python BAM/FASTQ handling dependency of CLEAR (>=0.8.4))
- **pybedtools** (Python genomic-interval manipulation dependency of CLEAR) — http://daler.github.io/pybedtools/

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- quant.txt exists and contains populated FPBcirc, FPBlinear, and CIRCscore fields for detected circRNAs
- readNumber field reflects junction (BSJ-spanning fragment) counts per circRNA locus
- FPB values are stable across sequencing-length variants: median FPB of 1x101 bp and 1x50 bp derivatives stays close to the 2x101 bp original, unlike FPM
- CIRCscore equals FPBcirc divided by FPBlinear at each locus (default --ratio 1) and circRNAs below FPB threshold 1 are filtered from linear SJ consideration
- For circ_quant input, the HISAT2 BAM contains no unmapped reads

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear quantification inaccurate
- FPB (rather than FPM) is required because FPM is unscaled to FPKM, making circRNA and linear RNA expression incomparable in prior pipelines
- FPM deviates with read length/strategy; use FPB when comparing datasets with different sequencing lengths or strategies (e.g. 1x101 vs 1x50 bp derivatives)

## Evidence

- [abstract] A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites or to: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites or to"
- [readme] clear_quant and circ_quant usage: "usage: clear_quant [-h] -1 M1 [-2 M2] -g GENOME -i HISAT -j BOWTIE1 -G GTF"
- [readme] BAM must not contain unmapped reads: "*hisat_aligned.bam should not contain unmapped reads.*"
- [readme] CIRCscore definition: "| CIRCscore   | Relative expression of circRNA        |"
- [discussion] CIRCscore limitation on RNase R data: "polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification and subsequent CIRCscore evaluation"
