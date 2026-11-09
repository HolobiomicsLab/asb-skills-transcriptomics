---
name: small-rna-seq-quality-control
description: Use when raw small RNA-seq FASTQ files (e.g. 50 nt reads from serum small
  RNA kits like NEXTflex) need adapter removal, ligation-base trimming, and low-quality
  read filtering before genome alignment or small RNA annotation.
when_to_use_negative:
- Input is not raw small RNA-seq FASTQ (e.g. already aligned BAM/SAM files should
  go straight to the Annotation module)
- The library kit does not leave randomized bases at ligation junctions or uses a
  different adapter than TGGAATTCTCGGGTGCCAAGG
- Reads are longer than 200 nt and not small RNAs — COMPSRA targets small RNAs (<200
  nt)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: COMPSRA
  role: Provides the QC module invoked with -qc that performs adapter trimming, randomized-base
    removal, quality filtering and read-length grouping
  repo: https://github.com/cougarlj/COMPSRA
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:089151bb98df3704d83eefc6ae079b76cd331699baa732b4b5a84117c0e2e4f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - exceRpt
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# small-rna-seq-quality-control

## Summary

Quality-control routine for small RNA-seq FASTQ data as implemented in the COMPSRA QC module: trims the 3' adapter and randomized ligation bases, applies Phred-score-based read/head/tail filtering, and groups qualified reads by length interval before downstream alignment and annotation.

## When to use

Use when raw small RNA-seq FASTQ files (e.g. 50 nt reads from serum small RNA kits like NEXTflex) need adapter removal, ligation-base trimming, and low-quality read filtering before genome alignment or small RNA annotation.

## When NOT to use

- Input is not raw small RNA-seq FASTQ (e.g. already aligned BAM/SAM files should go straight to the Annotation module)
- The library kit does not leave randomized bases at ligation junctions or uses a different adapter than TGGAATTCTCGGGTGCCAAGG
- Reads are longer than 200 nt and not small RNAs — COMPSRA targets small RNAs (<200 nt)

## Inputs

- Small RNA-seq FASTQ files (e.g. 50 nt single-end reads from 12 healthy human serum samples)
- 3' adapter sequence (TGGAATTCTCGGGTGCCAAGG)
- QC parameters: -rb 4, -rh 20 -rt 20 -rr 20, -rlh 8,17

## Outputs

- Gzipped FASTQ of qualified reads filtered to the specified length interval (e.g. *_17to50_FitRead.fastq.gz)
- Per-sample read statistics after adapter/quality filtering

## How to apply

Run the COMPSRA QC module on each FASTQ, removing the adapter at the 3' end (-ra TGGAATTCTCGGGTGCCAAGG) along with 4 random bases at both 5' and 3' ligation junctions (-rb 4), since some small RNA-seq kits add randomized bases at ligation junctions. Apply quality thresholds by removing low-quality head bases (-rh 20), tail bases (-rt 20), and whole reads whose average Phred score falls below 20 (-rr 20). Use read-length grouping (-rlh 8,17) to restrict downstream analysis to reads in the relevant small-RNA length intervals. Expect read length to drop from 50 nt raw to 0-42 nt after trimming; the QC output (e.g. sample01_17to50_FitRead.fastq.gz) feeds directly into the STAR-based Alignment module. The 20-score thresholds and -rb 4 setting are the defaults used in the article's 12-sample healthy human serum benchmark.

## Related tools

- **COMPSRA** (Provides the QC module invoked with -qc that performs adapter trimming, randomized-base removal, quality filtering and read-length grouping) — https://github.com/cougarlj/COMPSRA

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- Per-sample qualified-read output files named with the length-interval suffix (e.g. *_17to50_FitRead.fastq.gz) are produced for each of the 12 input FASTQ files
- Read lengths in the QC output fall within the expected 0-42 nt range after adapter and 4-base trimming of 50 nt raw reads
- All reads retaining adapter sequence TGGAATTCTCGGGTGCCAAGG at the 3' end are removed
- QC output feeds the Alignment module unchanged and yields hg38-aligned BAMs (via STAR with --outFilterMismatchNmax 1, --outFilterMatchNmin 16) consistent with the article's benchmark
- Pipeline with the same parameters on the 12 serum samples supports detection of 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs downstream

## Limitations

- The default thresholds (-rh 20 -rt 20 -rr 20) and adapter sequence are kit-specific (e.g. NEXTflex Small RNA-Seq kit); other kits require changed parameters
- Removing 4 random bases at both ends plus adapter trimming can reduce 50 nt raw reads to as little as 0 nt, so reads must be long enough to survive trimming for downstream length-grouped analysis
- COMPSRA v1.0 software version used for the article benchmark is not stated in the results section, so exact version pinning is ambiguous

## Evidence

- [methods] First, the adapter portions of a read are trimmed along with any randomized bases at ligation junctions that are produced by some small RNA-seq kits: "First, the adapter portions of a read are trimmed along with any randomized bases at ligation junctions that are produced by some small RNA-seq kits"
- [readme] Poor quality reads are removed according to quality control parameters set in the command line (-rh 20 –rt 20 –rr 20): "Poor quality reads are removed according to quality control parameters set in the command line (-rh 20 –rt 20 –rr 20). Users can specify qualified reads of specific length intervals"
- [results] The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
- [readme] java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -in ./example/sample01.fastq -out ./example_out/: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -in ./example/sample01.fastq -out ./example_out/"
