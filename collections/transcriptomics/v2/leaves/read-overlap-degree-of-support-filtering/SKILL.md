---
name: read-overlap-degree-of-support-filtering
description: Use when annotating genome-mapped small RNA-seq reads against reference
  RNA databases and you must decide which read-gene overlaps count as real evidence
  for an RNA annotation or read count.
when_to_use_negative:
- Input reads are not mapped to the human genome — those are routed to the Microbe
  module (BLAST against the UCSC nt database), not annotated with these RNA databases
- The goal is microbial (archaea/bacteria/fungi/viruses) profiling from unmapped reads
  rather than human small RNA annotation
- You need multi-mapper assignment or partial-overlap counting, since the default
  Rolp = 1 demands complete read containment within the annotated region
edam_operation: http://edamontology.org/operation_0362
tools:
- name: COMPSRA
  role: Annotation module that applies the overlap degree-of-read-support filter when
    counting human genome-mapped reads against small RNA databases
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default aligner producing the genome-mapped reads (BAM) that the overlap filter
    is applied to
- name: Java
  role: Runtime environment required to execute COMPSRA (JRE version 8 or up)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:2cf58c365acd5505ee57b3a04a66ab4c87be89b37014e240184142272b5cecf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Read overlap degree-of-support filtering

## Summary

Degree-of-read-support filtering used by COMPSRA's Annotation module to accept a small RNA annotation only when a mapped read overlaps the annotated gene region sufficiently, with default overlap ratio Rolp = 1 requiring the read to lie completely within the annotated feature.

## When to use

Use when annotating genome-mapped small RNA-seq reads against reference RNA databases and you must decide which read-gene overlaps count as real evidence for an RNA annotation or read count.

## When NOT to use

- Input reads are not mapped to the human genome — those are routed to the Microbe module (BLAST against the UCSC nt database), not annotated with these RNA databases
- The goal is microbial (archaea/bacteria/fungi/viruses) profiling from unmapped reads rather than human small RNA annotation
- You need multi-mapper assignment or partial-overlap counting, since the default Rolp = 1 demands complete read containment within the annotated region

## Inputs

- STAR alignment output BAM of hg38-aligned qualified reads (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam)
- Prebuilt small RNA reference databases (miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE release 27, circBase) for hg19/hg38

## Outputs

- Tab-delimited text file per RNA type (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) with read counts per annotated RNA molecule
- Per-RNA annotation records with the database items supporting each annotation

## How to apply

After STAR alignment of QC-qualified reads to hg38 (e.g. the sample01_17to50_FitRead_STAR_Aligned.out.bam), run the COMPSRA Annotation module with java -jar COMPSRA.jar -ann -ac 1,2,3,4,5,6 against the prebuilt miRNA (miRBase), piRNA (piRNABank/piRBase/piRNACluster), tRNA (gtRNAdb), snRNA/snoRNA (GENCODE release 27) and circRNA (circBase) databases, using LiftOver internally to harmonize hg18/hg19/hg38 coordinates. Apply the overlap-ratio filter with default Rolp = 1 (-aol 1.0) so each read must lie completely within the annotated gene region to support the annotation. Count each read once when it maps to multiple genomic loci of one RNA, but assign one count to each overlapping small RNA when distinct RNAs share sequence (e.g. let-7a-5p vs piR_008113). Rationale: complete-containment prevents partial, ambiguous overlaps from inflating counts, while per-RNA counting preserves quantification of co-annotated RNAs.

## Related tools

- **COMPSRA** (Annotation module that applies the overlap degree-of-read-support filter when counting human genome-mapped reads against small RNA databases) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default aligner producing the genome-mapped reads (BAM) that the overlap filter is applied to)
- **Java** (Runtime environment required to execute COMPSRA (JRE version 8 or up))

## Examples

```
java -jar COMPSRA.jar -ref hg38 -ann -ac 1,2,3,4,5,6 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam -out ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned
```

## Evaluation signals

- The tab-delimited annotation output exists and contains read counts for each of the six annotated RNA classes (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA)
- Each accepted read-gene pair satisfies the containment rule: the read lies completely within the annotated gene region (overlap ratio Rolp = 1 / -aol 1.0)
- A read mapping to multiple genomic loci of one RNA contributes exactly one count, not one per locus
- Distinct RNAs sharing sequence (e.g. let-7a-5p vs piR_008113) each receive one count from the same read rather than the read being assigned to a single RNA type
- Reference databases across hg18/hg19/hg38 were harmonized via the automatic LiftOver before overlap computation

## Limitations

- Default -aol 1.0 excludes reads that only partially overlap an annotated gene region, so genuinely overlapping but not fully contained reads are discarded
- Only human genome-mapped reads are annotated; unmapped qualified reads go to the Microbe module
- Currently supports hg19 and hg38 reference genomes only

## Evidence

- [methods] COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads"
- [readme] Annotation module databases and LiftOver: "COMPSRA currently uses several different small RNA databases for annotating human genome mapped reads and provides all the possible annotations"
- [other] Overlap filter and counting rule: [evidence span withheld — anchor status: not_found]
- [other] Multi-locus and shared-sequence counting: [evidence span withheld — anchor status: not_found]
