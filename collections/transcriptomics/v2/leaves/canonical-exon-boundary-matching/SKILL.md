---
name: canonical-exon-boundary-matching
description: Use when running INTEGRATE-Circ fusion detection and you must decide
  whether spanning-read breakpoints represent canonical exon-boundary junctions versus
  non-canonical (alternative-splicing/backsplice) junctions for a gene fusion.
when_to_use_negative:
- When the goal is de novo fusion detection without reference transcript annotation
  — the canonical/non-canonical decision depends on an Ensembl annotation file
- When analyzing fusions whose breakpoints are intentionally far from annotated exon
  boundaries and you do not want them suppressed by the -cfn spanning-read cutoff
- When visualizing already-called fusions — that is the role of INTEGRATE-Vis, not
  of this junction-classification step
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: INTEGRATE-Circ
  role: Fusion junction detection tool whose -bacc and -cfn parameters implement canonical
    exon-boundary matching and non-canonical filtering
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: INTEGRATE-Vis
  role: Downstream consumer of fcirc.txt output for visualization of detected fusion-derived
    circRNAs
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
- name: cmake
  role: Build dependency for compiling INTEGRATE-Circ (requires cmake version >= 2.8)
- name: g++
  role: Compiler for INTEGRATE-Circ; older compilers fail, recommend version >12
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:ac929d17d74594052f5cdf3c65fd3de8a51a1ecb4bbc73871b42f6db7bae175c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE-Circ
  - INTEGRATE-Vis
  - cmake
  - g++
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# canonical-exon-boundary-matching

## Summary

Classifies spanning RNA-Seq read clusters supporting a candidate fusion by comparing their breakpoints against an Ensembl annotation: junctions falling within a tolerated distance (-bacc) of annotated exon boundaries are treated as canonical, while non-canonical breakpoints are filtered by a separate spanning-read cutoff (-cfn). This anchoring step is what lets INTEGRATE-Circ designate a primary fusion junction and classify remaining clusters as secondary (alternative-splicing/backsplice) junctions.

## When to use

Use when running INTEGRATE-Circ fusion detection and you must decide whether spanning-read breakpoints represent canonical exon-boundary junctions versus non-canonical (alternative-splicing/backsplice) junctions for a gene fusion.

## When NOT to use

- When the goal is de novo fusion detection without reference transcript annotation — the canonical/non-canonical decision depends on an Ensembl annotation file
- When analyzing fusions whose breakpoints are intentionally far from annotated exon boundaries and you do not want them suppressed by the -cfn spanning-read cutoff
- When visualizing already-called fusions — that is the role of INTEGRATE-Vis, not of this junction-classification step

## Inputs

- Reference genome FASTA (e.g. hg19.chr21.fa)
- Ensembl annotation file (e.g. hg19.chr21.annotation.txt)
- BWT index directory produced by 'Integrate-Circ mkbwt'
- RNA-Seq reads in BAM format (mapped and unmapped reads merged in one file, passed twice on the command line)

## Outputs

- summary.tsv (primary and secondary junctions, backsplices)
- exons.tsv
- breakpoints.tsv
- bk_sv.vcf
- junctions.bedpe
- reads.txt
- fcirc.txt

## How to apply

Run 'Integrate-Circ fusion' with the reference FASTA (e.g. hg19.chr21.fa), the Ensembl annotation file (e.g. hg19.chr21.annotation.txt), the BWT directory produced by 'mkbwt', and the RNA read BAM supplied twice (mapped and unmapped reads share one file). With default parameters (-bacc 1, -cfn 3, -minW 2.0, -minDel 5000, -largeNum 4), the tool computes the maximum difference between each spanning read's breakpoint and the annotation: breakpoints within -bacc of an annotated exon boundary are called canonical and preferred over raw genomic base-pair coordinates; non-canonical fusions must instead clear the -cfn spanning-read cutoff (default 3). Once junction clusters are classified this way, the highest-read-support cluster becomes the primary fusion junction, and all other spanning read clusters are evaluated relative to that primary junction as secondary junctions arising from alternative splicing. Default output lands in INTEGRATE_Circ_output, where summary.tsv records primary junctions, secondary junctions, and backsplices so the canonical/non-canonical assignments can be inspected.

## Related tools

- **INTEGRATE-Circ** (Fusion junction detection tool whose -bacc and -cfn parameters implement canonical exon-boundary matching and non-canonical filtering) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **INTEGRATE-Vis** (Downstream consumer of fcirc.txt output for visualization of detected fusion-derived circRNAs) — https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
- **cmake** (Build dependency for compiling INTEGRATE-Circ (requires cmake version >= 2.8))
- **g++** (Compiler for INTEGRATE-Circ; older compilers fail, recommend version >12)

## Examples

```
Integrate-Circ mkbwt example-data/hg19.chr21.fa -dir example-data/bwts && Integrate-Circ fusion -f example-data/hg19.chr21.fa -a example-data/hg19.chr21.annotation.txt -b example-data/bwts -r RNA.Example.AllReads.bam -u RNA.Example.AllReads.bam
```

## Evaluation signals

- summary.tsv exists in INTEGRATE_Circ_output and contains an entry for the simulated TMPRSS2::ERG fusion with its primary junction
- summary.tsv contains associated splice-variant/backsplice annotations for the primary junction, verifiable via contains_substring checks
- Canonical junctions are reported at annotation coordinates (within -bacc of exon boundaries) rather than raw genomic base-pair offsets
- Non-canonical junction candidates have at least -cfn (default 3) spanning RNA-Seq reads or are absent from the output
- junctions.bedpe and bk_sv.vcf are produced and parse as valid BEDPE/VCF respectively

## Limitations

- INTEGRATE-Circ cannot be compiled by some older compilers; g++ version >12 is recommended, requiring an explicit CMAKE_CXX_COMPILER override if not default
- Canonical-vs-non-canonical decisions depend on the supplied Ensembl annotation; breakpoints absent from the annotation need to clear the -cfn cutoff or are dropped
- Genes showing more than -largeNum (default 4) spanning-support events are removed from results, which may drop highly rearranged loci

## Evidence

- [methods] -bacc     integer : max difference between spanning reads and annotation to decide canonical: "-bacc     integer : max difference between spanning reads and annotation to decide canonical"
- [methods] -cfn cutoff of spanning RNA-Seq reads for fusions with non-canonical exonic boundaries: "-cfn      integer : Cutoff of spanning RNA-Seq reads for fusions with non-canonical exonic boundaries"
- [readme] INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms: "INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA)."
- [readme] two input files are required for INTEGRATE-Circ: reference genome fasta file and Ensembl annotation file: "Besides user-provided sequencing data, two input files are required for INTEGRATE-Circ.

1. Reference genome fasta file
2. Ensembl annotation file"
