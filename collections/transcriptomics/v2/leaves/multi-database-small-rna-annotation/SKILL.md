---
name: multi-database-small-rna-annotation
description: Use when FASTQ small RNA-seq reads have been mapped to hg19/hg38 with
  STAR and you need to classify and count reads into multiple small RNA classes (miRNA,
  piRNA, snRNA, snoRNA, tRNA, circRNA) in one pass rather than running separate single-class
  tools.
when_to_use_negative:
- Reads have not yet been adapter-trimmed and aligned to the human genome — run the
  QC and Alignment modules first, since annotation requires a STAR-mapped BAM
- The goal is microbial profiling of unmapped reads — those belong to the Microbe
  module (BLAST against the UCSC nt database), not the annotation databases
- The target organism is not human or rat (hg19/hg38/rno5/rno6) — prebuilt databases
  are only provided for these genome versions
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Annotation module (-ann -ac 1,2,3,4,5,6) assigns aligned reads to miRNA, piRNA,
    snRNA, snoRNA, tRNA and circRNA classes using prebuilt databases and outputs per-type
    count files
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default aligner producing the genome-mapped BAM that is the input to annotation;
    small-RNA parameters include --outFilterMismatchNmax 1 and --outFilterMatchNmin
    16
- name: Java
  role: Runtime environment (JRE 8+) required to execute COMPSRA.jar and its ToolKit
    database downloader
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:76e22f4a605884b6da29b71648aabfbe2a4e019e6e6a8957e99d9ff1509ddf6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-database-small-rna-annotation

## Summary

Annotate genome-mapped small RNA-seq reads against multiple prebuilt reference databases (miRBase for miRNA, piRNABank/piRBase/piRNACluster for piRNA, gtRNAdb for tRNA, GENCODE release 27 for snRNA/snoRNA, circBase for circRNA) to quantify every small RNA class from a single alignment, using COMPSRA's Annotation module.

## When to use

Use when FASTQ small RNA-seq reads have been mapped to hg19/hg38 with STAR and you need to classify and count reads into multiple small RNA classes (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA) in one pass rather than running separate single-class tools.

## When NOT to use

- Reads have not yet been adapter-trimmed and aligned to the human genome — run the QC and Alignment modules first, since annotation requires a STAR-mapped BAM
- The goal is microbial profiling of unmapped reads — those belong to the Microbe module (BLAST against the UCSC nt database), not the annotation databases
- The target organism is not human or rat (hg19/hg38/rno5/rno6) — prebuilt databases are only provided for these genome versions

## Inputs

- STAR-aligned BAM file (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam) from small RNA-seq reads
- Prebuilt annotation databases: miRNA_hg38, piRNA_hg38, tRNA_hg38, snoRNA_hg38, snRNA_hg38, circRNA_hg38
- Reference genome version flag (-ref hg19/hg38)

## Outputs

- Tab-delimited per-sample read count files, one per RNA type (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA)
- Merged RNA-by-sample count matrix (via Function module -fm) with optional CpM normalization

## How to apply

Run the COMPSRA Annotation module (-ann) on the STAR-aligned BAM with -ac 1,2,3,4,5,6 to enable all six RNA classes, using the prebuilt databases downloaded via the ToolKit (miRNA_hg38, piRNA_hg38, tRNA_hg38, snoRNA_hg38, snRNA_hg38, circRNA_hg38). Keep the default full-read overlap requirement (-ann_overlap 1) so a read is assigned only when the aligned read overlaps the annotation feature completely. Because the source databases were built against different human genome versions, rely on COMPSRA's automatic LiftOver to harmonize coordinates to the chosen reference (e.g. hg38). Each RNA molecule's read count is output as a tab-delimited text file per RNA type, and multiple samples can be merged into a count matrix with the Function module (-fun -fm -fms 1-12 -fdclass 1,2,3,4,5,6) with Count-per-Million normalization as the default.

## Related tools

- **COMPSRA** (Annotation module (-ann -ac 1,2,3,4,5,6) assigns aligned reads to miRNA, piRNA, snRNA, snoRNA, tRNA and circRNA classes using prebuilt databases and outputs per-type count files) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default aligner producing the genome-mapped BAM that is the input to annotation; small-RNA parameters include --outFilterMismatchNmax 1 and --outFilterMatchNmin 16)
- **Java** (Runtime environment (JRE 8+) required to execute COMPSRA.jar and its ToolKit database downloader)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -ann -ac 1,2,3,4,5,6 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam -out ./example_out/sample01/
```

## Evaluation signals

- Reproducing the 12 healthy human serum samples should yield approximately 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs detected (read count threshold of 5)
- Each output count file should be tab-delimited with one row per RNA molecule of the corresponding class and contain only mapped-read counts
- All six classes should be annotated in a single run when -ac 1,2,3,4,5,6 is used; missing classes indicate a database failed to download or load
- Merged matrices from the Function module should have RNA molecules as rows and samples as columns, matching the number of input FASTQ files
- STAR index building on first use requires at least 30 GB RAM; annotation coordinates must correspond to the chosen -ref genome version after LiftOver

## Limitations

- Databases must be pre-downloaded with the ToolKit (-tk -dr -ck) before annotation; first-time STAR index building costs about 3 hours and failed builds require chmod on the STAR executable
- Source databases use differing human genome versions and depend on automatic UCSC LiftOver, which can introduce coordinate-mapping discrepancies
- COMPSRA's annotation is limited to the RNA classes covered by its prebuilt databases (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA) plus fragmental microbial RNA via the separate Microbe module

## Evidence

- [readme] COMPSRA currently uses several different (and expandable) small RNA databases for annotating human genome mapped reads and provides all the possible annotations: miRBase for miRNA; piRNABank, piRBase and piRNACluster for piRNA; gtRNAdb for tRNA; GENCODE release 27 for snRNA and snoRNA; circBase for circular RNA.: "COMPSRA currently uses several different small RNA databases for annotating human genome mapped reads and provides all the possible annotations: miRBase for miRNA; piRNABank, piRBase and piRNACluster"
- [readme] LiftOver harmonization of database versions.: "To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group."
- [results] Detected molecule counts across the 12 serum samples.: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs"
- [readme] Annotation command from the README examples.: "java -jar COMPSRA.jar -ref hg38 -ann -ac 1,2,3,4,5,6 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam"
- [readme] Count file aggregation described in the Function module.: "The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type."
