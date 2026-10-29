---
name: circular-rna-backsplice-detection
description: Use when RNA-Seq BAM reads may contain gene fusions and you need to call
  the primary fusion junction plus secondary splice-variant/backsplice junctions and
  fcircRNAs, e.g. sim56-like or TMPRSS2::ERG-style data.
when_to_use_negative:
- The question is general (non-fusion-derived) circRNA detection from RNA-Seq alone
  — INTEGRATE-Circ is a fusion junction detection tool whose fcircRNA calls are tied
  to identified gene fusion junctions.
- You lack a reference genome FASTA plus Ensembl annotation, which are the two required
  input files; the BWT-based workflow cannot run.
- You only need to quantify expression rather than identify fusion junctions and their
  isoforms — this skill performs junction/isoform calling, not quantification.
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: INTEGRATE-Circ
  role: 'Primary detector: clusters spanning RNA-Seq reads, calls the primary fusion
    junction, and annotates secondary junctions/backsplices and fcircRNAs'
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: INTEGRATE-Vis
  role: 'Downstream visualization: consumes the fcirc.txt output to visualize fusion-derived
    circRNAs'
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
- name: cmake
  role: Build dependency for compiling INTEGRATE-Circ (requires cmake version >= 2.8)
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: g++
  role: Compiler for INTEGRATE-Circ; older compilers fail, recommend g++ > 12 via
    -DCMAKE_CXX_COMPILER
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:9295352f931dcc5a2b246012bac1163620b1f15214acd4f85e4294dcebd89fca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE-Circ
  - INTEGRATE-Vis
  - cmake
  - g++
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# circular-rna-backsplice-detection

## Summary

Detects gene fusion transcripts and their isoforms, including fusion-derived circular RNAs (fcircRNAs), from RNA-Seq data by clustering spanning reads at fusion junctions and classifying secondary clusters relative to the highest-read-support primary junction as alternative-splicing/backsplice junctions.

## When to use

Use when RNA-Seq BAM reads may contain gene fusions and you need to call the primary fusion junction plus secondary splice-variant/backsplice junctions and fcircRNAs, e.g. sim56-like or TMPRSS2::ERG-style data.

## When NOT to use

- The question is general (non-fusion-derived) circRNA detection from RNA-Seq alone — INTEGRATE-Circ is a fusion junction detection tool whose fcircRNA calls are tied to identified gene fusion junctions.
- You lack a reference genome FASTA plus Ensembl annotation, which are the two required input files; the BWT-based workflow cannot run.
- You only need to quantify expression rather than identify fusion junctions and their isoforms — this skill performs junction/isoform calling, not quantification.

## Inputs

- Reference genome FASTA (e.g. hg19.chr21.fa)
- Ensembl annotation file (e.g. hg19.chr21.annotation.txt)
- BWT index directory built by 'Integrate-Circ mkbwt'
- RNA-Seq reads as BAM (mapped and unmapped reads, supplied twice if merged in one file, e.g. RNA.Example.AllReads.bam)
- Optional tumor and normal DNA BAMs for -rt normal/tumor DNA ratio filtering

## Outputs

- INTEGRATE_Circ_output/summary.tsv (primary junction, secondary junctions, backsplices)
- INTEGRATE_Circ_output/exons.tsv
- INTEGRATE_Circ_output/breakpoints.tsv
- INTEGRATE_Circ_output/bk_sv.vcf
- INTEGRATE_Circ_output/junctions.bedpe
- INTEGRATE_Circ_output/reads.txt
- INTEGRATE_Circ_output/fcirc.txt (input to INTEGRATE-Vis)

## How to apply

Build the Burrows-Wheeler transform of the reference genome once with 'Integrate-Circ mkbwt' (default -mb 10000000), then run 'Integrate-Circ fusion' with the reference fasta (e.g. hg19.chr21.fa), the Ensembl annotation file (e.g. hg19.chr21.annotation.txt), the BWT directory, and the RNA read file supplied twice as both mapped and unmapped reads when they are merged in a single BAM; omit the optional tumor/normal DNA BAMs unless DNA support filtering is wanted. Run with default fusion parameters (-cfn 3, -bacc 1, -minW 2.0, -minDel 5000, -largeNum 4) so junctions at canonical exon boundaries — within -bacc of the Ensembl annotation — are preferred over raw genomic base-pair coordinates. INTEGRATE-Circ then designates the spanning-read cluster with the highest read support as the primary fusion junction and evaluates every other spanning cluster relative to that primary junction, annotating them as secondary junctions arising from alternative splicing or backsplicing. Default outputs are written to the INTEGRATE_Circ_output directory (summary.tsv, exons.tsv, breakpoints.tsv, bk_sv.vcf, junctions.bedpe, reads.txt, fcirc.txt); fcirc.txt can be fed to INTEGRATE-Vis for fcircRNA visualization.

## Related tools

- **INTEGRATE-Circ** (Primary detector: clusters spanning RNA-Seq reads, calls the primary fusion junction, and annotates secondary junctions/backsplices and fcircRNAs) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **INTEGRATE-Vis** (Downstream visualization: consumes the fcirc.txt output to visualize fusion-derived circRNAs) — https://www.github.com/ChrisMaherLab/INTEGRATE-Vis
- **cmake** (Build dependency for compiling INTEGRATE-Circ (requires cmake version >= 2.8)) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **g++** (Compiler for INTEGRATE-Circ; older compilers fail, recommend g++ > 12 via -DCMAKE_CXX_COMPILER) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC

## Examples

```
Integrate-Circ mkbwt example-data/hg19.chr21.fa -dir example-data/bwts; Integrate-Circ fusion -f example-data/hg19.chr21.fa -a example-data/hg19.chr21.annotation.txt -b example-data/bwts -r example-data/RNA.Example.AllReads.bam -u example-data/RNA.Example.AllReads.bam -o INTEGRATE_Circ_output
```

## Evaluation signals

- summary.tsv exists in INTEGRATE_Circ_output and contains an entry for the expected fusion (e.g. simulated TMPRSS2::ERG) with its primary junction and associated splice-variant/backsplice annotations.
- exons.tsv, breakpoints.tsv, bk_sv.vcf, junctions.bedpe, reads.txt, and fcirc.txt are all present in the default INTEGRATE_Circ_output directory.
- Primary junction is the spanning-read cluster with the highest read support, and all reported secondary junctions are consistent in position with alternative splicing relative to that primary junction.
- Called junctions at canonical exon boundaries fall within -bacc (default 1) of the Ensembl annotation coordinates.
- fcirc.txt entries are accepted by INTEGRATE-Vis without format errors when passed for visualization.

## Limitations

- Some older compilers cannot build INTEGRATE-Circ; g++ version > 12 is recommended and must be passed to cmake explicitly if not default.
- If a gene shows greater than or equal to -largeNum (default 4) calls, it is removed from results — a gene-removal threshold that can drop candidates.
- No version number is stated for INTEGRATE-Circ in the methods/README, so exact behavior may drift with repository updates.
- Mapped and unmapped RNA reads must be handled carefully: when merged in one file, the BAM is supplied twice (mapped and unmapped) to the fusion command.

## Evidence

- [intro] INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA).: "INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA)."
- [abstract] We developed the first unbiased tool for detecting fcircRNAs (INTEGRATE-Circ) and visualizing fcircRNAs (INTEGRATE-Vis) from RNA-Seq data.: "We developed the first unbiased tool for detecting fcircRNAs (INTEGRATE-Circ) and visualizing fcircRNAs (INTEGRATE-Vis) from RNA-Seq data."
- [methods] cfn cutoff of spanning RNA-Seq reads for fusions with non-canonical exonic boundaries: "-cfn      integer : Cutoff of spanning RNA-Seq reads for fusions with non-canonical exonic boundaries"
- [readme] BWT of reference genome prerequisite: "Prior to analyzing sequencing data, INTEGRATE-Circ must perform a BWT (Burrows-Wheeler transform) of the reference genome."
- [methods] fcirc.txt feeds INTEGRATE-Vis: "the output fcirc.txt file can be supplied as input to the most recent version of the [INTEGRATE-vis tool developed by the Chris Maher Lab]"
- [methods] Simulated TMPRSS2::ERG example data: "The example data includes simulated reads consistent with a TMPRSS2::ERG gene fusion that also creates an fcircRNA."
