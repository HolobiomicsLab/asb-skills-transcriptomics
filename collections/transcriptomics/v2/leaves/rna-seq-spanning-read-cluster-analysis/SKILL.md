---
name: rna-seq-spanning-read-cluster-analysis
description: Use when RNA-Seq read support at a fusion locus forms multiple spanning-read
  junction clusters and one must decide which cluster is the primary fusion junction
  versus alternative-splice or backsplice junctions.
when_to_use_negative:
- The goal is general (non-fusion) circular RNA detection from back-splice alignments
  rather than fusion-derived circRNA discovery at gene fusion breakpoints
- Only short single-end reads without spanning-read support at candidate junctions
  are available, so no junction cluster can be designated as primary
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3522
tools:
- name: INTEGRATE-Circ
  role: Fusion junction detection tool whose clustering of spanning RNA-Seq reads
    assigns the highest-support cluster as the primary junction and the rest as secondary
    splice-variant/backsplice junctions
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: INTEGRATE-Vis
  role: Consumes the fcirc.txt output to visualize the detected fusion-derived circRNAs
    and their junction structure
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
- name: cmake
  role: Build dependency (>= 2.8) for compiling the Integrate-Circ executable
- name: g++
  role: Compiler required (recommend version >12) to build INTEGRATE-Circ
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:0dc3be034b7d616c4e7691130143b5839ca9092711705c336525b23b1edfc237
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE-Circ
  - INTEGRATE-Vis
  - cmake
  - g++
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rna-seq-spanning-read-cluster-analysis

## Summary

Classifies clusters of spanning RNA-Seq reads at candidate gene fusion loci into a single primary fusion junction (highest read support) and secondary junctions (interpreted as alternative splicing or backsplice events), as implemented in INTEGRATE-Circ for detecting fcircRNAs.

## When to use

Use when RNA-Seq read support at a fusion locus forms multiple spanning-read junction clusters and one must decide which cluster is the primary fusion junction versus alternative-splice or backsplice junctions.

## When NOT to use

- The goal is general (non-fusion) circular RNA detection from back-splice alignments rather than fusion-derived circRNA discovery at gene fusion breakpoints
- Only short single-end reads without spanning-read support at candidate junctions are available, so no junction cluster can be designated as primary

## Inputs

- RNA-Seq alignments in BAM format (mapped and unmapped reads; the same BAM is passed twice to 'Integrate-Circ fusion')
- Reference genome FASTA (e.g. hg19.chr21.fa) with a precomputed BWT index directory
- Ensembl gene annotation file (e.g. hg19.chr21.annotation.txt)
- Optional tumor and normal DNA BAMs for germline/somatic filtering

## Outputs

- summary.tsv listing primary and secondary junctions and backsplices per fusion
- fcirc.txt (fusion-derived circRNA calls, usable as INTEGRATE-Vis input)
- exons.tsv, breakpoints.tsv, bk_sv.vcf, junctions.bedpe, and reads.txt in the INTEGRATE_Circ_output directory

## How to apply

Build a Burrows-Wheeler transform of the reference genome (e.g. hg19.chr21.fa via 'Integrate-Circ mkbwt') once, then run the 'fusion' step on the RNA BAM (supplied twice, once as mapped and once as unmapped reads, since INTEGRATE-Circ merges them). Cluster spanning RNA-Seq reads at candidate breakpoints and designate the cluster with the highest read support as the primary fusion junction. Evaluate every other spanning-read cluster relative to that primary junction and label those secondary junctions as arising from alternative splicing, including backsplices that produce fcircRNAs. Keep default fusion parameters (-cfn 3, -bacc 1, -minW 2.0, -minDel 5000, -largeNum 4) so junctions within -bacc of canonical exon boundaries in the Ensembl annotation are preferred over raw genomic coordinates. Inspect summary.tsv, where primary and secondary junctions and backsplices are reported per fusion.

## Related tools

- **INTEGRATE-Circ** (Fusion junction detection tool whose clustering of spanning RNA-Seq reads assigns the highest-support cluster as the primary junction and the rest as secondary splice-variant/backsplice junctions) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **INTEGRATE-Vis** (Consumes the fcirc.txt output to visualize the detected fusion-derived circRNAs and their junction structure) — https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
- **cmake** (Build dependency (>= 2.8) for compiling the Integrate-Circ executable)
- **g++** (Compiler required (recommend version >12) to build INTEGRATE-Circ)

## Examples

```
Integrate-Circ fusion hg19.chr21.fa example-data/hg19.chr21.annotation.txt -bwt example-data/bwts -rna RNA.Example.AllReads.bam -umrna RNA.Example.AllReads.bam
```

## Evaluation signals

- summary.tsv exists in INTEGRATE_Circ_output and contains an entry for the simulated TMPRSS2::ERG fusion with a designated primary junction
- Secondary junctions and backsplice annotations appear in summary.tsv associated with the primary fusion junction
- Junction coordinates at canonical exon boundaries fall within -bacc (default 1) of the Ensembl annotation rather than arbitrary genomic positions
- fcirc.txt is produced and can be supplied to INTEGRATE-Vis without format errors
- The cluster designated as primary has read support at least as high as every secondary junction cluster at the same locus

## Limitations

- Secondary junctions are interpreted as alternative-splicing/backsplice events relative to the primary junction; this interpretation depends on the primary cluster having the highest read support
- INTEGRATE-Circ cannot be compiled by some older compilers (recommend g++ > 12)
- No version number is stated for INTEGRATE-Circ in the manuscript or README
- The BWT index must be rebuilt per reference genome, though only once per reference

## Evidence

- [intro] INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA).: "INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions"
- [methods] -bacc max difference between spanning reads and annotation to decide canonical: "-bacc     integer : max difference between spanning reads and annotation to decide canonical"
- [methods] Build BWT of the reference genome (mkbwt): "INTEGRATE-Circ must perform a BWT (Burrows-Wheeler transform) of the reference genome"
- [methods] Visualize fcircRNA with INTEGRATE-Vis: "the output fcirc.txt file can be supplied as input to the most recent version of the [INTEGRATE-vis tool developed by the Chris Maher Lab]"
- [methods] The example data includes simulated reads consistent with a TMPRSS2::ERG gene fusion that also creates an fcircRNA.: "The example data includes simulated reads consistent with a TMPRSS2::ERG gene fusion that also creates an fcircRNA."
