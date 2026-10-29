---
name: back-splice-junction-fragment-quantification
description: Use when you have ribosomal-RNA depleted RNA-seq FASTQ (or a CIRCexplorer2
  output plus HISAT2-aligned BAM) and need directly comparable circular vs. linear
  RNA expression values (FPB and CIRCscore) at shared gene loci.
when_to_use_negative:
- Input RNA-seq is poly(A)+ selected — circRNAs are depleted in poly(A)+ datasets,
  so BSJ fragments will be largely missing
- Data are poly(A)−/ribo− or RNase R-treated — linear RNAs are depleted, making FPBlinear
  and CIRCscore unreliable
- Input BAM still contains unmapped reads — hisat_aligned.bam must be mapped-reads-only
  before circ_quant
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline providing clear_quant and circ_quant commands for FPB quantification
    of circRNA BSJ and linear SJ fragments and CIRCscore computation
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Annotates circRNAs from TopHat-Fusion realigned BAM; its output file is the
    input to circ_quant (>=2.3.6 required)
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligns ribo− RNA-seq reads to hg38 to produce the mapped BAM used for linear
    SJ fragment quantification (>=2.0.5 required)
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: pysam
  role: Python package dependency (>=0.8.4) used by CLEAR for BAM handling
  repo: http://pysam.readthedocs.org/en/latest/
- name: pybedtools
  role: Python package dependency used by CLEAR for genomic interval operations
  repo: http://daler.github.io/pybedtools/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:8b628077f046a2faee364bde2c64f3143961b6c8516267aa00e43437e18558e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# back-splice-junction-fragment-quantification

## Summary

Quantifies circular and linear RNA expression from ribosomal-RNA depleted RNA-seq using the FPB (fragments per billion mapped bases) metric computed over fragments mapped to circRNA-specific BSJ sites and linear RNA-specific SJ sites, then computes CIRCscore = FPBcirc/FPBlinear for direct comparison at the same gene loci.

## When to use

Use when you have ribosomal-RNA depleted RNA-seq FASTQ (or a CIRCexplorer2 output plus HISAT2-aligned BAM) and need directly comparable circular vs. linear RNA expression values (FPB and CIRCscore) at shared gene loci.

## When NOT to use

- Input RNA-seq is poly(A)+ selected — circRNAs are depleted in poly(A)+ datasets, so BSJ fragments will be largely missing
- Data are poly(A)−/ribo− or RNase R-treated — linear RNAs are depleted, making FPBlinear and CIRCscore unreliable
- Input BAM still contains unmapped reads — hisat_aligned.bam must be mapped-reads-only before circ_quant

## Inputs

- Paired-end ribosomal-RNA depleted RNA-seq FASTQ files (mate_1 and mate_2), e.g. PA1 from GEO GSE75733/GSE73325
- hg38 genome FASTA (hg38.fa)
- Prebuilt hg38 HISAT2 index
- hg38 Bowtie1 index for TopHat-Fusion
- Annotation GTF file
- For circ_quant: CIRCexplorer2 circRNA output file, HISAT2 mapped BAM (unmapped reads removed), refFlat gene annotation

## Outputs

- Quantification table output_dir/quant/quant.txt with fields chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore
- CIRCexplorer2-based circRNA annotations with BSJ coordinates used for FPBcirc computation

## How to apply

Retrieve paired-end ribo− RNA-seq FASTQ (e.g. PA1 mate_1/mate_2 from GSE75733/GSE73325) and run clear_quant, which aligns reads to hg38 with HISAT2 (>=2.0.5), realigns unmapped reads through TopHat-Fusion via the hg38 Bowtie1 index, and relies on CIRCexplorer2 (>=2.3.6) for circRNA annotation; the hisat_aligned.bam must contain only mapped reads. To quantify from an intermediate, run circ_quant with the CIRCexplorer2 circRNA file, the HISAT2 BAM, and a refFlat annotation, applying the default FPB threshold of 1 to filter linear SJ and the default circ-to-linear ratio of 1 (optionally -l to consider all reads' lengths). Both circular (BSJ-mapped) and linear (SJ-mapped) fragments are normalized to FPB, making them directly comparable because FPM and FPKM are unscaled and incomparable. CIRCscore is computed as FPBcirc divided by FPBlinear, using the cognate linear RNA expression as background to flag highly-expressed circRNAs with low linear background. Results are written to output_dir/quant/quant.txt.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline providing clear_quant and circ_quant commands for FPB quantification of circRNA BSJ and linear SJ fragments and CIRCscore computation) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Annotates circRNAs from TopHat-Fusion realigned BAM; its output file is the input to circ_quant (>=2.3.6 required)) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligns ribo− RNA-seq reads to hg38 to produce the mapped BAM used for linear SJ fragment quantification (>=2.0.5 required)) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **pysam** (Python package dependency (>=0.8.4) used by CLEAR for BAM handling) — http://pysam.readthedocs.org/en/latest/
- **pybedtools** (Python package dependency used by CLEAR for genomic interval operations) — http://daler.github.io/pybedtools/

## Examples

```
circ_quant -c CIRCexplorer2_output.txt -b hisat_aligned.bam -r annotation.refFlat -o quant.txt
```

## Evaluation signals

- quant.txt exists at output_dir/quant/ and contains the schema fields chrom, start, end, strand, readNumber, geneName, FPBcirc, FPBlinear, and CIRCscore
- For every locus, CIRCscore equals FPBcirc divided by FPBlinear (with the default --ratio of 1)
- hisat_aligned.bam contains no unmapped reads
- CIRCexplorer2 version >=2.3.6 and HISAT2 version >=2.0.5 were used
- Highly-expressed circRNAs with low cognate linear RNA expression appear as high CIRCscore values and are recoverable for further investigation

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, which is unsuitable for accurate linear RNA quantification
- FPM (circRNA) is unscaled to FPKM (linear RNA), so pre-CLEAR quantification approaches are not directly comparable — use FPB instead
- Installation requires Python 2.7+ with pysam (>=0.8.4) and pybedtools, plus StringTie (>1.3.6)

## Evidence

- [abstract] A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually by fragments mapped to circRNA-specific BSJ sites"
- [abstract] CIRCscore definition: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [intro] FPM/FPKM incomparability rationale: "Since FPM is unscaled to FPKM, the relative expression levels of most circRNAs are not comparable to those of their cognate linear RNAs when analyzing RNA-seq datasets"
- [readme] clear_quant usage from README: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir"
- [readme] circ_quant BAM requirement: "*hisat_aligned.bam should not contain unmapped reads.*"
- [methods] Default thresholds: "Threshold of FPB for choose circRNAs to filter linear
                        SJ.[default: 1]"
