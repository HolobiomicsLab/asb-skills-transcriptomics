---
name: standalone-platform-execution-with-matched-parameters
description: Use when comparing a local stand-alone small RNA-seq pipeline against
  a web-based tool on identical FASTQ files and the comparison requires parameters
  matched to the reference tool for a fair tool-vs-tool evaluation.
when_to_use_negative:
- Data is too large to process locally but the goal is not tool comparison — the article
  notes web-based tools require uploading original sequencing files, which is unworkable
  for big data, so a stand-alone run is preferred for big data anyway
- The goal is microbial (archaea/bacteria/fungi/virus) profiling — that requires the
  optional Microbe module and BLAST databases (~3.5 hours average runtime), beyond
  the QC/Alignment/Annotation scope of this skill
- Only a differential expression analysis is needed on existing count matrices — use
  the Function module directly instead of re-running QC/Alignment/Annotation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Stand-alone modular pipeline (QC, Alignment, Annotation) run with exceRpt-matched
    parameters to identify and quantify miRNAs from serum small RNA-seq FASTQ files
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default RNA-seq aligner used by COMPSRA for local-end alignment of trimmed
    reads to hg38 with at most one mismatch
- name: miRBase
  role: miRNA reference database (miRBase20) used by the Annotation module to produce
    the miRNA count output
- name: Java
  role: Runtime environment (JRE 8+) required to execute the COMPSRA.jar stand-alone
    package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:b87f7297c8566b49c192919924c1cb01336a26c0a9703226066b56a66ca86a97
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# standalone-platform-execution-with-matched-parameters

## Summary

Execute a stand-alone small RNA-seq platform (COMPSRA v1.0) on local FASTQ inputs with alignment, trimming, and annotation parameters explicitly matched to a comparison pipeline (exceRpt), enabling a fair head-to-head comparison of miRNA detection on 12 serum samples.

## When to use

Use when comparing a local stand-alone small RNA-seq pipeline against a web-based tool on identical FASTQ files and the comparison requires parameters matched to the reference tool for a fair tool-vs-tool evaluation.

## When NOT to use

- Data is too large to process locally but the goal is not tool comparison — the article notes web-based tools require uploading original sequencing files, which is unworkable for big data, so a stand-alone run is preferred for big data anyway
- The goal is microbial (archaea/bacteria/fungi/virus) profiling — that requires the optional Microbe module and BLAST databases (~3.5 hours average runtime), beyond the QC/Alignment/Annotation scope of this skill
- Only a differential expression analysis is needed on existing count matrices — use the Function module directly instead of re-running QC/Alignment/Annotation

## Inputs

- 12 serum small RNA-seq FASTQ files (50 nt reads) from healthy human controls
- Prebuilt COMPSRA annotation databases (miRNA_hg38 etc.)
- STAR human reference genome index (star_hg38 or star_hg19)
- Sample list file for multi-sample runs (-inf)

## Outputs

- Tab-delimited miRNA count matrix per sample (miRBase-annotated)
- Count profiles of other small RNA types (piRNA, snRNA, snoRNA, tRNA, circRNA)
- QC-trimmed FASTQ files (read lengths 0–42 nt after trimming)
- STAR alignment BAM files (Aligned.out.bam)

## How to apply

Install COMPSRA v1.0 (Java JRE 8+) from https://github.com/cougarlj/COMPSRA on a server with at least 30 GB RAM, then download prebuilt annotation databases (e.g., miRNA_hg38) and the STAR genome index via the built-in toolkit (-tk -dr). Run the QC, Alignment, and Annotation modules with parameters matched to exceRpt: STAR local-end alignment with at most one mismatch, adapter trimming (-ra TGGAATTCTCGGGTGCCAAGG) with 4 random bases removed at both 5' and 3' ends (-rb 4), quality cutoffs (-rh 20 -rt 20 -rr 20), and default full-read annotation overlap (-ann_overlap 1 / -ac 1,2,3,4,5,6). Extract miRBase-annotated miRNA counts from the output tab-delimited count matrices and compare detected miRNA sets with the web-based exceRpt results on the same 12 serum FASTQ files (50 nt reads). The rationale is that matched parameters isolate differences in platform implementation rather than parameter choices; COMPSRA takes ~10 minutes per sample versus ~20 minutes for exceRpt, so the stand-alone route is also faster for large batches.

## Related tools

- **COMPSRA** (Stand-alone modular pipeline (QC, Alignment, Annotation) run with exceRpt-matched parameters to identify and quantify miRNAs from serum small RNA-seq FASTQ files) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default RNA-seq aligner used by COMPSRA for local-end alignment of trimmed reads to hg38 with at most one mismatch)
- **miRBase** (miRNA reference database (miRBase20) used by the Annotation module to produce the miRNA count output)
- **Java** (Runtime environment (JRE 8+) required to execute the COMPSRA.jar stand-alone package)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./serum/sample.list -out ./serum_out/
```

## Evaluation signals

- On the 12 serum samples, the platform should detect 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs at a read count threshold of 5
- The shared miRNA set between COMPSRA and exceRpt should reach ~358 miRNAs (~90% concordance) when parameters are correctly matched
- Runtime should be ~10 minutes per sample (vs ~20 minutes for exceRpt); a large deviation indicates misconfiguration
- After adapter trimming with 4 random bases removed at both ends, read lengths should fall in the 0–42 nt range from 50 nt raw reads
- The miRNA count output must be a tab-delimited text file with RNA molecules as rows and samples as columns

## Limitations

- First-time STAR index building costs about 3 hours and may fail if the STAR binary lacks execute permissions (fix with chmod)
- COMPSRA software version used in the comparison was not explicitly stated in the article's results section
- Supports only hg19/hg38 human genomes (plus rat rno5/rno6 added in v1.0.1) for genome alignment
- The Microbe module requires substantial additional resources (~3.5 hours average for 12 samples) and BLAST database downloads, so it is out of scope for a quick matched-parameter miRNA comparison

## Evidence

- [results] Matched-parameter run setup: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [results] Runtime advantage: "COMPSRA takes ~10 minutes per sample, but exceRpt will cost ~20 minutes for each sample (see Supplementary Fig. S1)"
- [results] Read length after trimming: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
- [results] Detection counts: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs"
- [readme] Pipeline invocation with matched parameters: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6"
