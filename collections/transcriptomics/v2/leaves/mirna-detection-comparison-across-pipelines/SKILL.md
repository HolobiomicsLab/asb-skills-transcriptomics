---
name: mirna-detection-comparison-across-pipelines
description: Use when two small RNA-seq miRNA identification pipelines (e.g. COMPSRA
  and exceRpt) have been run on identical FASTQ inputs and you need to quantify detection
  overlap and independently validate tool-unique miRNA calls.
when_to_use_negative:
- The analysis goal is profiling non-miRNA small RNA classes (piRNA, tRNA, snRNA,
  snoRNA, circRNA) or microbial content rather than comparing miRNA detection between
  pipelines
- The two tools being compared were run on different samples or with mismatched parameters,
  making the overlap analysis invalid
- No independent external dataset is available to validate tool-unique miRNA calls
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COMPSRA
  role: Stand-alone small RNA-seq pipeline (QC, Alignment, Annotation) run locally
    with exceRpt-matched parameters to produce miRNA counts
  repo: https://github.com/cougarlj/COMPSRA
- name: exceRpt
  role: Web-based small RNA-seq pipeline whose miRNA identification results are compared
    against COMPSRA on identical FASTQ inputs
- name: STAR
  role: Default aligner inside COMPSRA, used with local-end alignment allowing at
    most one mismatch
- name: miRBase
  role: Reference annotation database used by both tools to define the miRNA sets
    being compared
- name: Java
  role: Runtime environment (JRE 8+) required to execute COMPSRA.jar
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:b7d4db8130d7148d1427a8a61fb61a9f7834fccca33da58e67f907669750b174
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mirna-detection-comparison-across-pipelines

## Summary

Reproduces a head-to-head comparison of miRNA detection between the COMPSRA stand-alone pipeline and the web-based exceRpt pipeline on the same 12 serum small RNA-seq FASTQ samples, quantifying shared vs. tool-unique miRNA calls and validating unique calls against an external dataset (SRP120169).

## When to use

Use when two small RNA-seq miRNA identification pipelines (e.g. COMPSRA and exceRpt) have been run on identical FASTQ inputs and you need to quantify detection overlap and independently validate tool-unique miRNA calls.

## When NOT to use

- The analysis goal is profiling non-miRNA small RNA classes (piRNA, tRNA, snRNA, snoRNA, circRNA) or microbial content rather than comparing miRNA detection between pipelines
- The two tools being compared were run on different samples or with mismatched parameters, making the overlap analysis invalid
- No independent external dataset is available to validate tool-unique miRNA calls

## Inputs

- 12 serum small RNA-seq FASTQ files (50 nt reads) from healthy human controls
- exceRpt web pipeline miRNA identification results for the same 12 files
- COMPSRA miRNA count output (miRBase-annotated)
- Independent validation dataset SRP120169 (serum small RNA-seq)

## Outputs

- Shared miRNA count and per-tool unique miRNA lists (threshold: read count ≥ 5)
- SRP120169 validation rates for COMPSRA-unique (13/17 = 76.47%) and exceRpt-unique (11/24 = 45.83%) miRNAs
- mirna_overlap_stats.csv tabulating miRNA counts, overlap counts, and validation rates

## How to apply

Run COMPSRA (QC, Alignment, Annotation modules; STAR local-end alignment with at most one mismatch, adapter trimming removing 4 random bases at both 5' and 3' ends, default full-read annotation overlap -ann_overlap 1) on the 12 serum FASTQ files with parameters matched to exceRpt on a server with 30 GB RAM, then run the web-based exceRpt pipeline on the identical files. Extract miRBase-annotated miRNA lists from both tools' miRNA count outputs (read count threshold of 5) and compute the shared set (~358 miRNAs, ~90% concordance) and per-tool unique sets. Validate each tool-unique miRNA list by checking detectability in the independent SRP120169 serum small RNA-seq dataset, expecting 13/17 (76.47%) for COMPSRA-unique versus 11/24 (45.83%) for exceRpt-unique. Compile counts, overlap, and validation rates into mirna_overlap_stats.csv. Expect COMPSRA to take ~10 minutes per sample versus ~20 minutes per sample for exceRpt.

## Related tools

- **COMPSRA** (Stand-alone small RNA-seq pipeline (QC, Alignment, Annotation) run locally with exceRpt-matched parameters to produce miRNA counts) — https://github.com/cougarlj/COMPSRA
- **exceRpt** (Web-based small RNA-seq pipeline whose miRNA identification results are compared against COMPSRA on identical FASTQ inputs)
- **STAR** (Default aligner inside COMPSRA, used with local-end alignment allowing at most one mismatch)
- **miRBase** (Reference annotation database used by both tools to define the miRNA sets being compared)
- **Java** (Runtime environment (JRE 8+) required to execute COMPSRA.jar)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- Shared miRNA count between COMPSRA and exceRpt equals 358 (~90% concordance) within rounding tolerance
- SRP120169 validation rate is 13/17 (76.47%) for COMPSRA-unique miRNAs and 11/24 (45.83%) for exceRpt-unique miRNAs (17 and 24 unique miRNAs, respectively, at read count threshold 5)
- Both pipelines were run on the identical 12 FASTQ files with matched parameters (30 GB RAM server, same trimming and alignment settings)
- mirna_overlap_stats.csv contains miRNA counts, overlap counts, and SRP120169 validation rates in a consistent tabular schema
- Per-sample runtimes fall near ~10 minutes for COMPSRA and ~20 minutes for exceRpt

## Limitations

- COMPSRA software version used in the comparison is not stated in the article
- COMPSRA's comparison was specific to 50 nt serum small RNA-seq reads; after adapter and 4-base random-region trimming, read lengths ranged from 0 to 42 nt
- Web-based tools like exceRpt require uploading original sequencing files, which is unworkable for big data
- First-time STAR index building inside COMPSRA takes about 3 hours

## Evidence

- [methods] matched_parameters: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [results] runtime_difference: "COMPSRA takes ~10 minutes per sample, but exceRpt will cost ~20 minutes for each sample"
- [results] read_lengths: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
- [readme] pipeline_command: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -in ./example/sample01.fastq"
- [intro] big_data_upload: "All the tools need users to upload the original sequencing files, which is un-workable for big data"
