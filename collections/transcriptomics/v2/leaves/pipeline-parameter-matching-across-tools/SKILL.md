---
name: pipeline-parameter-matching-across-tools
description: 'Use when comparing runtime or performance of two small RNA-seq analysis
  pipelines and you need a controlled benchmark: same FASTQ inputs, same parameters,
  same server resources (e.g. 30 GB RAM).'
when_to_use_negative:
- The goal is to compare RNA detection sensitivity or abundance rather than runtime
  — the article's comparison showed COMPSRA identified a greater diversity and abundance
  of small RNAs, which is a different evaluation from timing
- Only one pipeline is available or the inputs differ between tools — the benchmark's
  validity depends on running both tools on the same 12 FASTQ files with matched parameters
- The data are not small RNA-seq (e.g. bulk RNA-seq with >200 nt inserts) or lack
  the NEXTflex-style adapter/random-base structure the COMPSRA QC parameters are configured
  for
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: COMPSRA
  role: Tool being benchmarked; run via COMPSRA.jar in pipeline mode (-qc -aln -ann)
    with parameters matching exceRpt, timed per sample
  repo: https://github.com/cougarlj/COMPSRA
- name: exceRpt
  role: Web-based comparison pipeline; run on the same 12 FASTQ inputs and timed per
    sample to provide the reference runtime
- name: STAR
  role: Default aligner inside COMPSRA's Alignment module; its runtime is included
    in the per-sample timing
- name: BLAST
  role: Aligner in COMPSRA's Microbe module mapping unmapped reads to the UCSC nt
    database; its runtime is measured separately (~3.5 h average)
- name: Java
  role: Runtime environment required to execute COMPSRA (JRE version 8 or up)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:98b9aa58d7619ea9df29562dfed512be858c97ce4600bf326bbba6efaf92f156
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - Java
  - STAR
  - exceRpt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# pipeline-parameter-matching-across-tools

## Summary

A fair benchmarking skill for comparing per-sample runtime of two small RNA-seq pipelines (COMPSRA v1.0 vs the web-based exceRpt) by running both on identical 50-nt serum FASTQ inputs with matched parameters and equivalent hardware, then aggregating measured timings per module.

## When to use

Use when comparing runtime or performance of two small RNA-seq analysis pipelines and you need a controlled benchmark: same FASTQ inputs, same parameters, same server resources (e.g. 30 GB RAM).

## When NOT to use

- The goal is to compare RNA detection sensitivity or abundance rather than runtime — the article's comparison showed COMPSRA identified a greater diversity and abundance of small RNAs, which is a different evaluation from timing
- Only one pipeline is available or the inputs differ between tools — the benchmark's validity depends on running both tools on the same 12 FASTQ files with matched parameters
- The data are not small RNA-seq (e.g. bulk RNA-seq with >200 nt inserts) or lack the NEXTflex-style adapter/random-base structure the COMPSRA QC parameters are configured for

## Inputs

- 12 small RNA-seq FASTQ files from healthy human serum samples (50 nt raw reads)
- COMPSRA v1.0 (COMPSRA.jar) with prebuilt annotation databases (miRBase20, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE release 27, circBase) and hg38 STAR index
- exceRpt pipeline run with parameters matched to COMPSRA
- Microbial prebuilt BLAST databases from UCSC nt (blast_archaea, blast_bacteria, blast_fungi, blast_viruses) plus blast_taxonomy

## Outputs

- Per-sample runtime measurements for COMPSRA (QC + STAR alignment + annotation modules)
- Per-sample runtime measurements for exceRpt on the same inputs
- Average runtime for the Microbe module across the four microbial taxons
- Aggregated runtime benchmark table (e.g. runtime_benchmark.csv) with reported reference values alongside measured ones

## How to apply

Obtain the 12 healthy human serum small RNA-seq FASTQ samples (raw reads of 50 nt; after trimming adapters and 4 random bases at both 5' and 3' ends, read length varies from 0 nt to 42 nt). Install COMPSRA v1.0 (requires Java JRE 8 or up) alongside STAR and BLAST via its ToolKit (-tk -dr -ck), and run it on a server with 30 GB RAM using 'the same parameters as exceRpt' — e.g. adapter 'TGGAATTCTCGGGTGCCAAGG', 4 random bases removed at each end (-rb 4), Phred 20 thresholds (-rh 20 -rt 20 -rr 20), hg38 reference. Time the full per-sample pipeline (QC, STAR alignment to hg38, annotation) for COMPSRA, then run exceRpt on the same 12 FASTQ inputs and record its per-sample runtime. Optionally time the Microbe module (BLAST of unmapped reads against the UCSC nt database across viruses, archaea, bacteria and fungi). Aggregate timings per tool and module into a table such as runtime_benchmark.csv, with a column of reported reference values for validation.

## Related tools

- **COMPSRA** (Tool being benchmarked; run via COMPSRA.jar in pipeline mode (-qc -aln -ann) with parameters matching exceRpt, timed per sample) — https://github.com/cougarlj/COMPSRA
- **exceRpt** (Web-based comparison pipeline; run on the same 12 FASTQ inputs and timed per sample to provide the reference runtime)
- **STAR** (Default aligner inside COMPSRA's Alignment module; its runtime is included in the per-sample timing)
- **BLAST** (Aligner in COMPSRA's Microbe module mapping unmapped reads to the UCSC nt database; its runtime is measured separately (~3.5 h average))
- **Java** (Runtime environment required to execute COMPSRA (JRE version 8 or up))

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- Measured COMPSRA per-sample runtime approximates the reported ~10 minutes per sample on the 12 serum samples
- Measured exceRpt per-sample runtime approximates the reported ~20 minutes per sample
- Microbe module average runtime across the 12 samples approximates the reported 3.5 hours including viruses, archaea, bacteria and fungi
- Both tools were run with genuinely identical parameters (adapter sequence, random-base trimming, Phred thresholds) and the same 12 FASTQ inputs, on hardware with 30 GB RAM
- runtime_benchmark.csv contains measured values, per-tool/per-module breakdown, and a column of reported reference values for direct validation

## Limitations

- The article's results section does not state the exact COMPSRA software version used for the timing comparison (README releases range from v1.0 to v1.0.3)
- Timing was performed on a specific server (30 GB RAM); absolute runtimes will not transfer to different hardware
- The Microbe module adds substantial cost (~3.5 hours average per the 12 test samples) on top of the ~10-minute core pipeline, and requires large additional resources (UCSC nt BLAST databases)
- First-time runs build the STAR index, a process that 'will cost about 3 hours', which must be excluded from or accounted for in per-sample runtime

## Evidence

- [results] COMPSRA takes ~10 minutes per sample vs ~20 minutes for exceRpt on the 12 serum samples: "COMPSRA takes ~10 minutes per sample, but exceRpt will cost ~20 minutes for each sample (see Supplementary Fig. S1)"
- [results] COMPSRA run with parameters matched to exceRpt on 30 GB RAM server: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [results] Microbiome module average runtime of 3.5 hours across the 12 samples: "For the 12 test samples, the estimated time on average is 3.5 hours, including viruses, archaea, bacteria and fungi (see Supplementary Table S1)"
- [results] Raw reads are 50 nt with adapters and 4 random bases trimmed at both ends: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
- [readme] STAR index build takes about 3 hours on first run and must be excluded from benchmark timing: "Note: -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours."
