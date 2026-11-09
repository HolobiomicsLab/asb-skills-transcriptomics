---
name: tool-unique-mirna-validation-in-independent-cohort
description: Use when two small RNA-seq pipelines have been run on the same samples
  and you must determine which tool's tool-unique miRNA calls are better supported
  by an external dataset.
when_to_use_negative:
- The two tools have not both been run on the identical FASTQ inputs with matched
  parameters, so the unique sets are not comparable
- No independent external dataset with per-miRNA read counts is available to serve
  as the validation cohort
- The goal is differential expression between case/control rather than comparing tool-specific
  detection reliability
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: COMPSRA
  role: Stand-alone small RNA-seq pipeline whose miRNA count output provides one tool's
    detected miRNA set; run with parameters matched to exceRpt (-aln -mt star, -ann
    -ac 1,2,3,4,5,6)
  repo: https://github.com/cougarlj/COMPSRA
- name: exceRpt
  role: Web-based comparison pipeline run on the same 12 serum FASTQ files to produce
    the second tool's miRNA identification results
- name: STAR
  role: Default aligner inside COMPSRA for local-end alignment of QC-passed reads
    to hg19/hg38 with at most one mismatch
- name: miRBase
  role: Reference miRNA annotation database (miRBase20 prebuilt in COMPSRA) used to
    define the miRNA identifier space for set comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:11daa748c5a9f308e51940d273487c2fdf3c99fc0ef974df8d073586f23d4623
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  - exceRpt
  - miRBase
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# tool-unique-mirna-validation-in-independent-cohort

## Summary

Validates miRNAs uniquely detected by one small RNA-seq pipeline (e.g., COMPSRA vs exceRpt) by checking their detectability in an independent serum small RNA-seq dataset (SRP120169) under a read-count threshold of 5, to judge which tool's unique calls are more reliable.

## When to use

Use when two small RNA-seq pipelines have been run on the same samples and you must determine which tool's tool-unique miRNA calls are better supported by an external dataset.

## When NOT to use

- The two tools have not both been run on the identical FASTQ inputs with matched parameters, so the unique sets are not comparable
- No independent external dataset with per-miRNA read counts is available to serve as the validation cohort
- The goal is differential expression between case/control rather than comparing tool-specific detection reliability

## Inputs

- miRBase-annotated miRNA count output from COMPSRA (tab-delimited counts per sample for 12 serum samples)
- miRBase-annotated miRNA identification results from the web-based exceRpt pipeline on the same 12 FASTQ files
- Independent serum small RNA-seq dataset SRP120169 (read counts per miRNA)

## Outputs

- Shared and per-tool-unique miRNA set counts (e.g., 358 shared; 17 COMPSRA-unique; 24 exceRpt-unique)
- Per-tool validation rates in SRP120169 at read-count threshold 5 (13/17 = 76.47% vs 11/24 = 45.83%)
- Tabular summary written to mirna_overlap_stats.csv

## How to apply

Extract the miRBase-annotated miRNA count matrices from each tool (COMPSRA runs on the 12 serum FASTQ files with STAR local-end alignment, at most one mismatch, 4 random bases trimmed at both 5' and 3' ends, and -ann_overlap 1) and compute each tool's unique set against the shared set (358 shared, ~90% concordance). Then query an independent serum small RNA-seq dataset (SRP120169) and count how many of each tool's unique miRNAs are detected above a read-count threshold of 5. Compare validation rates: 13/17 (76.47%) for COMPSRA-unique versus 11/24 (45.83%) for exceRpt-unique miRNAs indicates the tool with the higher confirmation rate produces more reliable unique calls. Compile counts, overlap, and validation rates into mirna_overlap_stats.csv.

## Related tools

- **COMPSRA** (Stand-alone small RNA-seq pipeline whose miRNA count output provides one tool's detected miRNA set; run with parameters matched to exceRpt (-aln -mt star, -ann -ac 1,2,3,4,5,6)) — https://github.com/cougarlj/COMPSRA
- **exceRpt** (Web-based comparison pipeline run on the same 12 serum FASTQ files to produce the second tool's miRNA identification results)
- **STAR** (Default aligner inside COMPSRA for local-end alignment of QC-passed reads to hg19/hg38 with at most one mismatch)
- **miRBase** (Reference miRNA annotation database (miRBase20 prebuilt in COMPSRA) used to define the miRNA identifier space for set comparison)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./serum12.list -out ./serum_out/
```

## Evaluation signals

- Two-tool shared miRNA count equals 358 (~90% concordance) within rounding tolerance
- SRP120169 validation rate for COMPSRA-unique miRNAs is 13/17 = 76.47%
- SRP120169 validation rate for exceRpt-unique miRNAs is 11/24 = 45.83%
- Per-miRNA detection in SRP120169 is called confirmed only when read count >= 5
- mirna_overlap_stats.csv contains miRNA counts, overlap counts, and both SRP120169 validation rates

## Limitations

- Validation depends on detection thresholds; using a read-count cutoff other than 5 changes the confirmation rates
- exceRpt is web-based and requires uploading original sequencing files, which is unworkable for big data
- COMPSRA runs ~10 minutes per sample versus ~20 minutes for exceRpt, and the software version used in the comparison is not stated

## Evidence

- [results] Validating against SRP120169 with a read count threshold of 5, 13 of 17 COMPSRA-unique miRNAs (76.47%) were confirmed versus 11 of 24 exceRpt-unique miRNAs (45.83%): [evidence span withheld — anchor status: not_found]
- [results] overlap of 358 shared miRNAs: [evidence span withheld — anchor status: not_found]
- [results] matched parameters on 30 GB RAM server: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [readme] COMPSRA README pipeline invocation: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./example/sample.list -out ./example_out/"
