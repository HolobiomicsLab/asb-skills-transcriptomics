---
name: genome-coordinate-liftover-harmonization
description: Use when annotation reference databases or BED/GFF resources are provided
  in different genome builds (e.g. hg18, hg19, hg38) than the genome used for read
  alignment, and coordinates must be converted before overlap-based annotation.
when_to_use_negative:
- All annotation resources are already in the same build as the aligned reads — no
  LiftOver step is needed
- The analysis target is non-human reads; COMPSRA's pre-built LiftOver-harmonized
  annotation databases cover human hg19/hg38 small RNAs, while unmapped reads go to
  the Microbe module via BLAST against the UCSC nt database
- Alignment or annotation must be redone from scratch against a custom genome build
  not among the pre-built hg19/hg38 resources
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: COMPSRA
  role: Provides the Annotation module (-ann) with pre-built, LiftOver-harmonized
    small RNA databases named per RNA type and genome version (e.g. miRNA_hg38), downloadable
    via its ToolKit
  repo: https://github.com/cougarlj/COMPSRA
- name: LiftOver
  role: UCSC Genome Browser Group tool used automatically inside COMPSRA to conform
    differing reference human genome versions across the annotation databases
- name: Java
  role: Runtime (JRE 8 or up) in which COMPSRA, and thus its database download and
    annotation commands, execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:969d2a11c5b6467ff58679fd09acca67247db66218482ee87a8c37350958abff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - LiftOver
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Genome coordinate LiftOver harmonization

## Summary

Reconciles heterogeneous reference genome builds (hg18/hg19/hg38) across small RNA annotation databases so that reads mapped to one genome version can be annotated against all pre-built resources, as implemented internally by COMPSRA using UCSC LiftOver.

## When to use

Use when annotation reference databases or BED/GFF resources are provided in different genome builds (e.g. hg18, hg19, hg38) than the genome used for read alignment, and coordinates must be converted before overlap-based annotation.

## When NOT to use

- All annotation resources are already in the same build as the aligned reads — no LiftOver step is needed
- The analysis target is non-human reads; COMPSRA's pre-built LiftOver-harmonized annotation databases cover human hg19/hg38 small RNAs, while unmapped reads go to the Microbe module via BLAST against the UCSC nt database
- Alignment or annotation must be redone from scratch against a custom genome build not among the pre-built hg19/hg38 resources

## Inputs

- STAR-aligned BAM of human genome-mapped reads (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam) in a known build (hg19 or hg38)
- Pre-built small RNA annotation databases per build: miRNA_hg38/hg19, piRNA_hg38/hg19, tRNA_hg38/hg19, snoRNA_hg38/hg19, snRNA_hg38/hg19, circRNA_hg38/hg19

## Outputs

- Coordinate-harmonized annotation databases consistent with the alignment reference genome
- Per-RNA-type tab-delimited count files (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) with database items supporting each annotation

## How to apply

Identify the genome build of the aligned reads (e.g. STAR BAM from hg38-qualified reads such as sample01_17to50_FitRead_STAR_Aligned.out.bam) and the build of each annotation resource (miRBase miRNA, piRNABank/piRBase/piRNACluster piRNA, gtRNAdb tRNA, GENCODE release 27 snRNA/snoRNA, circBase circRNA). Rather than manually converting each source, use a pre-built, already-harmonized database set where LiftOver (UCSC Genome Browser Group) has been applied automatically, as COMPSRA does — resources are named by RNA type plus genome version (e.g. miRNA_hg38, piRNA_hg19) and downloaded via the ToolKit (-tk -dr -ck). Annotate reads against the harmonized resources with a strict overlap criterion (default Rolp = 1, i.e. the read must lie completely within the annotated gene region, -aol 1.0), counting each read once when it hits multiple loci of one RNA and once per RNA when distinct RNAs share sequence. Harmonization should be transparent to downstream output: per-RNA-type tab-delimited count files regardless of which source build was lifted.

## Related tools

- **COMPSRA** (Provides the Annotation module (-ann) with pre-built, LiftOver-harmonized small RNA databases named per RNA type and genome version (e.g. miRNA_hg38), downloadable via its ToolKit) — https://github.com/cougarlj/COMPSRA
- **LiftOver** (UCSC Genome Browser Group tool used automatically inside COMPSRA to conform differing reference human genome versions across the annotation databases)
- **Java** (Runtime (JRE 8 or up) in which COMPSRA, and thus its database download and annotation commands, execute)

## Examples

```
java -jar COMPSRA.jar -tk -dr -ck miRNA_hg38,piRNA_hg38,tRNA_hg38,snoRNA_hg38,snRNA_hg38,circRNA_hg38
```

## Evaluation signals

- Pre-built database resources match the alignment reference genome (e.g. miRNA_hg38 used with -ref hg38) rather than mixing builds
- Tab-delimited annotation output exists and contains read counts for all six annotated RNA classes (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) with the database items supporting each annotation
- With default overlap ratio Rolp = 1 (-aol 1.0), every counted read lies completely within the annotated gene region
- A read mapping to multiple genomic loci of one RNA is counted once per RNA, not once per locus; distinct RNAs sharing sequence (e.g. let-7a-5p vs piR_008113) each receive one count
- Counts are invariant to which source database build was lifted over, since harmonization occurs upstream of annotation

## Limitations

- Pre-built harmonized databases exist only for hg19 and hg38 human genome versions; other builds require external handling
- Rigorous default overlap (Rolp = 1, -aol 1.0) excludes reads that only partially overlap an annotated region, which can drop boundary-spanning reads
- Distinct RNAs with identical sequences (e.g. let-7a-5p vs piR_008113) cannot be uniquely assigned — one count is given to each overlapping RNA

## Evidence

- [methods] To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group.: "To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group."
- [methods] COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA"
- [readme] All the databases used are already pre-built, enabling speedy annotation.: "All the databases used are already pre-built, enabling speedy annotation."
- [readme] COMPSRA supports the annotation of miRNA, piRNA, tRNA, snoRNA, snRNA and circRNA with both hg38 and hg19 version.: "COMPSRA supports the annotation of miRNA, piRNA, tRNA, snoRNA, snRNA and circRNA with both hg38 and hg19 version."
