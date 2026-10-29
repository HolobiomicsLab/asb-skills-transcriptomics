---
name: fusion-junction-primary-secondary-annotation
description: Use when a fusion caller produces multiple spanning-read clusters per
  candidate fusion and you must decide which junction is primary versus which represent
  alternative-splice or backsplice (fcircRNA) junctions, e.g. annotating INTEGRATE-Circ
  summary.tsv entries for a fusion like TMPRSS2::ERG.
when_to_use_negative:
- You need discovery of fusions or circRNAs de novo — this skill annotates primary
  vs secondary junctions within INTEGRATE-Circ's existing candidate clusters, it is
  not a standalone discovery or visualization method (use INTEGRATE-Vis for plotting
  fcirc.txt results).
- Your junction classification relies on coordinates far from annotated exon boundaries
  under different thresholds — changing -cfn, -bacc, -minW, -minDel, or -largeNum
  alters which junctions survive to be classified.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3512
tools:
- name: INTEGRATE-Circ
  role: fusion junction detection tool whose algorithm designates the highest-read-support
    junction cluster as the primary fusion junction and evaluates remaining clusters
    as secondary (alternative-splicing/backsplice) junctions
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: cmake
  role: build dependency (>= 2.8) for compiling INTEGRATE-Circ before running junction
    classification
- name: g++
  role: compiler for INTEGRATE-Circ (recommend version >12 of g++)
- name: INTEGRATE-Vis
  role: downstream visualization of fcirc.txt output containing primary/secondary
    junction and backsplice classifications
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:f13898a7c3976d25f4364d925542314a9973ed573e0f2ad96e14c0afc38eb49c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE-Circ
  - INTEGRATE-Vis
  - cmake
  - g++
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fusion-junction-primary-secondary-annotation

## Summary

Reconstructs INTEGRATE-Circ's logic for classifying clusters of spanning RNA-Seq reads at a candidate gene fusion: the highest-read-support junction cluster becomes the primary fusion junction, and all remaining clusters are evaluated relative to it as secondary junctions (alternative splicing / backsplice junctions).

## When to use

Use when a fusion caller produces multiple spanning-read clusters per candidate fusion and you must decide which junction is primary versus which represent alternative-splice or backsplice (fcircRNA) junctions, e.g. annotating INTEGRATE-Circ summary.tsv entries for a fusion like TMPRSS2::ERG.

## When NOT to use

- You need discovery of fusions or circRNAs de novo — this skill annotates primary vs secondary junctions within INTEGRATE-Circ's existing candidate clusters, it is not a standalone discovery or visualization method (use INTEGRATE-Vis for plotting fcirc.txt results).
- Your junction classification relies on coordinates far from annotated exon boundaries under different thresholds — changing -cfn, -bacc, -minW, -minDel, or -largeNum alters which junctions survive to be classified.

## Inputs

- RNA-Seq reads (mapped and unmapped merged in one BAM, e.g. RNA.Example.AllReads.bam, passed twice to 'Integrate-Circ fusion')
- Reference genome fasta (e.g. hg19.chr21.fa) with a pre-built Burrows-Wheeler transform (BWT directory)
- Ensembl gene annotation file (e.g. hg19.chr21.annotation.txt)
- Optional tumor and normal DNA BAMs (omitted in the default example workflow)

## Outputs

- INTEGRATE_Circ_output/summary.tsv with primary junction, secondary junctions, and backsplice annotations per fusion
- exons.tsv, breakpoints.tsv, bk_sv.vcf, junctions.bedpe, reads.txt
- fcirc.txt (fusion-derived circRNA calls, usable as input to INTEGRATE-Vis)

## How to apply

Run INTEGRATE-Circ's fusion step on the RNA reads (supplying the reference fasta hg19.chr21.fa, the Ensembl annotation hg19.chr21.annotation.txt, and the BWT directory built with 'Integrate-Circ mkbwt'), merging mapped and unmapped reads from one BAM by passing it twice. Use default parameters (-cfn 3, -bacc 1, -minW 2.0, -minDel 5000, -largeNum 4) so junctions at canonical exon boundaries — within -bacc of the annotation — are preferred over genomic base-pair coordinates. The cluster of spanning RNA-Seq reads with the highest read support is designated the primary fusion junction; every other spanning-read cluster is then assessed relative to that primary junction and treated as a secondary junction arising from alternative splicing, including fusion-derived backsplices. The classification is emitted in INTEGRATE_Circ_output/summary.tsv alongside exons.tsv, breakpoints.tsv, bk_sv.vcf, junctions.bedpe, reads.txt, and fcirc.txt.

## Related tools

- **INTEGRATE-Circ** (fusion junction detection tool whose algorithm designates the highest-read-support junction cluster as the primary fusion junction and evaluates remaining clusters as secondary (alternative-splicing/backsplice) junctions) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **cmake** (build dependency (>= 2.8) for compiling INTEGRATE-Circ before running junction classification)
- **g++** (compiler for INTEGRATE-Circ (recommend version >12 of g++))
- **INTEGRATE-Vis** (downstream visualization of fcirc.txt output containing primary/secondary junction and backsplice classifications) — https://www.github.com/ChrisMaherLab/INTEGRATE-Vis

## Examples

```
Integrate-Circ mkbwt example-data/hg19.chr21.fa -dir example-data/bwts; Integrate-Circ fusion -f example-data/hg19.chr21.fa -a example-data/hg19.chr21.annotation.txt -b example-data/bwts -r example-data/RNA.Example.AllReads.bam -u example-data/RNA.Example.AllReads.bam
```

## Evaluation signals

- summary.tsv exists in the INTEGRATE_Circ_output directory and contains entries for the simulated TMPRSS2::ERG fusion (file_exists / contains_substring checks).
- summary.tsv lists one primary junction per fusion plus associated splice-variant/backsplice (secondary junction) annotations.
- All expected companion outputs (exons.tsv, breakpoints.tsv, bk_sv.vcf, junctions.bedpe, reads.txt, fcirc.txt) are present in INTEGRATE_Circ_output.
- Primary junction corresponds to the spanning-read cluster with the highest read support; secondary junctions are consistent with alternative-splicing relative to that primary junction.
- Canonical-boundary junction calls fall within -bacc (= 1) of the Ensembl annotation rather than being reported as raw genomic breakpoints.

## Limitations

- No version number is stated for INTEGRATE-Circ in the methods; behavior may vary across repository commits.
- Older compilers fail to build the tool (recommend g++ > 12); cmake >= 2.8 is required.
- Gene candidates showing greater than or equal to the -largeNum count (default 4) are removed from results, which can drop heavily promiscuous genes.
- The classification of secondary junctions as alternative splicing is an interpretation made relative to the primary junction and depends on simulated/example-data validation (TMPRSS2::ERG, hg19.chr21).

## Evidence

- [methods] INTEGRATE-Circ designates the highest-read-support junction cluster as the primary fusion junction and then evaluates all other spanning read clusters relative to that primary junction: [evidence span withheld — anchor status: not_found]
- [intro] INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms: "INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions"
- [methods] summary.tsv exists and contains entries for the simulated TMPRSS2::ERG fusion with its primary junction and associated splice-variant/backsplice annotations: [evidence span withheld — anchor status: not_found]
- [methods] the output fcirc.txt file can be supplied as input to the most recent version of the INTEGRATE-vis tool: "the output fcirc.txt file can be supplied as input to the most recent version of the [INTEGRATE-vis tool developed by the Chris Maher Lab]"
- [readme] INTEGRATE-Circ cannot be compiled by some older compilers (recommend version >12 of g++): "Note that INTEGRATE-Circ cannot be compiled by some older compilers (recommend version >12 of g++)"
