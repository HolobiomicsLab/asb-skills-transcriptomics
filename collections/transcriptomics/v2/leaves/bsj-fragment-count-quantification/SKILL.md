---
name: bsj-fragment-count-quantification
description: Use when quantifying circRNAs by back-splicing junction (BSJ)-supporting
  fragment counts from ribo− RNA-seq, or when cross-comparing circRNA call sets from
  CIRCexplorer2/CLEAR, MapSplice, and DCC under a >= 3 BSJ-fragment filter.
when_to_use_negative:
- 'Input is poly(A)+ selected RNA-seq: circRNAs lack 3′-end poly(A) tails and are
  depleted in poly(A)+ datasets, so BSJ-fragment quantification will under-detect.'
- 'Dataset is poly(A)−/ribo− or RNase R-treated and a CIRCscore is wanted: polyadenylated
  linear RNAs are largely depleted, making FPBlinear and CIRCscore unreliable.'
- 'The BAM handed to DCC or circ_quant still contains unmapped reads: the pipeline
  requires a mapped-reads-only HISAT2 BAM.'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline wrapper combining HISAT2 alignment and CIRCexplorer2 calling; emits
    quant.txt whose readNumber field is the BSJ junction-read count used for the >=3
    filter and FPB/CIRCscore quantification
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: CircRNA caller (>=2.3.6) invoked inside clear_quant to detect back-splicing
    junctions and report per-locus readNumber
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Splice-aware aligner (>=2.0.5) producing the mapped BAM from which BSJ fragments
    are counted and on which circtools/DCC is run
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: MapSplice
  role: Independent circRNA caller applied to the same PA1 ribo− FASTQ files to build
    the second BSJ call set with the same >=3 fragment filter
- name: DCC/circtools
  role: Third circRNA detector run on the unmapped-read-free HISAT2 BAM; its comparatively
    inefficient BSJ fragment calling is the benchmark outcome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:d2819baa18d0c40bc98883e6df35147f63974b47175609608deba6eb54beb248
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - DCC/circtools
  - HISAT2
  - MapSplice
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# bsj-fragment-count-quantification

## Summary

Quantifies circular RNAs by counting RNA-seq fragments mapped to back-splicing junction (BSJ) sites — the readNumber metric of CIRCexplorer2/CLEAR — and cross-compares BSJ-based circRNA calls from CIRCexplorer2, MapSplice, and circtools-embedded DCC on ribo− RNA-seq to expose tool-specific detection inefficiencies.

## When to use

Use when quantifying circRNAs by back-splicing junction (BSJ)-supporting fragment counts from ribo− RNA-seq, or when cross-comparing circRNA call sets from CIRCexplorer2/CLEAR, MapSplice, and DCC under a >= 3 BSJ-fragment filter.

## When NOT to use

- Input is poly(A)+ selected RNA-seq: circRNAs lack 3′-end poly(A) tails and are depleted in poly(A)+ datasets, so BSJ-fragment quantification will under-detect.
- Dataset is poly(A)−/ribo− or RNase R-treated and a CIRCscore is wanted: polyadenylated linear RNAs are largely depleted, making FPBlinear and CIRCscore unreliable.
- The BAM handed to DCC or circ_quant still contains unmapped reads: the pipeline requires a mapped-reads-only HISAT2 BAM.

## Inputs

- Paired-end ribo− (ribosomal-RNA depleted) RNA-seq FASTQ files, e.g. PA1 ribo− mate_1.fastq / mate_2.fastq
- hg38 genome FASTA (hg38.fa), HISAT2 index (hg38.hisat_index), bowtie1 index for TopHat-Fusion (hg38.bowtie_index), and annotation GTF (annotation.gtf)
- HISAT2 alignment BAM with unmapped reads removed (required by circ_quant and circtools/DCC)
- CIRCexplorer2 circRNA output file and refFlat-format annotation (annotation.refFlat) for the circ_quant entry point
- MapSplice and circtools/DCC circRNA call lists produced on the same samples

## Outputs

- CLEAR quantification table (output_dir/quant/quant.txt) with per-circRNA readNumber (junction reads), FPBcirc, FPBlinear and CIRCscore
- Per-tool circRNA call sets filtered to >= 3 BSJ-supporting fragments, keyed by chrom, start, end, strand
- Venn-style pairwise and three-way overlap counts plus per-circRNA per-tool BSJ fragment counts written to circrna_tool_overlap_counts.tsv

## How to apply

Install CLEAR from https://github.com/YangLab/CLEAR (git clone; python ./setup.py install; requires CIRCexplorer2 >=2.3.6, HISAT2 >=2.0.5, StringTie >1.3.6, pysam >=0.8.4, pybedtools). Run clear_quant on paired PA1 ribo− FASTQ files with the hg38 genome FASTA, HISAT2 index, bowtie1 (TopHat-Fusion) index, and annotation GTF, then take the readNumber field (number of junction reads) from output_dir/quant/quant.txt as the per-locus BSJ fragment count. Independently call circRNAs on the same FASTQs with MapSplice and with circtools-embedded DCC run on the HISAT2 BAM, which must not contain unmapped reads. Apply the same >= 3 BSJ-supporting fragments (readNumber >= 3) filter to all three call sets before any comparison. Intersect the filtered sets by exact genomic coordinates (chrom, start, end, strand) to obtain pairwise and three-way overlaps, expecting the Figure 3B pattern of 356 circRNAs detected by both CIRCexplorer2 and MapSplice but missed by DCC and 787 three-way shared circRNAs. For the shared loci, tabulate per-tool readNumber to show DCC's reduced BSJ fragment calling (Figure 3C) and write the overlap counts and per-circRNA per-tool counts to circrna_tool_overlap_counts.tsv.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline wrapper combining HISAT2 alignment and CIRCexplorer2 calling; emits quant.txt whose readNumber field is the BSJ junction-read count used for the >=3 filter and FPB/CIRCscore quantification) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (CircRNA caller (>=2.3.6) invoked inside clear_quant to detect back-splicing junctions and report per-locus readNumber) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Splice-aware aligner (>=2.0.5) producing the mapped BAM from which BSJ fragments are counted and on which circtools/DCC is run) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **MapSplice** (Independent circRNA caller applied to the same PA1 ribo− FASTQ files to build the second BSJ call set with the same >=3 fragment filter)
- **DCC/circtools** (Third circRNA detector run on the unmapped-read-free HISAT2 BAM; its comparatively inefficient BSJ fragment calling is the benchmark outcome)

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- The overlap table contains 356 in the CIRCexplorer2-and-MapSplice-only intersection and 787 in the three-way overlap, matching Figure 3B
- All three call sets are filtered to readNumber / BSJ-supporting fragments >= 3 before intersecting, and intersections are keyed on (chrom, start, end, strand)
- Per-circRNA per-tool readNumber table shows systematically reduced DCC BSJ fragment counts among the 787 shared circRNAs (Figure 3C)
- quant.txt parses with the documented fields (readNumber, FPBcirc, FPBlinear, CIRCscore) and readNumber equals the number of junction reads
- DCC was run on a HISAT2 BAM verified to contain no unmapped reads

## Limitations

- hisat_aligned.bam should not contain unmapped reads — DCC and circ_quant inputs must be mapped-reads-only
- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear quantification inaccurate
- Detection is tool-dependent: 356 circRNAs supported by both CIRCexplorer2 and MapSplice were missed by DCC, and DCC inefficiently calls BSJ-mapped fragments among the 787 shared circRNAs
- Installation requires CIRCexplorer2 >=2.3.6, HISAT2 >=2.0.5, StringTie >1.3.6, pysam >=0.8.4 and pybedtools

## Evidence

- [abstract] circRNA profiling relies on detecting RNA-seq fragments spanning BSJ sites: "examination of global circRNA expression from RNA-seq datasets generally relies on the detection of RNA-seq fragments spanning BSJ sites"
- [abstract] FPB normalizes fragments mapped to BSJ and SJ sites individually: "A new quantitation parameter, fragments per billion mapped bases (FPB), is applied to evaluate circular and linear RNA expression individually"
- [discussion] CLEAR quantification is based solely on fragments mapped to BSJ or canonical SJ sites: "we have developed CLEAR by applying normalized RNA-seq fragments solely mapped to BSJ or canonical SJ sites individually"
- [methods] clear_quant invocation for paired-end ribo− FASTQ with hg38 references: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [methods] CIRCexplorer2 version requirement for the CLEAR pipeline: "[CIRCexplorer2](https://github.com/YangLab/CIRCexplorer2/tree/master/circ2) (>=2.3.6)"
- [readme] BAM input to quantification must exclude unmapped reads: "hisat_aligned.bam should not contain unmapped reads."
- [other] Minimum >=3 BSJ-supporting fragments filter applied to each tool's calls: [evidence span withheld — anchor status: not_found]
- [other] Figure 3B overlap counts the skill must reproduce: [evidence span withheld — anchor status: not_found]
