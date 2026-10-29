---
name: mirna-annotation-against-mirbase
description: Use when small RNA-seq reads (e.g., 50 nt serum FASTQ with 3' adapter
  TGGAATTCTCGGGTGCCAAGG) have been STAR-aligned to hg19/hg38 and must be quantified
  against miRBase miRNAs to build a miRNA-by-sample count matrix.
when_to_use_negative:
- Reads have not yet been adapter-trimmed and quality-filtered by the COMPSRA QC module
  (e.g., raw FASTQ still containing the TGGAATTCTCGGGTGCCAAGG adapter and random ligation
  bases)
- The goal is non-miRNA small RNA annotation (piRNA, tRNA, snRNA, snoRNA, circRNA)
  or microbial profiling — use the corresponding databases or the COMPSRA Microbe/BLAST
  module instead
- The data are not human (hg19/hg38); COMPSRA's prebuilt miRBase databases also cover
  rat (rno5/rno6), but other organisms require different resources
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Runs QC, Alignment and Annotation modules; annotates STAR-mapped reads to
    prebuilt miRBase miRNA databases with the -ann and -ac options and aggregates
    counts via the Function module
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default aligner used by COMPSRA's Alignment module to map QC-qualified reads
    to hg38 before miRBase annotation
- name: miRBase
  role: Reference miRNA database (prebuilt for hg38/hg19 via COMPSRA ToolKit, e.g.,
    miRNA_hg38) used as annotation background for miRNA identification and quantification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:004ddf6bdfc1ac7806f89f91a8e4054c61b7e2f6ffd48d78c92c0835b3ea81c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - STAR
  - miRBase
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# mirna-annotation-against-mirbase

## Summary

Annotate STAR-mapped small RNA-seq reads to miRBase miRNAs using the COMPSRA Annotation module, producing per-sample miRNA read counts suitable for downstream aggregation and benchmarking against reference miRNA sets.

## When to use

Use when small RNA-seq reads (e.g., 50 nt serum FASTQ with 3' adapter TGGAATTCTCGGGTGCCAAGG) have been STAR-aligned to hg19/hg38 and must be quantified against miRBase miRNAs to build a miRNA-by-sample count matrix.

## When NOT to use

- Reads have not yet been adapter-trimmed and quality-filtered by the COMPSRA QC module (e.g., raw FASTQ still containing the TGGAATTCTCGGGTGCCAAGG adapter and random ligation bases)
- The goal is non-miRNA small RNA annotation (piRNA, tRNA, snRNA, snoRNA, circRNA) or microbial profiling — use the corresponding databases or the COMPSRA Microbe/BLAST module instead
- The data are not human (hg19/hg38); COMPSRA's prebuilt miRBase databases also cover rat (rno5/rno6), but other organisms require different resources

## Inputs

- STAR Aligned.out.bam from hg19/hg38-aligned, QC-filtered small RNA-seq reads (e.g., 12 serum FASTQ files from SRP120169)
- Prebuilt miRBase miRNA annotation database for hg38 (miRNA_hg38 via COMPSRA ToolKit)
- STAR hg38 genome index (star_hg38)

## Outputs

- Per-sample miRNA read count files (tab-delimited text, one per RNA type)
- Aggregated miRNA-by-sample count matrix (COMPSRA Function module merge output)
- Downstream derived sensitivity/specificity table (e.g., sensitivity_specificity.tsv: 85%/87% overall, 93%/98% for top 30% miRNAs)

## How to apply

Run the COMPSRA Annotation module (-ann) on the STAR Aligned.out.bam produced by the Alignment module (-aln -mt star) after QC (-qc with adapter -ra TGGAATTCTCGGGTGCCAAGG, -rb 4 random-base trimming, -rh 20 -rt 20 -rr 20 quality filters, -rlh 8,17 length grouping). Use the prebuilt miRBase miRNA database for the chosen genome (download via java -jar COMPSRA.jar -tk -dr -ck miRNA_hg38), and set the annotation class code (-ac 1) with the default full-overlap criterion (Rolp = 1) so a read is only assigned when it falls entirely within the miRNA locus. Collect the per-sample miRNA count outputs and aggregate them with the Function module merge mode (-fun -fm -fms 1-12 -fdclass 1,2,3,4,5,6 -fdann) into a tab-delimited miRNA-by-sample matrix; this matrix is then the basis for downstream validation, e.g., computing sensitivity/specificity of detected miRNAs against an external reference set such as SRP120169 under miRBase v21 background. The rationale is that miRBase annotation on pre-built, LiftOver-harmonized databases (via UCSC LiftOver across genome versions) is faster and more stable than re-annotating de novo.

## Related tools

- **COMPSRA** (Runs QC, Alignment and Annotation modules; annotates STAR-mapped reads to prebuilt miRBase miRNA databases with the -ann and -ac options and aggregates counts via the Function module) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default aligner used by COMPSRA's Alignment module to map QC-qualified reads to hg38 before miRBase annotation)
- **miRBase** (Reference miRNA database (prebuilt for hg38/hg19 via COMPSRA ToolKit, e.g., miRNA_hg38) used as annotation background for miRNA identification and quantification)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -ann -ac 1 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam -out ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned
```

## Evaluation signals

- Read assignment uses the default full-overlap criterion (Rolp = 1): a read is counted for a miRNA only if fully contained within its annotated locus
- On the 12 healthy human serum samples with a read count threshold of 5, COMPSRA should detect approximately 375 miRNAs
- Benchmarked against the SRP120169 reference miRNA set (miRBase v21 background), the detected list should yield sensitivity 85% / specificity 87% overall and 93% / 98% restricted to the top 30% most abundant miRNAs
- The aggregated Function-module output must be a miRNA-by-sample count matrix (rows = RNA molecules, columns = samples) in tab-delimited text
- Output files appear per sample under the -out directory following the sample01_17to50_FitRead_STAR_Aligned naming pattern

## Limitations

- Annotation databases are pre-built and genome-version specific (hg19/hg38, rno5/rno6); cross-version consistency relies on UCSC LiftOver, so mismatches between genome versions of databases can propagate annotation errors
- The default full-overlap criterion (Rolp = 1) is stricter than partial-overlap annotation; reads partially overlapping a miRNA are unannotated, potentially reducing sensitivity
- miRBase is only one of several reference databases COMPSRA can annotate against; using -ac codes other than the miRNA class (e.g., -ac 1,2,3,4,5,6) mixes in piRNA, tRNA, snRNA, snoRNA and circRNA annotations

## Evidence

- [readme] COMPSRA currently uses several different (and expandable) small RNA databases for annotating human genome mapped reads and provides all the possible annotations: miRBase for miRNA: "COMPSRA currently uses several different small RNA databases for annotating human genome mapped reads and provides all the possible annotations: miRBase for miRNA"
- [methods] Qualified reads from the QC module output are first mapped to the target genome (e.g., hg19/hg38), and then aligned reads are quantified and annotated in the Annotation Module: "Qualified reads from the QC module output are first mapped to the target genome (e.g., hg19/hg38), and then aligned reads are quantified and annotated in the Annotation Module"
- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs in the 12 serum samples (read count threshold of 5): "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs"
- [readme] The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type: "The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type"
- [results] COMPSRA achieved a sensitivity of 85% and specificity of 87% against the SRP120169 benchmark overall: [evidence span withheld — anchor status: not_found]
