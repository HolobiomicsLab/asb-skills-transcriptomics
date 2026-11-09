---
name: multi-mapping-read-count-disambiguation
description: Use when quantifying small RNA-seq reads (miRNA/piRNA/tRNA/snRNA/snoRNA/circRNA)
  where reads map to multiple loci of one RNA or shared sequences create cross-RNA-type
  ambiguity and per-RNA read counts are the output.
when_to_use_negative:
- You need a single exclusive RNA-type assignment per read (this scheme deliberately
  assigns one count to each overlapping RNA, not a unique best annotation)
- Quantifying RNA classes outside COMPSRA's supported set (miRNA, piRNA, snRNA, snoRNA,
  tRNA, circRNA, microbial RNA) or non-human genomes other than the supported human
  hg19/hg38 and rat rno5/rno6 references
- You require partial-overlap annotations — the default Rolp = 1 (-aol 1.0) demands
  the read lie completely within the annotated gene region
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: COMPSRA
  role: Annotation module applies the multi-mapping counting rule and overlap filter
    (-aol) to STAR-aligned BAM input against pre-built small RNA databases
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default aligner producing the multi-mapping alignments of qualified reads
    that the counting rule disambiguates
- name: Java
  role: Runtime environment (JRE 8+) in which COMPSRA.jar executes the annotation/counting
    step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:c33d2bd9d2a03d8b9c10ddcc82316609c8772697d268d9aaff2de29531b944f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Multi-mapping read count disambiguation

## Summary

COMPSRA's Annotation module rule for counting small RNA-seq reads without double-counting: a read mapping to multiple genomic loci of the same RNA contributes one count to that RNA, while a read sharing sequence with distinct RNAs (e.g. let-7a-5p vs piR_008113) contributes one count to each overlapping RNA.

## When to use

Use when quantifying small RNA-seq reads (miRNA/piRNA/tRNA/snRNA/snoRNA/circRNA) where reads map to multiple loci of one RNA or shared sequences create cross-RNA-type ambiguity and per-RNA read counts are the output.

## When NOT to use

- You need a single exclusive RNA-type assignment per read (this scheme deliberately assigns one count to each overlapping RNA, not a unique best annotation)
- Quantifying RNA classes outside COMPSRA's supported set (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA, microbial RNA) or non-human genomes other than the supported human hg19/hg38 and rat rno5/rno6 references
- You require partial-overlap annotations — the default Rolp = 1 (-aol 1.0) demands the read lie completely within the annotated gene region

## Inputs

- STAR alignment output BAM of hg38-mapped qualified reads (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam)
- Pre-built COMPSRA small RNA annotation databases: miRNA_hg38, piRNA_hg38, tRNA_hg38, snRNA_hg38, snoRNA_hg38, circRNA_hg38 (LiftOver-harmonized across hg18/hg19/hg38)

## Outputs

- Per-sample tab-delimited text files of read counts for each annotated RNA molecule, one file per RNA type (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA)
- Aggregated RNA-by-sample count matrix via the Function module (-fm/-fms)

## How to apply

Run the COMPSRA Annotation module on the STAR-aligned BAM (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam) with -ac 1,2,3,4,5,6 to query all six pre-built reference databases (miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE release 27, circBase). Apply the degree-of-read-support overlap filter with default Rolp = 1 (-aol 1.0), meaning each read must lie completely within the annotated gene region to count. When one read maps to multiple genomic loci belonging to a single RNA, record that RNA once (do not inflate the count per locus). When distinct RNAs share identical sequence (e.g. let-7a-5p overlapping a piR_008113 piRNA), assign one count to each overlapping RNA rather than forcing a single-type assignment, so a read is not restricted to one RNA type. Rationale: this preserves multi-locus quantification integrity while still reporting all plausible annotations across RNA classes, and results are emitted as per-RNA-type tab-delimited count files.

## Related tools

- **COMPSRA** (Annotation module applies the multi-mapping counting rule and overlap filter (-aol) to STAR-aligned BAM input against pre-built small RNA databases) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default aligner producing the multi-mapping alignments of qualified reads that the counting rule disambiguates)
- **Java** (Runtime environment (JRE 8+) in which COMPSRA.jar executes the annotation/counting step)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -ann -ac 1,2,3,4,5,6 -aol 1.0 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam -out ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned
```

## Evaluation signals

- Each output tab-delimited count file contains read counts for each of the six annotated RNA classes (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) with the supporting database items listed per annotation
- Reads mapping to multiple loci of one RNA appear exactly once in that RNA's count (count does not scale with locus multiplicity)
- Sequence-shared RNAs (e.g. let-7a-5p and piR_008113) each receive a count for the shared read rather than one being suppressed
- Only reads fully contained in an annotated gene region (overlap ratio Rolp = 1 with -aol 1.0) are counted; partially overlapping reads are excluded
- Sanity range: on 12 serum samples with read count threshold 5, COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs — counts far outside these orders suggest misconfiguration

## Limitations

- A read is not restricted to a single RNA type; cross-class double counting (e.g. miRNA/piRNA sequence sharing) is inherent to the 'all possible annotations' design
- Reference databases span hg18/hg19/hg38 genome versions and depend on automatic UCSC LiftOver for harmonization, which can introduce coordinate-conversion artifacts
- Pre-built databases are fixed releases (miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE release 27, circBase); the set is expandable but current annotations are dated to the paper's releases

## Evidence

- [methods] Multi-mapping and overlapping annotation policy: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA"
- [other] Default overlap filter: [evidence span withheld — anchor status: not_found]
- [other] Counting rules for multi-loci and shared sequences: [evidence span withheld — anchor status: not_found]
- [readme] All-possible-annotations design (readme): "COMPSRA currently uses several different small RNA databases for annotating human genome mapped reads and provides all the possible annotations: miRBase for miRNA"
- [readme] LiftOver harmonization (readme): "To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group."
