---
name: small-rna-read-count-quantification
description: Use when STAR-aligned BAM files from small RNA-seq (e.g. hg38-mapped
  qualified reads) need to be converted into per-molecule read counts for six small
  RNA classes, or when deciding whether a read mapping to multiple loci/RNA types
  should be counted once or per RNA.
when_to_use_negative:
- The reads were not mapped to the human genome — those qualified unmapped reads belong
  to the Microbe module (BLAST against the UCSC nt database), not Annotation.
- The goal is raw adapter-trimmed/quality-filtered read production or genome alignment
  — use the QC (-qc) and Alignment (-aln) modules upstream first.
- Differential expression or count-matrix aggregation across samples is needed directly
  — use the Function module (-fun -fd / -fm) on the annotation count outputs.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: COMPSRA
  role: Provides the Annotation module (-ann -ac 1,2,3,4,5,6) with pre-built, expandable
    small RNA databases, LiftOver-based genome-version harmonization, and per-RNA-type
    count output
  repo: https://github.com/cougarlj/COMPSRA
- name: Java
  role: Runtime environment (JRE 8 or up) required to execute COMPSRA.jar for annotation
- name: STAR
  role: Default aligner producing the genome-mapped BAM (STAR_Aligned.out.bam) that
    serves as the annotation input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:a9ceaaef6df4e94d1a95f4c821870b6c6a6f0df4c24df138a1062e28851a4bda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# small-rna-read-count-quantification

## Summary

Quantify small RNA molecules from human genome-mapped small RNA-seq reads by annotating them against multiple pre-built reference databases (miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE release 27, circBase) and outputting per-RNA-type read count tables. This is the core COMPSRA Annotation module behavior for profiling miRNA, piRNA, tRNA, snRNA, snoRNA and circRNA.

## When to use

Use when STAR-aligned BAM files from small RNA-seq (e.g. hg38-mapped qualified reads) need to be converted into per-molecule read counts for six small RNA classes, or when deciding whether a read mapping to multiple loci/RNA types should be counted once or per RNA.

## When NOT to use

- The reads were not mapped to the human genome — those qualified unmapped reads belong to the Microbe module (BLAST against the UCSC nt database), not Annotation.
- The goal is raw adapter-trimmed/quality-filtered read production or genome alignment — use the QC (-qc) and Alignment (-aln) modules upstream first.
- Differential expression or count-matrix aggregation across samples is needed directly — use the Function module (-fun -fd / -fm) on the annotation count outputs.

## Inputs

- STAR alignment output BAM of human-genome-mapped qualified reads (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam)
- Pre-built annotation databases: miRBase (miRNA), piRNABank/piRBase/piRNACluster (piRNA), gtRNAdb (tRNA), GENCODE release 27 (snRNA/snoRNA), circBase (circRNA)
- Annotation class selection via -ac (1,2,3,4,5,6 for miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA)

## Outputs

- Tab-delimited text file of read counts per annotated RNA molecule, one output per RNA type (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA)

## How to apply

Load the STAR alignment output BAM (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam) and run the COMPSRA Annotation module (java -jar COMPSRA.jar -ann -ac 1,2,3,4,5,6) against the pre-built databases: miRBase for miRNA, piRNABank/piRBase/piRNACluster for piRNA, gtRNAdb for tRNA, GENCODE release 27 for snRNA and snoRNA, and circBase for circular RNA. Apply the degree-of-read-support filter with default overlap ratio Rolp = 1 (-aol 1.0), requiring each read to lie completely within the annotated gene region. Harmonize differing reference genome versions (hg18/hg19/hg38) across databases using an automatic LiftOver from the UCSC Genome Browser Group. Count each read once when it maps to multiple genomic loci of one RNA, and assign one count to each overlapping small RNA when distinct RNAs share sequence (e.g. let-7a-5p vs piR_008113), so reads are not restricted to a single RNA type. Emit the read count of each annotated RNA molecule as a tab-delimited text file per RNA type.

## Related tools

- **COMPSRA** (Provides the Annotation module (-ann -ac 1,2,3,4,5,6) with pre-built, expandable small RNA databases, LiftOver-based genome-version harmonization, and per-RNA-type count output) — https://github.com/cougarlj/COMPSRA
- **Java** (Runtime environment (JRE 8 or up) required to execute COMPSRA.jar for annotation)
- **STAR** (Default aligner producing the genome-mapped BAM (STAR_Aligned.out.bam) that serves as the annotation input)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -ann -ac 1,2,3,4,5,6 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam -out ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned
```

## Evaluation signals

- Tab-delimited annotation output files exist for all six RNA classes (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) and contain integer read counts per RNA molecule.
- A read mapping to multiple genomic loci of the same RNA contributes exactly one count (not one per locus).
- A read overlapping distinct RNAs sharing sequence (e.g. let-7a-5p vs piR_008113) appears in the count table of each overlapping RNA.
- All annotated reads satisfy the overlap-ratio filter with Rolp = 1: each read lies completely within the annotated gene region.
- Pre-built databases were auto-harmonized to the selected reference genome (hg19 or hg38) via internal LiftOver without manual coordinate conversion.

## Limitations

- The default overlap ratio Rolp = 1 (-aol 1.0) discards reads that only partially overlap an annotated RNA region, potentially losing fragment-derived counts.
- Only pre-built reference versions are supported out of the box (hg19 and hg38 annotation databases); other genome versions require expandable/custom database setup.
- Annotation covers the six supported small RNA classes plus downstream microbial profiling separately — fragmental microbial RNA annotation depends on the Microbe module, not Annotation.

## Evidence

- [methods] COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA"
- [methods] piRNABank21, piRBase22 and piRNACluster23 for piRNA, gtRNAdb24 for tRNA, GENCODE release 2725 for snRNA and snoRNA and circBase26 for circular RNA: [evidence span withheld — anchor status: not_found]
- [readme] To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group.: "To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group."
- [readme] The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type.: "The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type."
- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea"
